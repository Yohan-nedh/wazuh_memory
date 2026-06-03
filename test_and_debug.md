# TEST
## TEST 1 : shuffle dans l'envoi des alertes wazuh et la création d'évènement et l'attribution des attribut (MISP 2 et 3)

Je précise que dans ce test j'ai effectué un brute force sur une machine vulnérable

Voici l'architecture que j'ai mis en place:

<img width="1920" height="1065" alt="image" src="https://github.com/user-attachments/assets/ebf0f271-5a24-4ef6-bcad-9b0e95a2c6d9" />

## Configuration des des apps MISP

**MISP_2 CONFIG**
---
```
body:
{
  "info": "$exec.all_fields.rule.description — Agent: $exec.all_fields.agent.name — Règle #$exec.all_fields.rule.id",
  "distribution": 0,
  "threat_level_id": 2,
  "analysis": 0,
  "comment": "Niveau: $exec.all_fields.rule.level | SrcIP: $exec.all_fields.data.srcip | Agent IP: $exec.all_fields.agent.ip | Timestamp: $exec.all_fields.timestamp"
}

path:
/events

API: misp (nom de l'api)

```
---

**MISP_3 CONFIG**
---
```
body:
{
  "category": "Network activity",
  "type": "ip-src",
  "value": "$exec.all_fields.data.srcip",
  "comment": "Règle $exec.all_fields.rule.id — Niveau $exec.all_fields.rule.level — Agent $exec.all_fields.agent.name",
  "to_ids": 1
}

path:
https://34.155.172.52:8443/attributes/add/$misp_2.body.Event.id

API: misp (nom de l'api)

```

De façon normale nous devrions avoir ces résultats:

<img width="1920" height="1065" alt="image" src="https://github.com/user-attachments/assets/1830efd1-a803-4f95-adee-d5159af5607e" />
<img width="1920" height="1065" alt="image" src="https://github.com/user-attachments/assets/feea80f7-8541-4537-97cd-f8e4aa62dde1" />

---

Mais nous avous ces résultat aujourd'hui

<img width="1920" height="1065" alt="image" src="https://github.com/user-attachments/assets/76cf5bf0-1fe1-4f53-87d0-85ad85f38942" />

sur la plateforme de **MISP** nous voyons que l'alerte est bien arrivé mais les informations lié à l'alerte ne sont pas arriver.

### noeud MISP_2 Résultat

Comme nous le voyons nous avons un succès de la part su noeud ce qui suggère que l'alerte est envoyer vers l'interface de **MISP**

<img width="1920" height="1065" alt="image" src="https://github.com/user-attachments/assets/d884be31-4862-40e2-bf77-2a20c608750c" />

### noeud MISP_3 Résultat

<img width="1920" height="1065" alt="image" src="https://github.com/user-attachments/assets/86f63efd-1bc9-44f5-98f5-b766bef7c123" />

nous voyons ici qu'il est dit que l'api n'as pas les autorisations requisent mais cette erreur n'est pas admissible.


## reconstruction

En faite c'est pas une reconstruction du problème mais plus juste une revérification en effet je n'ai pas encore trouver ce qui cloche mais pour l'instant ça fonctionne

<img width="1857" height="1043" alt="image" src="https://github.com/user-attachments/assets/61b30b48-daae-4cf9-bec0-a5a44a29f4c8" />

**MISPcreatevent**
---
```
body:
{
  "info": "$exec.all_fields.rule.description — Agent: $exec.all_fields.agent.name — Règle #$exec.all_fields.rule.id",
  "distribution": 0,
  "threat_level_id": 2,
  "analysis": 0,
  "comment": "Niveau: $exec.all_fields.rule.level | SrcIP: $exec.all_fields.data.srcip | Agent IP: $exec.all_fields.agent.ip | Timestamp: $exec.all_fields.timestamp"
}

path:
/events

API: misp (nom de l'api)

```
---


**MISPattribut**
---
```
body:
{
  "category": "Network activity",
  "type": "ip-src",
  "value": "$exec.all_fields.data.srcip",
  "comment": "Règle $exec.all_fields.rule.id — Niveau $exec.all_fields.rule.level — Agent $exec.all_fields.agent.name",
  "to_ids": 1
}

path:
https://34.155.172.52:8443/attributes/add/$mispcreatevent.body.Event.id

API: misp (nom de l'api)

```
---

pour avoir ce résultat

<img width="1857" height="1043" alt="image" src="https://github.com/user-attachments/assets/57195080-4fce-4d3e-8a1c-3d6389a4a5e9" />
<img width="1857" height="1043" alt="image" src="https://github.com/user-attachments/assets/2e6b085a-a420-42fd-9b61-49c92ad8af20" />

