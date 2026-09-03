# Configuraton de la VM GCP

Ici nous avons utiliser **google cloud platforme** dans sa version **free trial** mais nous vous conseillons d'utiliser **contabo** si vous avez les moyens de vous le permettre à **14$** pour avoir les même configuration.

## 1- Création de la vm

Nous voici sur l'interface de **GCP** et nous allons cliquer sur créer une instance

<img width="1400" height="720" alt="image" src="https://github.com/user-attachments/assets/6e09a00d-dc16-4d4e-90b7-066581bb2849" />

Puis nous avons les différentes étapes de création de la vm que vous configurez en fonction de vos envies 

<img width="1860" height="1041" alt="image" src="https://github.com/user-attachments/assets/37763842-6ac6-48fc-898f-e2b5067dfc8d" />

pour ce projet vu que c'est un lab assez simple j'ai pris cette configuration pour la ram et le cpu

<img width="1428" height="645" alt="image" src="https://github.com/user-attachments/assets/97045ddd-e1cd-4a06-aeae-90338633efac" />

et vous faite le choix de l'os qui vous confient dans mon cas j'ai pris **Ubuntu 24.04 LTS MINIMAL** et les informations sur le disque choisi et sa taille de **150 Go**

<img width="1833" height="624" alt="image" src="https://github.com/user-attachments/assets/de35156d-18c5-4c1a-bf2b-41891f04bc0f" />

Configuration réseaux de la VM ici nous avons autorisé le traffic **http et https** et par la suite nous allons configurer l'ip statique 

<img width="1698" height="520" alt="image" src="https://github.com/user-attachments/assets/e273c2e4-3f68-44f1-809b-cee7675cf20b" />

Et voici comment nous avons configurer notre ip statique et une ip statique sera générer

<img width="1039" height="791" alt="image" src="https://github.com/user-attachments/assets/91fbf592-144e-4bd1-a35e-e4a0d6cad0e9" />

puis nous avons crée la vm en appuyant sur le bouton **créer** et voici le résultat 

<img width="1562" height="392" alt="image" src="https://github.com/user-attachments/assets/4a3d4188-623c-41a5-9577-a1d3d418152f" />

## 2- Connection ssh 

Pour cela nous allons générer **un clé ssh** sur notre pc en loccurance moi je suis sur linux avec cette commande:

```bash
ssh-keygen -t ed25519 -C "nom de la clé"
``` 

<img width="777" height="472" alt="image" src="https://github.com/user-attachments/assets/0421dab8-d990-465e-856e-52cde02833d6" />

Puis vous faite cette commande pour voir le contenu de votre **clé publique** ajouté la clé publique dans GCP : **Compute Engine → Métadonnées → Clés SSH**

```zsh
cat ~/.ssh/lab-soc.pub
```

<img width="1851" height="482" alt="image" src="https://github.com/user-attachments/assets/1f68bbb3-1b27-4ea5-9524-c4dc48039876" />

Après avoir copier coller la clé appuyer enregistrer et maintenant vous pouver vous connecter sur votre terminal via cette commande:
```bash
ssh -i ~/.ssh/id_ed25519 ton_utilisateur@<IP_EXTERNE_STATIQUE>
```

mais je vous conseille de changer le numéro de port par default de **ssh** par exemple 2222 pour changer cela il faut aller dans les **Stratégies de pare-feu** et **crée une règle de pare-feu** et donc nous avons ceci:

Créer la règle de pare-feu GCP pour le port 2822:

  Console GCP → Réseau VPC → Pare-feu → Créer une règle
  
  Nom : allow-ssh-2822
  
  Cibles : toutes les instances (ou tag réseau de lab-soc)
  
  Plage IP source : ton IP publique en /32 (ou 0.0.0.0/0 si tu ne peux pas la restreindre)
  
  Protocoles/ports : TCP, port 2822

<img width="1642" height="487" alt="image" src="https://github.com/user-attachments/assets/41a68a1f-2b6b-4a47-90cc-c4c4f92b10f0" />


Et je vous conseille aussi d'installer **fail2ban**


Et sur la vm il faut modifier le numéro de port en fonction de vos besoin la où nous avons **#Port 22** modifié par le numéro de port que vous avez décider et pour acceder  ce fichier effectuer cette commande

```bash
sudo nano /etc/ssh/sshd_config
```

<img width="1723" height="890" alt="image" src="https://github.com/user-attachments/assets/6b2d748d-e0e9-4e8d-bdbf-e9fc146c3b8e" />

## 3- configuration de règle de pare-feu pour les outils

la configuration est assez simple et se présente de cette façon pour les port 1514 et 1515 de wazuh 

<img width="1846" height="1002" alt="image" src="https://github.com/user-attachments/assets/a922a4b5-b128-484c-87ee-eec12206ea55" />






