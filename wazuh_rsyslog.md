## Résumé complet des commandes utilisées

### 1. Nettoyage rsyslog

```bash
# Supprimer l'ancien fichier fortigate inutile
sudo rm /etc/rsyslog.d/30-fortigate.conf

# Désactiver le fichier Google Cloud (écrivait vers /dev/console inexistant)
sudo mv /etc/rsyslog.d/90-google.conf /etc/rsyslog.d/90-google.conf.disabled
```

---

### 2. Création des fichiers rsyslog

```bash
# Fichier des modules d'entrée (imudp/imtcp) — chargés UNE SEULE FOIS
sudo nano /etc/rsyslog.d/10-inputs.conf
```
ou
```bash
# ici c'est plus simple que de configurer un fichier dans ce dossier il faut juste décommenter
sudo nano /etc/rsyslog.conf

```


```bash
module(load="imudp")
input(type="imudp" port="514")
module(load="imtcp")
input(type="imtcp" port="514")
```

```bash
# Fichier de filtrage Cisco
sudo nano /etc/rsyslog.d/30-cisco.conf
```
```bash
if ($syslogfacility-text == 'local7') and ($fromhost-ip == '137.255.48.157') then {
  action(type="omfile" file="/var/log/cisco/cisco.log")
  stop
}
```

---

### 3. Création du répertoire de logs

```bash
sudo mkdir -p /var/log/cisco
sudo chmod 750 /var/log/cisco
sudo chown root:adm /var/log/cisco
```

---

### 4. Redémarrage et vérification rsyslog

```bash
sudo systemctl restart rsyslog
sudo systemctl status rsyslog
# Validation de la config sans redémarrer
sudo rsyslogd -N1
```

---

### 5. Intégration Wazuh

```bash
# Ajouter la localfile dans ossec.conf
sudo nano /var/ossec/etc/ossec.conf
```
```xml
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/cisco/cisco.log</location>
</localfile>
```
```bash
sudo systemctl restart wazuh-manager
```

---

### 6. Commandes de vérification

```bash
# Vérifier que les paquets UDP 514 arrivent
sudo tcpdump -i any udp port 514 -n

# Suivre les logs Cisco en temps réel
sudo tail -f /var/log/cisco/cisco.log

# Vérifier que Wazuh lit le fichier
sudo tail -f /var/ossec/logs/ossec.log | grep -i cisco

# Vérifier que rsyslog écoute sur 514
sudo ss -ulnp | grep 514
```

---

### 7. Commandes routeur Cisco (GNS3)

```cisco
! Envoyer un log test
R1# send log 6 "Test syslog vers Wazuh"

! Vérifier la config logging
R1# show logging
```

---

**État final du pipeline :**
```
R1 (GNS3) → UDP 514 → NAT box → IP publique → VPS rsyslog
→ /var/log/cisco/cisco.log → Wazuh Manager → Dashboard
```

<img width="758" height="254" alt="image" src="https://github.com/user-attachments/assets/dd61d3db-7d3e-40e1-b50f-597d48ec400c" />

```
conf t
no logging host 81.17.98.133 transport udp port 1514
logging host 81.17.98.133 transport udp port 514
end
write memory
```


```
# Accès
enable
configure terminal

# Interface DHCP
interface FastEthernet0/0
 ip address dhcp
 no shutdown

# Interface statique
interface FastEthernet0/0
 ip address 192.168.50.2 255.255.255.0
 no shutdown

# Route par défaut
ip route 0.0.0.0 0.0.0.0 192.168.50.1

# Timestamps
service timestamps log datetime msec localtime show-timezone
service timestamps debug datetime msec localtime show-timezone

# Syslog
logging trap informational
logging host 81.17.98.133
logging buffered 16384 informational
logging console informational

# Auth logs
login on-failure log
login on-success log

# Archive config
archive
 log config
  logging enable
  logging size 200
  hidekeys

# Sécurité
service password-encryption
no ip http server
no ip http secure-server

# Sauvegarde
end
write memory

# Vérification
show ip interface brief
show logging
ping 81.17.98.133
send log 6 "TEST SYSLOG VERS WAZUH"

```