pour l'instant nous allons continuer en espérant que tout se passe bien.

## Étape 1 — Ajouter d'autres attributs MISP selon le type d'alerte (ip-dst, url, email, filename, md5…)

### pour l'ip destination nous avons ajouter ceci (mispaddattribut)

```
body:
{
  "Attribute": [
    {
      "category": "Network activity",
      "type": "ip-src",
      "value": "$exec.all_fields.data.srcip",
      "comment": "Règle $exec.all_fields.rule.id — Niveau $exec.all_fields.rule.level — Agent $exec.all_fields.agent.name",
      "to_ids": 1
    },
    {
      "category": "Network activity",
      "type": "ip-dst",
      "value": "$exec.all_fields.agent.ip",
      "comment": "Règle $exec.all_fields.rule.id — Niveau $exec.all_fields.rule.level — Agent $exec.all_fields.agent.name",
      "to_ids": 1
    }
  ]
}
```
<img width="1456" height="818" alt="image" src="https://github.com/user-attachments/assets/5df48f14-188f-4bce-9d69-31af99a104ba" />
<img width="1456" height="818" alt="image" src="https://github.com/user-attachments/assets/3be8cdb2-36ee-438a-a771-f581e986a8ac" />
nous voyons bien lip attaquante et l'ip cible

### nouvelle architecture et creation de condition 

voici notre architecture qui correspont  trois types d'attaques 

<img width="1857" height="1043" alt="image" src="https://github.com/user-attachments/assets/c10c8166-1868-4bfe-bdd6-73aa99d47a95" />

### condition entre mispcreatevent et mispaddattribut(pour le brute force)

tu clique sur la liaison entre les deux et tu as ça 

<img width="1857" height="1043" alt="image" src="https://github.com/user-attachments/assets/6801da58-903e-4d74-a426-ee725bf3e4cb" />

puis sur **new condition** 

<img width="1857" height="1043" alt="image" src="https://github.com/user-attachments/assets/cc6580b6-7dda-444d-8588-aeb65bdfac04" />

Parfait ! Le panneau Condition est ouvert. Je vois les champs **source** et **destination** avec **"equals"** au milieu.

---

**Pour la condition SSH, remplis comme suit :**

**Champ source (gauche) :**
```
$exec.all_fields.rule.groups
```

**Opérateur (milieu) :**
- Clique sur **"equals"** et change en **"contains"**

**Champ destination (droite) :**
```
sshd
```

---

**Visuellement :**

```
[$exec.all_fields.rule.groups]  [contains]  [sshd]
```

---

**Attention — note importante**

Le champ `rule.groups` dans Wazuh retourne un **tableau** comme :
```json
["sshd", "authentication_failed", "invalid_login"]
```

Donc **"contains"** est le bon opérateur, pas "equals".

---

Remplis les champs et clique **Submit**

<img width="1857" height="1043" alt="image" src="https://github.com/user-attachments/assets/73776441-c254-4034-b2bf-a68366a77a4b" />

nous voyons que la condition  bien été faite **1 condition**

<img width="1456" height="818" alt="image" src="https://github.com/user-attachments/assets/7762e7b3-7983-45f7-89c5-d576af4c5664" />


## configuration de MispaddattributeFIM

Il faudra configurer :

**URL :**
```
https://34.155.172.52:8443/attributes/add/$mispcreatevent.body.Event.id
```

**Method :** `POST`

**Body :**
```json
{
  "Attribute": [
    {
      "category": "Payload delivery",
      "type": "filename",
      "value": "$exec.all_fields.syscheck.path",
      "comment": "Règle $exec.all_fields.rule.id — Niveau $exec.all_fields.rule.level — Agent $exec.all_fields.agent.name",
      "to_ids": 1
    },
    {
      "category": "Payload delivery",
      "type": "md5",
      "value": "$exec.all_fields.syscheck.md5_after",
      "comment": "Règle $exec.all_fields.rule.id — Niveau $exec.all_fields.rule.level — Agent $exec.all_fields.agent.name",
      "to_ids": 1
    }
  ]
}
```

**Headers :** même que `Mispaddattribute` (Authorization + Content-Type + Accept)

**SSL verify :** False

<img width="1857" height="1043" alt="image" src="https://github.com/user-attachments/assets/311e8356-f400-4c99-81b9-a9e530a8218b" />
<img width="1857" height="1043" alt="image" src="https://github.com/user-attachments/assets/9e41bd81-dee4-4df4-ba51-9da3d8850f0e" />

### condition

<img width="1857" height="1043" alt="image" src="https://github.com/user-attachments/assets/b4fbce58-823f-4335-addd-c2a48a710aad" />

 




