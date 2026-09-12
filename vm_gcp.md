# Configuration de la VM GCP

Ce guide détaille la configuration d'une machine virtuelle sur Google Cloud Platform (GCP) pour un environnement de laboratoire SOC avec Wazuh.

> **Note** : Bien que GCP offre une version gratuite (free trial), nous recommandons **Contabo** pour un meilleur rapport coût/performance (à partir de 14$/mois).

---

## 1. Création de la machine virtuelle

### Étape 1 : Accéder à la console GCP
Connectez-vous à [Google Cloud Console](https://console.cloud.google.com) et accédez à **Compute Engine → Instances VM**.

Cliquez sur le bouton **Créer une instance**.

<img width="1400" height="720" alt="image" src="https://github.com/user-attachments/assets/6e09a00d-dc16-4d4e-90b7-066581bb2849" />

### Étape 2 : Configuration générale

Vous verrez l'assistant de création avec plusieurs étapes. Voici les paramètres recommandés :

**Configuration matérielle :**
- Pour un laboratoire simple, voici les spécifications minimum recommandées :

<img width="1428" height="645" alt="image" src="https://github.com/user-attachments/assets/97045ddd-e1cd-4a06-aeae-90338633efac" />

### Étape 3 : Sélection du système d'exploitation

Choisissez l'image de démarrage selon vos préférences. Pour ce projet, nous avons sélectionné :
- **OS** : Ubuntu 24.04 LTS MINIMAL (léger et stable)
- **Disque de démarrage** : SSD standard de 150 Go (ajustez selon vos besoins)

<img width="1833" height="624" alt="image" src="https://github.com/user-attachments/assets/de35156d-18c5-4c1a-bf2b-41891f04bc0f" />

### Étape 4 : Configuration réseau

Dans la section **Réseau**, configurez les paramètres suivants :
- **Réseau VPC** : Sélectionnez le réseau par défaut ou créez-en un
- **Sous-réseau** : Choisissez le sous-réseau approprié
- **Pare-feu** : Cochez les cases pour autoriser :
  - ✅ Trafic HTTP
  - ✅ Trafic HTTPS

*Vous configurerez les autres règles de pare-feu (SSH, Wazuh) dans les étapes suivantes.*

<img width="1698" height="520" alt="image" src="https://github.com/user-attachments/assets/e273c2e4-3f68-44f1-809b-cee7675cf20b" />

### Étape 5 : Configuration IP statique

Pour garantir une IP stable pour vos connexions SSH et Wazuh :

1. Dans la section **Adresses IP**, cliquez sur le menu déroulant **Temporaire**
2. Sélectionnez **Créer une adresse IP statique**
3. Donnez un nom explicite (ex: `lab-soc-static-ip`)
4. Cliquez sur **Réserver**

L'IP statique sera assignée et affichée dans la console.

<img width="1039" height="791" alt="image" src="https://github.com/user-attachments/assets/91fbf592-144e-4bd1-a35e-e4a0d6cad0e9" />

### Étape 6 : Finalisation

Cliquez sur le bouton **Créer** pour lancer la VM. La création prend généralement 1-2 minutes.

<img width="1562" height="392" alt="image" src="https://github.com/user-attachments/assets/4a3d4188-623c-41a5-9577-a1d3d418152f" />

---

## 2. Configuration SSH

### Étape 1 : Générer une clé SSH locale

Sur votre machine locale (Linux/macOS), générez une paire de clés SSH :

```bash
ssh-keygen -t ed25519 -C "lab-soc"
```

**Paramètres** :
- `-t ed25519` : Utilise l'algorithme Ed25519 (moderne et sécurisé)
- `-C "lab-soc"` : Ajoute un commentaire pour identifier la clé

Appuyez sur **Entrée** pour tous les paramètres par défaut (chemin et passphrase vides).

<img width="777" height="472" alt="image" src="https://github.com/user-attachments/assets/0421dab8-d990-465e-856e-52cde02833d6" />

### Étape 2 : Récupérer la clé publique

Affichez le contenu de votre clé publique :

```bash
cat ~/.ssh/lab-soc.pub
```

Copiez l'intégralité du contenu (commence par `ssh-ed25519`).

<img width="1851" height="482" alt="image" src="https://github.com/user-attachments/assets/1f68bbb3-1b27-4ea5-9524-c4dc48039876" />

### Étape 3 : Ajouter la clé à GCP

1. Allez dans **Compute Engine → Métadonnées → Onglet "Clés SSH"**
2. Cliquez sur **Ajouter une clé SSH**
3. Collez votre clé publique
4. Cliquez sur **Enregistrer**

GCP extraira automatiquement le nom d'utilisateur de la clé.

### Étape 4 : Connexion SSH

Connectez-vous à votre VM avec :

```bash
ssh -i ~/.ssh/lab-soc ubuntu@<IP_EXTERNE_STATIQUE>
```

**Remplacez** :
- `<IP_EXTERNE_STATIQUE>` par l'IP affichée dans la console GCP

---

## 3. Sécurisation SSH (Optionnel mais recommandé)

### Changer le port SSH par défaut

Pour réduire les tentatives de brute-force, changez le port SSH du port 22 vers un port custom (ex: 2822).

### Étape 1 : Créer une règle de pare-feu GCP

1. Allez dans **VPC Réseau → Pare-feu → Créer une règle de pare-feu**
2. Remplissez les champs :

| Champ | Valeur |
|-------|--------|
| **Nom** | `allow-ssh-2822` |
| **Direction du trafic** | Ingress (entrée) |
| **Cibles** | Toutes les instances (ou tag réseau) |
| **Plage IP source** | Votre IP publique `/32` ou `0.0.0.0/0` |
| **Protocoles/ports** | TCP, port `2822` |

3. Cliquez sur **Créer**

<img width="1642" height="487" alt="image" src="https://github.com/user-attachments/assets/41a68a1f-2b6b-4a47-90cc-c4c4f92b10f0" />

### Étape 2 : Modifier le fichier de configuration SSH sur la VM

Sur la VM, éditez le fichier de configuration SSH :

```bash
sudo nano /etc/ssh/sshd_config
```

Trouvez la ligne `#Port 22` et décommentez-la, puis changez-la :

```bash
Port 2822
```

Sauvegardez (Ctrl+O, Entrée, Ctrl+X).

<img width="1723" height="890" alt="image" src="https://github.com/user-attachments/assets/6b2d748d-0e9d-4e8d-bdbf-e9fc146c3b8e" />

### Étape 3 : Redémarrer SSH

```bash
sudo systemctl restart ssh
```

### Étape 4 : Se reconnecter

Fermez la connexion actuelle et reconnectez-vous avec :

```bash
ssh -i ~/.ssh/lab-soc -p 2822 ubuntu@<IP_EXTERNE_STATIQUE>
```

### Étape 5 : Installer Fail2Ban (recommandé)

Pour une protection supplémentaire contre les attaques par brute-force :

```bash
sudo apt update
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

Fail2Ban monitore les tentatives de connexion échouées et bannit automatiquement les adresses IP suspectes.

---

## 4. Configuration des règles de pare-feu pour Wazuh

Wazuh utilise les ports suivants pour la communication :
- **Port 1514** : Réception des événements depuis les agents (UDP/TCP)
- **Port 1515** : Communication agent-manager sécurisée (TCP)

### Créer les règles de pare-feu

1. Allez dans **VPC Réseau → Pare-feu → Créer une règle de pare-feu**

2. **Première règle (Port 1514)** :

| Champ | Valeur |
|-------|--------|
| **Nom** | `allow-wazuh-1514` |
| **Direction du trafic** | Ingress |
| **Cibles** | Tag réseau : `wazuh-manager` (ou toutes les instances) |
| **Plage IP source** | `0.0.0.0/0` (ou restreindre à vos agents) |
| **Protocoles/ports** | TCP, UDP port `1514` |

3. **Deuxième règle (Port 1515)** :

| Champ | Valeur |
|-------|--------|
| **Nom** | `allow-wazuh-1515` |
| **Direction du trafic** | Ingress |
| **Cibles** | Tag réseau : `wazuh-manager` |
| **Plage IP source** | `0.0.0.0/0` (ou restreindre à vos agents) |
| **Protocoles/ports** | TCP port `1515` |

4. Cliquez sur **Créer** pour chaque règle

<img width="1846" height="1002" alt="image" src="https://github.com/user-attachments/assets/a922a4b5-b128-484c-87ee-eec12206ea55" />

### Vérifier la connectivité

Une fois Wazuh installé, testez la connexion depuis un agent :

```bash
nc -zv <IP_MANAGER> 1514
nc -zv <IP_MANAGER> 1515
```

---

## Résumé des configurations

| Composant | Port | Protocole | Source |
|-----------|------|-----------|--------|
| **SSH** | 22 (ou 2822) | TCP | Votre IP |
| **HTTP** | 80 | TCP | Toute adresse |
| **HTTPS** | 443 | TCP | Toute adresse |
| **Wazuh Events** | 1514 | TCP/UDP | Agents Wazuh |
| **Wazuh Agent** | 1515 | TCP | Agents Wazuh |

---

## Prochaines étapes

- 📦 [Installation de Wazuh Manager](./wazuh_installation.md)
- 🔧 [Configuration des agents Wazuh](./agent_configuration.md)
- 🛡️ [Hardening de la VM](./vm_hardening.md)
