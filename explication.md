## Explication complète de ce qu'on a fait

---

### Vue d'ensemble

On a construit un système où **Wazuh surveille ton réseau** et **MISP lui fournit une liste noire d'éléments malveillants connus**. Quand Wazuh détecte quelque chose qui correspond à la liste noire de MISP, il génère automatiquement une alerte critique.

---

### Étape 1 — Installer MISP

MISP est une plateforme de **partage de threat intelligence**. Elle contient des bases de données d'éléments malveillants connus appelés **IoCs (Indicators of Compromise)** :
- Des **hashes** de fichiers malveillants (MD5, SHA256...)
- Des **IPs** de serveurs utilisés par des attaquants
- Des **domaines** malveillants
- Des **URLs** malveillantes

On l'a installé via Docker sur le port 8443 de ton serveur Ubuntu.

---

### Étape 2 — Configurer les feeds MISP

Les feeds sont des **sources externes qui alimentent MISP en IoCs**. On a activé :

- **MalwareBazaar** (abuse.ch) — hashes de malwares connus
- **URLhaus** (abuse.ch) — URLs malveillantes
- **Threatfox** (abuse.ch) — IPs de serveurs C2 (Command & Control)

Concrètement, quand on fetche ces feeds, MISP télécharge automatiquement des milliers d'IoCs depuis ces sources et les stocke dans sa base de données.

---

### Étape 3 — Exporter les IoCs vers Wazuh (CDB lists)

C'est le **cœur de l'intégration MISP → Wazuh**.

Wazuh utilise des fichiers appelés **CDB lists** (Constant Database) pour faire des recherches ultra-rapides. Ce sont de simples fichiers texte avec une entrée par ligne :

```
192.168.1.1:
malware.com:
00007875751bd7572ed590fee27f3f91:
```

On a créé un script Python (`misp_export.py`) qui :
1. **Interroge l'API de MISP** pour récupérer tous les IoCs
2. **Les écrit dans des fichiers** dans `/var/ossec/etc/lists/`
3. **Wazuh compile** ces fichiers en `.cdb` (format binaire optimisé pour la recherche)

On a exporté :
- **6811 IPs** malveillantes → `misp-ip-dst`
- **9836 domaines** malveillants → `misp-domains`
- **9995 hashes** malveillants → `misp-hashes`
- **9999 URLs** malveillantes → `misp-urls`

---

### Étape 4 — Créer les règles de détection Wazuh

Les règles Wazuh définissent **quoi surveiller et comment réagir**. On a créé 4 règles dans `local_rules.xml` :

**Règle 100199** — Règle parent :
> "Quand Wazuh reçoit un log au format JSON, active les règles enfants"

**Règle 100200** — Détection IP :
> "Si le champ `srcip` du log JSON correspond à une entrée dans `misp-ip-dst` → alerte niveau 10"

**Règle 100201** — Détection domaine :
> "Si le champ `hostname` correspond à une entrée dans `misp-domains` → alerte niveau 10"

**Règle 100202** — Détection hash :
> "Si le champ `md5_after` correspond à une entrée dans `misp-hashes` → alerte niveau 12 (critique)"

**Règle 100203** — Détection URL :
> "Si le champ `url` correspond à une entrée dans `misp-urls` → alerte niveau 10"

---

### Étape 5 — Test de détection

On a utilisé `wazuh-logtest` pour simuler un faux événement contenant un hash malveillant connu :

```json
{"md5_after":"00007875751bd7572ed590fee27f3f91","srcip":"1.2.3.4","hostname":"test-host"}
```

Wazuh a :
1. **Décodé** le log JSON (Phase 1 & 2)
2. **Cherché** le hash dans la CDB list `misp-hashes`
3. **Trouvé une correspondance** et déclenché la règle 100202 (Phase 3)
4. **Généré une alerte** de niveau 12 avec le message *"MISP - Hash malveillant détecté"*

---

### Le flux complet en résumé

```
Sources externes          MISP                    Wazuh
(MalwareBazaar,    →   Base d'IoCs      →    CDB Lists
 URLhaus,               (hashes, IPs,        (misp-hashes,
 Threatfox)             domaines, URLs)       misp-ip-dst...)
                                                   ↓
                                            Règles de détection
                                                   ↓
                        Endpoint surveillé → Log entrant
                                                   ↓
                                         Correspondance trouvée ?
                                            OUI → Alerte niveau 12
                                            NON → Rien
```

---

### Ce que ça apporte concrètement

Sans cette intégration, Wazuh ne saurait pas si une IP ou un fichier est malveillant — il verrait juste une connexion ou un fichier normal.

Avec cette intégration, Wazuh dispose d'une **intelligence externe** : dès qu'un élément malveillant connu apparaît sur le système surveillé, il est **immédiatement détecté et signalé**, même si c'est la première fois qu'il apparaît sur ce système.

---

