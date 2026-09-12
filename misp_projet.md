# Installation et Configuration de MISP

## Installation officielle de MISP via `misp-docker`

Cette installation utilise le dépôt officiel `MISP/misp-docker`, en partant du principe que Docker est déjà installé.

### 1. Cloner le dépôt
```bash
git clone https://github.com/MISP/misp-docker
cd misp-docker
```

### 2. Préparer le fichier d'environnement
```bash
cp template.env .env
nano .env
```

#### Configuration du fichier `.env`

Le fichier `.env` contient les variables de runtime pour MISP. Voici les paramètres clés à configurer :

**Variables principales :**
- `ADMIN_EMAIL` : Email pour le compte administrateur par défaut (`admin@admin.test`)
- `ADMIN_ORG` : Nom de l'organisation #1, par défaut défini par MISP
- `ADMIN_ORG_UUID` : UUID de l'organisation, généré automatiquement
- `ADMIN_KEY` : Clé API de l'admin, générée automatiquement
- `ADMIN_PASSWORD` : Mot de passe admin, par défaut `admin`
- `GPG_PASSPHRASE` : Passphrase pour GPG (par défaut `passphrase`)
- `CRON_USER_ID` : ID de l'utilisateur cron (par défaut `1`)
- `BASE_URL` : URL de base pour accéder à MISP (par défaut `http://localhost`)
  - ⚠️ **Important** : Si tu exposes MISP sur un port non-standard, tu dois inclure le port dans l'URL, ex. : `http://192.168.0.1:4433`
- `NGINX_HTTP_PORT` : Port HTTP (par défaut `80`)

**Exemple de configuration pour ta VM :**
```env
ADMIN_EMAIL=nedhsoc@gmail.com
ADMIN_PASSWORD=TonMotDePasse123!
BASE_URL=http://<IP-de-ta-VM>:8080
NGINX_HTTP_PORT=8080
NGINX_HTTPS_PORT=8443
MYSQL_PASSWORD=TaMotDePasseMySQL123!
```

⚠️ **Prévention des conflits de ports** :
- Wazuh Dashboard prend généralement le **443** et **5601**
- MISP prend **80/443** par défaut
- Configure MISP sur les ports **8080** (HTTP) et **8443** (HTTPS) pour éviter les conflits

### 3. Récupérer les images (ou builder toi-même)
```bash
docker compose pull
```
Ou `docker compose build` si tu veux builder les images toi-même (plus long).

### 4. Lancer la stack
```bash
docker compose up -d
```
Le `-d` fait tourner les conteneurs en arrière-plan.

### 5. Se connecter
Va sur `https://<IP-de-ta-VM>:8443` (ou le port HTTPS que tu as configuré) dans le navigateur.
- Identifiant : `admin@admin.test` (ou celui que tu as défini)
- Mot de passe : `admin` (ou celui que tu as défini)

⚠️ **Pense à changer ces identifiants par défaut dès la première connexion.**

## Points d'attention pour ta VM (Wazuh + MISP + Shuffle en simultané)

- **Prérequis officiels** : Docker Engine 25+ et Docker Compose plugin 2.17+
  - Vérifier avec : `docker -v` et `docker compose version`
  
- **Registre d'images** : MISP tire ses images depuis `ghcr.io`
  - Vérifier que ta VM GCP a bien accès à ce registre sortant
  
- **Gestion des ports** : ⚠️ Attention aux conflits !
  - MISP prend le 443/80 par défaut
  - Si Wazuh dashboard (443) ou Shuffle sont déjà dessus, il faudra remapper les ports dans `docker-compose.yml`
  - **Solution** : Configure MISP sur 8080/8443 et update `BASE_URL` en conséquence

## À faire ensuite

- [ ] Configurer le `.env` avec des valeurs cohérentes pour ton setup
- [ ] Gérer les ports (BASE_URL, éviter les conflits avec Wazuh/Shuffle)
- [ ] Changer les identifiants par défaut après la première connexion
- [ ] Configurer les intégrations avec Wazuh et Shuffle
