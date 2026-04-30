


Bonne question ! **Ça dépend du type de feed.**

---

## Cas 1 — Le feed ajoute des IoCs de types déjà gérés

Si le nouveau feed contient des **hashes, IPs, domaines ou URLs**, tu n'as **rien à modifier** dans le script. Il suffit d'activer et fetcher le feed dans MISP, puis relancer le script :

```bash
sudo python3 /var/ossec/etc/lists/misp_export.py
```

Le script interroge MISP par **type d'attribut**, pas par feed. Donc tous les hashes de tous les feeds seront automatiquement récupérés.

---

## Cas 2 — Le feed ajoute des IoCs de nouveaux types

Si tu veux gérer par exemple des **certificats SSL**, des **emails** ou des **numéros de CVE**, là il faut ajouter une ligne dans le script :

```python
fetch_iocs("email-src", OUTPUT_DIR + "misp-emails")
fetch_iocs("vulnerability", OUTPUT_DIR + "misp-cve")
```

Et créer la règle Wazuh correspondante.

---

## Recommandation pour ton mémoire

Automatise la mise à jour avec un **cron job** pour que les listes se mettent à jour automatiquement :

```bash
crontab -e
```

Ajoute :
```bash
# Mise à jour des IoCs MISP toutes les 6 heures
0 */6 * * * python3 /var/ossec/etc/lists/misp_export.py && systemctl restart wazuh-manager
```

Tu veux qu'on configure ce cron job maintenant ?
