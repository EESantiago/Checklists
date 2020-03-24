### Host Discovery

Basic ping sweep with nmap 
```
nmap -sP <TARGETRANGE>
```

Ping sweep and send the the list of IPs that are up to a seperate file
```
nmap -sP <TARGETRANGE> | grep -B1 'Host is up' | egrep -o '([0-9]{1,3}\.){3}[0-9]{1,3}' > ips.txt
```


### Port Scanning  


Basic port scan (top 1000 ports)
```
nmap -Pn <IP_ADDRESS>
```
Scan all ports on all target devices discovered previously
```
nmap -sC -sV -Pn -p- -iL ips.txt
```

Metasploit SYN scan
```
msf > db_nmap -sS -n <Target_IP> -p 0-65535
```

Banner grab a port
```
nc -nzv <IP_ADDDRESS> <PORT>
```
