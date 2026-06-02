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
Pas tout à fait ! Laisse-moi clarifier les catégories MISP :

## Les catégories MISP

| Catégorie | Rôle |
|---|---|
| **Network activity** | Tout ce qui est réseau : IP, port, domaine, URL |
| **External analysis** | Résultats d'analyse externe : VirusTotal, sandbox, règle IDS |
| **Payload delivery** | Fichiers malveillants, hash, malware |
| **Artifacts dropped** | Fichiers créés par le malware |
| **Attribution** | Acteur, groupe, campagne |

---

## Dans ton cas brute force SSH

```
data.srcip   → Network activity  (c'est une IP réseau)
agent.ip     → Network activity  (c'est une IP réseau)
rule.id      → External analysis (c'est une règle IDS/Wazuh)
rule.level   → External analysis (c'est une analyse)
timestamp    → External analysis (métadonnée d'analyse)
```

---

## Donc

**Network activity** = les IPs → ce sont les **IOCs**

**External analysis** = les métadonnées Wazuh → c'est le **contexte de l'alerte**

Les deux ensemble = **enrichissement complet** de l'événement MISP 🎯

