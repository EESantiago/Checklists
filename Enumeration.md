##WORK IN PROGRESS*

### Host Discovery

Basic ping sweep with nmap 

```
nmap -sP <TARGETRANGE>
```


### Port 21 - FTP

Attempt anonymous login 

```
ftp <IP_ADDRESS>
USER anonymous
# Leave the password empty
PASS 
# grab any files of interest
GET <FILE>
# change the mode so you can upload binaries 
bin
# put a file on the target FTP server 
PUT <FILE>
```

Nmap ftp vulnerabilities scan

```
nmap -sV -Pn -p 21 --script=ftp-vuln* <IP_ADDRESS>
```

### Port 22 - SSH

Bruteforce with a single username and rockyou password list (can use a list of of users as well)

```
hydra -f -V -t 1 -l <USERNAME> -P /usr/share/wordlists/rockyou.txt -s 22 <ip addr> ssh

medusa -u <USERNAME> -P /usr/share/wordlists/rockyou.txt -e ns -h <ip addr> - 22 -M ssh

ncrack -vv -p 22 --user <USERNAME> -P /usr/share/wordlists/rockyou.txt <ip addr>
```
### Port 23 - Telnet
  
### Port 25 - SMTP

### Port 53 - DNS

Zone tranfer

```
fierce -dns 10.10.10.123
```

### Port 80/443 - HTTP/HTTPS

		a. Open the ip in web browser 
			i. What does it display?
			ii.  Is it a potentially vulnerable web application? 
			iii. Is it a default web server page which reveals version information?
			iv. View source and check if the server is running an application such as a CMS 






