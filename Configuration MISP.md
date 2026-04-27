# MISP configuration

Dans cette partie de ce mémoire nous allons configurer **MISP** pour deux choses **l'enrichessement des alertes et la correlation et l'analyse**
Mais avant tout procedons à l'installation et configuration de **MISP** via docker

Le script a détecté Ubuntu 24.04 mais **ne la supporte pas encore officiellement**. Il s'est arrêté avec le message :

---

## Solution — Installation via Docker (recommandée pour Ubuntu 24.04)

C'est la méthode la plus fiable pour Ubuntu 24.04. Voici les étapes :

### Étape 1 — Installer Docker

```bash
# Installer Docker
curl -fsSL https://get.docker.com | sudo bash

# Ajouter ton utilisateur au groupe docker
sudo usermod -aG docker $USER

# Vérifier l'installation
docker --version
```

### Étape 2 — Installer Docker Compose

```bash
sudo apt install -y docker-compose-plugin

# Vérifier
docker compose version
```

### Étape 3 — Déployer MISP via Docker

```bash
# Cloner le repo officiel MISP Docker
git clone https://github.com/MISP/misp-docker.git
cd misp-docker

# Copier la configuration par défaut
cp template.env .env

# Lancer MISP
docker compose up -d
```

---

### configurations des variable de .env

<img width="953" height="678" alt="image" src="https://github.com/user-attachments/assets/a5c5ac31-3f93-4fa2-a7bb-fd5a75556343" />

Je vois ton fichier `.env`. Voici **uniquement ce qui est nécessaire** pour ton cas, avec les explications :

---

## Ce qu'il faut renseigner et pourquoi

### 1. `ADMIN_EMAIL` — Identifiant de connexion à MISP
```bash
ADMIN_EMAIL=admin@misp.local
```
> C'est le login que tu utiliseras pour te connecter à l'interface web MISP.

---

### 2. `ADMIN_PASSWORD` — Déjà renseigné ✅
```bash
ADMIN_PASSWORD=m15p_w47u1-1...
```
> Tu l'as déjà. Note ce mot de passe quelque part, c'est le mot de passe admin.

---

### 3. `ADMIN_ORG` — Nom de ton organisation
```bash
ADMIN_ORG=MonLabo
```
> MISP est conçu pour le partage entre organisations. Mets le nom de ton labo ou université.

---

### 4. `BASE_URL` — Déjà renseigné ✅
```bash
BASE_URL=https://172.16.120.39:8443
```
> C'est l'adresse à laquelle MISP sera accessible depuis ton navigateur.

---

### 5. `DISABLE_SSL_REDIRECT` — Éviter le conflit avec Wazuh
```bash
DISABLE_SSL_REDIRECT=true
```
> Sans ça, MISP essaie de forcer le port 443 qui est déjà pris par Wazuh.

---

## Ce qu'il ne faut PAS toucher pour l'instant
Tout le reste (LDAP, OIDC, AAD, S3, SMTP...) — **laisse vide**, c'est pour des configurations avancées inutiles pour ton mémoire.

---

## Résumé des modifications à faire

```bash
ADMIN_EMAIL=admin@misp.local
ADMIN_ORG=MonLabo
DISABLE_SSL_REDIRECT=true
# BASE_URL et ADMIN_PASSWORD sont déjà bons ✅
```

Fais ces modifications, sauvegarde, puis :

```bash
newgrp docker
docker compose down
docker compose up -d
docker compose ps
```

### Enrichissement des alerts(logs)

Dans un SIEM, MISP sert à enrichir les événements de sécurité en temps réel. Quand le SIEM reçoit un log contenant une IP ou un hash, il le compare automatiquement avec les IoC stockés dans MISP pour savoir si c'est une menace connue.
Concrètement :

Un log réseau contient une IP → le SIEM interroge MISP → MISP répond "cette IP est associée à un groupe APT"
Le SIEM enrichit alors l'alerte avec ce contexte


problème
[+] up 38/48
 ⠙ Image ghcr.io/misp/misp-docker/misp-modules:latest [⣿⣿⣿⣿⣿⡀⡀⡀⣿] 135.3MB / 606.2MB          Pulling                                                                                                                                 1375.0s
 ✔ Image ghcr.io/egos-tech/smtp:1.1.3                                                        Pulled                                                                                                                                  1289.7s
 ✔ Image valkey/valkey:7.2                                                                   Pulled                                                                                                                                   384.9s
[+] up 51/51iadb:10.11                                                                       Pulled                  ✔ Image ghcr.io/misp/misp-docker/misp-modules:latest Pulled                                                 2978.9s[+] up 52/56r.io/egos-tech/smtp:1.1.3                 Pulled                                                 1289.7s
[+] up 56/57r.io/misp/misp-docker/misp-modules:latest Pulled                                                 2978.9s ✔ Image ghcr.io/misp/misp-docker/misp-modules:latest Pulled                                                 2978.9s ✔ Image ghcr.io/egos-tech/smtp:1.1.3                 Pulled                                                 1289.7s
 ✔ Image valkey/valkey:7.2                            Pulled                                                  384.9s
 ✔ Image mariadb:10.11                                Pulled                                                  619.0s
 ✔ Image ghcr.io/misp/misp-docker/misp-core:latest    Pulled                                                 2581.1s
 ✔ Network misp-docker_default                        Created                                                   0.2s
 ✔ Volume misp-docker_cache_data                      Created                                                   0.0s
 ✔ Volume misp-docker_mysql_data                      Created                                                   0.0s
 ✔ Volume misp-docker_misp_guard_ca                   Created                                                   0.0s
 ✔ Container misp-docker-redis-1                      Healthy                                                   8.8s
 ✘ Container misp-docker-misp-modules-1               Error dependency misp-modules failed to start            13.3s
 ✔ Container misp-docker-mail-1                       Started                                                   3.2s
 ⠋ Container misp-docker-db-1                         Waiting                                                  13.3s
 ✔ Container misp-docker-misp-core-1                  Created                                                   0.2s
dependency failed to start: container misp-docker-misp-modules-1 is unhealthy

