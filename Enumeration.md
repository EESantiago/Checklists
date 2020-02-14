## Host Discovery

Basic ping sweep with nmap 

```
nmap -sP <TARGETRANGE>
```


### Port 21 - FTP

Attempt Anonymous login 

```
ftp <IPADDRESS>
USER anonymous
# Leave the password empty
PASS 
# grab any files of interest
GET <filename>
# change the mode so you can upload binaries 
bin
# put a file on the target FTP server 
PUT <filename>
```

### Port 22 - SSH

- Name:
- Version:
- Takes-password:
- If you have usernames test login with username:username

INSERTSSHCONNECT

```
nc INSERTIPADDRESS 22
```

