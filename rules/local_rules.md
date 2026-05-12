## IDS MODE
```
alert icmp any any -> 8.8.8.8 any (msg: "ICMP traffic to 8.8.8.8 detected"; sid: 10000001; rev: 1;)

alert tcp any any -> any 4444 (msg: "Connection to Remote IP on Port 4444 detected"; sid: 10000002; rev: 1;)

alert tcp any any -> any 80 ( msg:"HTTP URL contains .exe";  content:"|2e|exe"; nocase; http_uri; sid:10000004; rev:1;)

```

## IPS MODE
```
drop tcp any any <> any 21 (msg:"Drop FTP traffic"; sid:10000003; rev:1;)
```
