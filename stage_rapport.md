## Alertes emails
Par défault nous avons dans le fichier /var/ossec/etc/psssec.conf pour la configuration d'alertes par emails

<img width="1852" height="1003" alt="email_default_1" src="https://github.com/user-attachments/assets/9771555f-99c1-4dd0-9063-8af2603b9245" />

ce que j'ai fait, j'ai mis mon adresse email pour pourvoir recevoir des alertes lors d'alertes

<img width="1852" height="1003" alt="email_conf_2" src="https://github.com/user-attachments/assets/3c97838b-673c-429e-ae2a-9bd28bbbe62b" />

puis passons  l'installation de postfix

```zsh
sudo apt install postfix libsasl2-modules -y
```

puis éditons le fichier main.cf 
```zsh
sudo nano /etc/postfix/main.cf
```
puis ajouter cela à la fin du fichier

```zsh
relayhost = [smtp.gmail.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_security_level = encrypt
```

Créer le fichier de credentials Gmail
```zsh
sudo nano /etc/postfix/sasl_passwd
```

puis ajoutons cela dans le fichier
[smtp.gmail.com]:587 ton_email@gmail.com:MOT_DE_PASSE_APPLICATION
