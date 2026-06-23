# TEST FINAUX V1

## Photo du workflow
<img width="1043" height="668" alt="image" src="https://github.com/user-attachments/assets/c4021f91-3ca7-4b5c-8479-bd9cbbe085b3" />

### Premier test sur windows

Nous allons simuler un utilisateur ayant télécharger un fichier .zip contenant un fichier malveillant 

ici nous avons l'utilisateur qui décrompresse le fichier

<img width="1043" height="668" alt="image" src="https://github.com/user-attachments/assets/e24e8631-264c-480e-904c-e0573fd177e4" />
<img width="605" height="136" alt="image" src="https://github.com/user-attachments/assets/c24a7e39-7b1c-4aec-b87e-e04c2fc06d92" />

De ce pas allons sur wazuh et vérifions ce qu'il en est 

<img width="1455" height="270" alt="image" src="https://github.com/user-attachments/assets/697d35ce-c5a8-41fd-b9c3-841dd8a22c54" />

Nous voyons que virus Ttal à bien analyser le fichier et si nous cliquons sur le lien nous ramenant à virus total nous voyons que le fichier .exe est un trojan

<img width="1847" height="986" alt="image" src="https://github.com/user-attachments/assets/b6c3a19d-8d5a-402b-9152-d3ce1a78b8de" />

du coté de shuffle nous avons ceci
 l'évènement qui est crée dans misp 
 <img width="506" height="706" alt="image" src="https://github.com/user-attachments/assets/e29c4eee-bb98-498b-8f72-75f288a5431a" />

puis le noued qui doit ecrire les attribut les ecrit

<img width="506" height="706" alt="image" src="https://github.com/user-attachments/assets/0ebafa02-3def-48fb-a6bf-891b87afdca3" />

et enfin l'active-response qui se fait 

<img width="506" height="706" alt="image" src="https://github.com/user-attachments/assets/32a81c11-e0c2-46a7-8e37-2a55217eaaff" />

et revenons wazuh pour voir ce qu'il en est 

