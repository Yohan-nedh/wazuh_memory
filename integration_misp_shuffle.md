# MISP sous Wazuh

Pour des raison de simplicité et de cohérence j'ai préféré utiliser une api MISP connecter à shuffle pour l'enrichissement des alertes wazuh. Car la premiere approche par script pose déjà ces limites mais bon continuons avec **un api** 

## étape 1 : Générer la clé API MISP et configuration de l'app

Connecte à linterface interface MISP : https://34.155.172.52:8443 et allons sur https://34.155.172.52:8443/users/view/1 pour récuperer **l'api** je précise que j'avais déjà générer une avant.

<img width="1858" height="1072" alt="image" src="https://github.com/user-attachments/assets/0549d6fc-5a69-4280-a79b-100d18bf98e1" />

Et après avoir copier l'api nous allons sur **Shuffle** et allons dans la app

<img width="1858" height="1072" alt="image" src="https://github.com/user-attachments/assets/3c5c8a77-390a-4906-aa00-d7fe8ef15010" />

puis vous allez sur voir MISP 

<img width="1858" height="1072" alt="image" src="https://github.com/user-attachments/assets/0e17ae8e-38d9-4d94-9a4d-000c000edabf" />

et vous allez voir le bouton **activate** cliquez dessus et je précise que l'on était dans la section **Discover public Apps**

<img width="1858" height="1072" alt="image" src="https://github.com/user-attachments/assets/5b424a20-4d32-4896-9b7e-19f3886b0a1d" />

puis nous allons dans section **Organization Apps** nous voyons que notre application est là.

<img width="1858" height="1072" alt="image" src="https://github.com/user-attachments/assets/b096bb3b-9584-467c-b6a2-297aeae66bb8" />

## Construire le workflow complet avec MISP

pour cela nous allons retourner dans notre workflows qui est sur shuffle et ajouter **MISP** (j'avais déjà fait un workflow test)

<img width="1858" height="1072" alt="image" src="https://github.com/user-attachments/assets/042724ec-e445-4a07-a4ac-2e2067715cd0" />

Nous allons supprimer cela pour le moment et juste laisser wazuh-alert nous allons connecter ces deux là

<img width="1858" height="1072" alt="image" src="https://github.com/user-attachments/assets/0bbd21f7-eb80-42ce-89ec-babe2630518f" />

Et nous allons passer à la configuration de **MISP**

<img width="1858" height="1072" alt="image" src="https://github.com/user-attachments/assets/4533d0d9-7239-452e-b288-63aaff746c9a" />

Nous cliquons sur **Add Authentification** nous allons mettre **la clé API** 

<img width="1858" height="1072" alt="image" src="https://github.com/user-attachments/assets/09652db5-f4fe-4ba3-a292-87ecc661ee53" />


<img width="1858" height="1072" alt="image" src="https://github.com/user-attachments/assets/073155da-c341-49d7-9d51-f82173c6d8b2" />

nous allons selectionner **Restsearch attributes** qui est l'action MISP qui permet de rechercher si une valeur (IP, hash, domaine) existe dans la base de threat intelligence. C'est l'endpoint `/attributes/restSearch` de l'API MISP, utilisé ici pour vérifier en temps réel si une IP détectée par Wazuh est un IOC connu.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a1787f21-2cc4-4f75-beff-26d1c57812a0" />
