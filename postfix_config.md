# Configuration pour l'envoie de mail 

Pour que wazuh puisse envoyer des mails au différents analystes de l'organisation

Parfait ! Voici le guide complet étape par étape pour votre configuration Ubuntu.

---

## 📧 Configuration Email Wazuh — Ubuntu

### Étape 1 — Installer les paquets nécessaires

```bash
sudo apt-get update && sudo apt-get install postfix mailutils libsasl2-2 ca-certificates libsasl2-modules -y
```

> Lors de l'installation de Postfix, si une fenêtre apparaît, choisissez **"Internet Site"** et entrez votre nom de domaine (ou le hostname du serveur).

<img width="946" height="393" alt="image" src="https://github.com/user-attachments/assets/207e1b02-58c5-4c97-8b05-f8d3e553ff5a" />

---

### Étape 2 — Configurer Postfix

Ajoutez ces lignes à la fin du fichier `/etc/postfix/main.cf` :

```bash
cat >> /etc/postfix/main.cf << 'EOF'
relayhost = [smtp.gmail.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
smtp_use_tls = yes
smtpd_relay_restrictions = permit_mynetworks, permit_sasl_authenticated, defer_unauth_destination
EOF
```

---

### Étape 3 — Configurer les identifiants Gmail

Remplacez `VOTRE_EMAIL` et `VOTRE_MOT_DE_PASSE_APPLICATION` par vos vraies valeurs :

```bash
echo "[smtp.gmail.com]:587 VOTRE_EMAIL@gmail.com:VOTRE_MOT_DE_PASSE_APPLICATION" > /etc/postfix/sasl_passwd
postmap /etc/postfix/sasl_passwd
```

---

### Étape 4 — Sécuriser le fichier de credentials

```bash
chown root:root /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
chmod 0600 /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
```

---

### Étape 5 — Redémarrer Postfix

```bash
systemctl restart postfix
```

---

### Étape 6 — Tester l'envoi d'email

Remplacez les adresses par les vôtres :

```bash
echo "Test mail from postfix" | mail -s "Test Postfix" -r "VOTRE_EMAIL@gmail.com" EMAIL_DESTINATAIRE@example.com
```

✅ Si vous recevez l'email, Postfix est bien configuré.

---

### Étape 7 — Configurer Wazuh pour les notifications

Éditez le fichier `/var/ossec/etc/ossec.conf` et ajoutez/modifiez la section `<global>` :

```bash
nano /var/ossec/etc/ossec.conf
```

Contenu à mettre dans la balise `<global>` :

```xml
<global>
  <email_notification>yes</email_notification>
  <smtp_server>localhost</smtp_server>
  <email_from>VOTRE_EMAIL@gmail.com</email_from>
  <email_to>EMAIL_DESTINATAIRE@example.com</email_to>
</global>
```

---

### Étape 8 — Redémarrer Wazuh Manager

```bash
systemctl restart wazuh-manager
```

---

### Étape 9 — (Optionnel) Configurer des alertes spécifiques

Ajoutez ces blocs dans `ossec.conf` selon vos besoins :

**Par niveau d'alerte (ex: niveau 4 et plus) :**
```xml
<email_alerts>
  <email_to>EMAIL_DESTINATAIRE@example.com</email_to>
  <level>4</level>
  <do_not_delay/>
</email_alerts>
```

**Par ID de règle :**
```xml
<email_alerts>
  <email_to>EMAIL_DESTINATAIRE@example.com</email_to>
  <rule_id>515, 516</rule_id>
  <do_not_delay/>
</email_alerts>
```

**Par machine source :**
```xml
<email_alerts>
  <email_to>EMAIL_DESTINATAIRE@example.com</email_to>
  <event_location>server1</event_location>
  <do_not_delay/>
</email_alerts>
```

---

> ⚠️ **Rappel important :** Le mot de passe Gmail doit être un **App Password** (mot de passe d'application), pas votre mot de passe Gmail habituel. Vous pouvez en créer un sur [myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords).

Dites-moi si vous bloquez sur une étape ou si vous avez une erreur ! 🚀