### Ce qui reste à faire

1. **Wazuh → MISP** : quand Wazuh génère une alerte, envoyer cette alerte à MISP pour enrichissement contextuel
2. **Shuffle** : orchestrer tout ça automatiquement (recevoir l'alerte Wazuh, interroger MISP, décider d'une réponse)
3. **Cron job** : automatiser la mise à jour des CDB lists toutes les X heures

Des questions sur une partie spécifique ?




Parfait ! Passons à l'intégration **Wazuh → MISP**.

---

## Ce qu'on va faire

Quand Wazuh génère une alerte, un script Python va automatiquement :
1. **Récupérer les détails** de l'alerte (IP, hash, domaine...)
2. **Interroger MISP** pour savoir si cet élément est connu
3. **Créer un événement dans MISP** si c'est une nouvelle menace
4. **Enrichir l'alerte** avec le contexte MISP

---

## Étape 1 — Créer le script d'intégration

```bash
sudo tee /var/ossec/integrations/custom-misp.py << 'EOF'
#!/usr/bin/env python3
import sys
import json
import requests
import urllib3
from datetime import datetime

urllib3.disable_warnings()

MISP_URL = "https://172.16.120.39:8443"
MISP_KEY = "tbITmryVSAGMXITFgR0k8kQkbvTzm9gAZCstl5pm"
VERIFY_SSL = False

headers = {
    "Authorization": MISP_KEY,
    "Content-Type": "application/json",
    "Accept": "application/json"
}

def query_misp(value, ioc_type):
    payload = {
        "returnFormat": "json",
        "value": value,
        "type": ioc_type,
        "limit": 5
    }
    r = requests.post(
        f"{MISP_URL}/attributes/restSearch",
        headers=headers,
        json=payload,
        verify=VERIFY_SSL
    )
    return r.json().get("response", {}).get("Attribute", [])

def create_misp_event(alert):
    event = {
        "Event": {
            "info": f"Wazuh Alert - {alert.get('rule', {}).get('description', 'Unknown')}",
            "distribution": 0,
            "threat_level_id": 2,
            "analysis": 1,
            "date": datetime.now().strftime("%Y-%m-%d"),
            "Attribute": []
        }
    }
    # Ajouter les attributs selon ce qui est disponible
    data = alert.get("data", {})
    if data.get("srcip"):
        event["Event"]["Attribute"].append({
            "type": "ip-src",
            "value": data["srcip"],
            "to_ids": True,
            "category": "Network activity"
        })
    if data.get("md5_after"):
        event["Event"]["Attribute"].append({
            "type": "md5",
            "value": data["md5_after"],
            "to_ids": True,
            "category": "Payload delivery"
        })
    r = requests.post(
        f"{MISP_URL}/events/add",
        headers=headers,
        json=event,
        verify=VERIFY_SSL
    )
    return r.json()

# Lecture de l'alerte Wazuh
alert_file = sys.argv[1]
with open(alert_file) as f:
    alert = json.load(f)

# Extraction des données
data = alert.get("data", {})
rule_level = alert.get("rule", {}).get("level", 0)

print(f"[*] Traitement alerte niveau {rule_level}")

# Vérification dans MISP
srcip = data.get("srcip")
md5 = data.get("md5_after")

if srcip:
    results = query_misp(srcip, "ip-src")
    if results:
        print(f"[!] IP {srcip} trouvée dans MISP : {len(results)} correspondances")
    else:
        print(f"[-] IP {srcip} non trouvée dans MISP")

if md5:
    results = query_misp(md5, "md5")
    if results:
        print(f"[!] Hash {md5} trouvé dans MISP : {len(results)} correspondances")
    else:
        print(f"[-] Hash {md5} non trouvé dans MISP")

# Créer un événement MISP si alerte critique (niveau >= 10)
if rule_level >= 10:
    print(f"[*] Création événement MISP...")
    result = create_misp_event(alert)
    if result.get("Event"):
        print(f"[+] Événement MISP créé : ID {result['Event']['id']}")
    else:
        print(f"[-] Erreur création événement : {result}")

EOF
```

---

## Étape 2 — Permissions

```bash
sudo chmod 750 /var/ossec/integrations/custom-misp.py
sudo chown root:wazuh /var/ossec/integrations/custom-misp.py
```

---

## Étape 3 — Configurer ossec.conf

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Ajoute avant la balise `</ossec_config>` :

```xml
<integration>
  <name>custom-misp</name>
  <hook_url>https://172.16.120.39:8443</hook_url>
  <api_key>tbITmryVSAGMXITFgR0k8kQkbvTzm9gAZCstl5pm</api_key>
  <level>10</level>
  <alert_format>json</alert_format>
</integration>
```

---

## Étape 4 — Redémarrer Wazuh

```bash
sudo systemctl restart wazuh-manager
```

Lance les étapes 1 et 2 et partage le résultat.
