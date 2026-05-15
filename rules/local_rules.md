## IDS MODE
```
alert icmp any any -> 8.8.8.8 any (msg: "ICMP traffic to 8.8.8.8 detected"; sid: 10000001; rev: 1;)

alert tcp any any -> any 4444 (msg: "Connection to Remote IP on Port 4444 detected"; sid: 10000002; rev: 1;)

alert tcp any any -> any 80 ( msg:"HTTP URL contains .exe";  content:"|2e|exe"; nocase; http_uri; sid:10000004; rev:1;)

alert tcp any 80 -> any any (msg:"Potential Windows executable download over HTTP"; content:"Content-Type|3A| application/x-msdownload"; nocase; http_header; sid:10000005; rev:2;)

alert tcp any 80 -> any any (msg:"HTTP payload contains DOS MZ or PE executable file signature"; file_data; content:"|4D 5A|"; depth:2; sid:10000006; rev:1;)

alert tcp any any -> any any (msg:"Potential SSLoad activity via User-Agent"; content:"User-Agent|3A| SSLoad/1.1"; http_header; nocase; sid:10000007; rev:1;)
```

## IPS MODE
```
drop tcp any any <> any 21 (msg:"Drop FTP traffic"; sid:10000003; rev:1;)
```
