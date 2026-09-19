# Résolution d'un échec SSL entre Wazuh et Shuffle (intégration `wazuh-integratord`)

## Contexte

Dans le cadre du pipeline SOC (Wazuh → Shuffle → MISP), les alertes Wazuh n'atteignaient jamais Shuffle : aucune exécution n'apparaissait dans l'historique du workflow, alors que MISP renvoyait par ailleurs une erreur trompeuse (`403 - Authorization failed`) qui semblait pointer vers un problème de permissions API.

## Diagnostic

L'investigation a montré que le blocage se situait en réalité **en amont de MISP**, au niveau de la connexion HTTPS entre `wazuh-integratord` et le webhook Shuffle. Trois causes distinctes ont été identifiées successivement :

1. **Certificat auto-signé non reconnu** — Le module Python `requests`, utilisé par le script d'intégration Wazuh, rejetait la connexion avec l'erreur `SSLCertVerificationError: self-signed certificate`. Wazuh embarque son propre interpréteur Python avec son propre magasin de certificats (`certifi`), indépendant de celui du système.

2. **Absence de SAN (Subject Alternative Name)** — Une fois le certificat ajouté au magasin de confiance, une nouvelle erreur est apparue : `certificate is not valid for '34.178.224.154' — IP address mismatch`. Le certificat par défaut livré avec l'image Docker `shuffle-frontend` (`CN=Shuffle`, généré par l'éditeur du projet) ne contenait aucun SAN. Depuis plusieurs années, la validation TLS moderne ignore le champ CN et exige un SAN correspondant à l'IP ou au nom d'hôte contacté — aucune configuration réseau ne pouvait donc contourner ce problème sans régénérer le certificat.

3. **Certificat intégré en dur dans l'image Docker** — Le fichier `entrypoint.sh` du conteneur `shuffle-frontend` ne fait que du templating de configuration nginx (`envsubst`) ; il ne régénère pas les certificats. Ceux-ci (`/etc/nginx/fullchain.cert.pem` et `/etc/nginx/privkey.pem`) sont figés dans l'image et identiques pour tout déploiement Shuffle non personnalisé.

## Solution appliquée, étape par étape

### 1. Générer un certificat auto-signé avec le bon SAN

Depuis le dossier d'installation de Shuffle :

```bash
cd ~/Shuffle
openssl req -x509 -nodes -days 825 -newkey rsa:2048 \
  -keyout shuffle-privkey.pem \
  -out shuffle-fullchain.pem \
  -subj "/C=BJ/ST=Atlantique/L=Abomey-Calavi/O=Shuffle/CN=34.178.224.154" \
  -addext "subjectAltName=IP:34.178.224.154"
```

Vérification du SAN :

```bash
openssl x509 -in shuffle-fullchain.pem -noout -text | grep -A 1 "Subject Alternative Name"
```

### 2. Monter le nouveau certificat dans le conteneur `frontend`

Ajout d'une section `volumes:` dans le bloc `frontend:` du `docker-compose.yml` :

```yaml
  frontend:
    image: ghcr.io/shuffle/shuffle-frontend:latest
    container_name: shuffle-frontend
    hostname: shuffle-frontend
    ports:
      - "${FRONTEND_PORT}:80"
      - "${FRONTEND_PORT_HTTPS}:443"
    networks:
      - shuffle
    volumes:
      - ./shuffle-fullchain.pem:/etc/nginx/fullchain.cert.pem:ro
      - ./shuffle-privkey.pem:/etc/nginx/privkey.pem:ro
    environment:
      - BACKEND_HOSTNAME=${BACKEND_HOSTNAME}
    restart: unless-stopped
    depends_on:
      - backend
```

### 3. Recréer le conteneur frontend

```bash
docker compose up -d --force-recreate frontend
```

### 4. Vérifier que nginx sert bien le nouveau certificat

```bash
openssl s_client -connect 34.178.224.154:3443 -showcerts </dev/null 2>/dev/null \
  | openssl x509 -noout -text | grep -A 1 "Subject Alternative Name"
```

Résultat attendu : `IP Address:34.178.224.154`.

### 5. Mettre à jour le magasin de confiance Python de Wazuh

Récupération du nouveau certificat :

```bash
openssl s_client -connect 34.178.224.154:3443 -showcerts </dev/null 2>/dev/null \
  | openssl x509 -outform PEM > /tmp/shuffle.crt
```

Ajout au `cacert.pem` utilisé par l'interpréteur Python embarqué de Wazuh :

```bash
cat /tmp/shuffle.crt | sudo tee -a \
  /var/ossec/framework/python/lib/python3.10/site-packages/certifi/cacert.pem
```

### 6. Redémarrer Wazuh et vérifier les logs

```bash
sudo systemctl restart wazuh-manager
sudo tail -f /var/ossec/logs/ossec.log | grep integratord
```

Résultat attendu : disparition des erreurs `SSLError` ; les tentatives d'envoi vers Shuffle aboutissent, et une nouvelle exécution apparaît dans l'historique du workflow Shuffle.

## Point retenu pour le mémoire

L'usage d'un certificat auto-signé sans SAN, tel que livré par défaut dans l'image Docker de Shuffle, illustre une limitation typique des environnements de lab : la vérification TLS moderne (RFC 6125) ignore le champ `CN` et exige un SAN explicite, ce qui impose soit une régénération manuelle du certificat, soit l'emploi d'une autorité de certification reconnue (Let's Encrypt, etc.) en environnement de production.

## À traiter ensuite

Le problème initial ayant motivé cette investigation — l'erreur MISP `"Info cannot be empty."` lors de la création d'événements depuis Shuffle — reste à résoudre. Il faudra vérifier la variable utilisée pour construire le champ `info` dans le node MISP du workflow Shuffle, maintenant que les alertes Wazuh parviennent effectivement jusqu'à Shuffle.
