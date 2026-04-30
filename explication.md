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
