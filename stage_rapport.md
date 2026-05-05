## Alertes emails
Par défault nous avons

<img width="1852" height="1003" alt="email_default_1" src="https://github.com/user-attachments/assets/9771555f-99c1-4dd0-9063-8af2603b9245" />

ce que j'ai fait

<img width="1852" height="1003" alt="email_conf_2" src="https://github.com/user-attachments/assets/3c97838b-673c-429e-ae2a-9bd28bbbe62b" />

puis passons  l'installation de postfix

`
sudo apt install postfix libsasl2-modules -y
`

puis éditons le fichier main.cf 
sudo nano /etc/postfix/main.cf
