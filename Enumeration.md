## Ping Sweep

Basic ping sweep with nmap 

```
nmap -sP <TARGETRANGE>
```


### Port 21 - FTP

- FTP-Name:
- FTP-version:
- Anonymous login:

INSERTFTPTEST


```
nmap --script=ftp-anon,ftp-libopie,ftp-proftpd-backdoor,ftp-vsftpd-backdoor,ftp-vuln-cve2010-4221,tftp-enum -p 21 INSERTIPADDRESS
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

