## 1- Installation de wazuh 

 Pour l'installation de wazuh nous avons suivi la démarche montrer dans le guide de démarrage (https://documentation.wazuh.com/current/quickstart.html) et utiliser la commande suivante:
```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```
mais libre à vous de choisir le moyen d'installation ici, il sera directement installer sur le serveur **lab-soc** dont nous auront ces trois composant c'est à dire **le manager, indexer et le dashboard** et à la fin de l'installation
nous aurons le nom et le mot de passe pour se connecter

<img width="1227" height="281" alt="image" src="https://github.com/user-attachments/assets/c8317f5e-610c-486b-80ac-e987db99f015" />
<img width="918" height="934" alt="image" src="https://github.com/user-attachments/assets/186b5f1a-78ad-4c32-945c-bdc6e7fa7f23" />

et voilà notre **wazuh**

<img width="1857" height="906" alt="image" src="https://github.com/user-attachments/assets/2aa45f1b-bcfb-461c-8f44-3a7b926e5158" />

## Configuratoin reception de mail

Voir la configuration de postfix mentionner dans ce dépôt sous le nom de postfix_config

