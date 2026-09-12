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
Dans `.env`, tu peux ajuster (optionnel mais recommandé) :
- `MYSQL_PASSWORD`
- `BASE_URL`
- `ADMIN_EMAIL`
- etc.

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
Va sur `https://<IP-de-ta-VM>` dans le navigateur.
- Identifiant : `admin@admin.test`
- Mot de passe : `admin`

⚠️ **Pense à changer ces identifiants par défaut dès la première connexion.**

## Points d'attention pour ta VM (Wazuh + MISP + Shuffle en simultané)

- **Prérequis officiels** : Docker Engine 25+ et Docker Compose plugin 2.17+
  - Vérifier avec : `docker -v` et `docker compose version`
  
- **Registre d'images** : MISP tire ses images depuis `ghcr.io`
  - Vérifier que ta VM GCP a bien accès à ce registre sortant
  
- **Gestion des ports** : ⚠️ Attention aux conflits !
  - MISP prend le 443/80 par défaut
  - Si Wazuh dashboard (443) ou Shuffle sont déjà dessus, il faudra remapper les ports dans `docker-compose.yml`

## À faire ensuite

- [ ] Configurer le `.env` avec des valeurs cohérentes pour ton setup
- [ ] Gérer les ports (BASE_URL, éviter les conflits avec Wazuh/Shuffle)
- [ ] Changer les identifiants par défaut après la première connexion
