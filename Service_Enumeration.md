## WORK IN PROGRESS*

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

### Port 21 - FTP

Bruteforce with a single username and rockyou password list (can use a list  of users as well)
```
hydra -f -V -t 1 -l <USERNAME> -P /usr/share/wordlists/rockyou.txt 192.168.2.62 -s 21 <IP_ADDRESS> ftp
```

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

Bruteforce with a single username and rockyou password list (can use a list of of users as well)
```
hydra -f -V -t 1 -l <USERNAME> -P /usr/share/wordlists/rockyou.txt -s 23 <IP_ADDRESS> telnet
```
  
### Port 25 - SMTP

nmap scripts
```
nmap --script=smtp-ntlm-info.nse,smtp-vuln-cve2010-4344.nse,smtp-commands.nse,smtp-open-relay.nse,smtp-vuln-cve2011-1720.nse,smtp-enum-users.nse,smtp-strangeport.nse,smtp-vuln-cve2011-1764.nse -p 25 <IP_ADDRESS>
```

Conenct with netcat
```
nc -nv <IP_ADDRESS> 25
```

Conenct with telnet
```
telnet <IP_ADDRESS> 25
```
#need to add how to verify usernames from PWK

### Port 53 - DNS

Zone tranfer
```
fierce -dns 10.10.10.123
```

Enumerate DNS records for the domain
```
dnsrecon -r <SUBNET> -n <DC_IP_ADDRESS>
```

nmap scripts
```
nmap --script=broadcast-dns-service-discovery.nse,dns-blacklist.nse,dns-zone-transfer.nse,dns-zeustracker.nse,dns-cache-snoop.nse,dns-check-zone.nse,dns-client-subnet-scan.nse,dns-fuzz.nse,dns-ip6-arpa-scan.nse,dns-nsec3-enum.nse,dns-nsec-enum.nse,dns-nsid.nse,dns-random-srcport.nse,dns-random-txid.nse,dns-recursion.nse,dns-service-discovery.nse,dns-srv-enum.nse,dns-update.nse -sV -Pn -p T:53,U:53 --open <IP_ADDRESS>
```

### Port 80/443 - HTTP/HTTPS

a. Open the ip in web browser 
i. What does it display?
ii.  Is it a potentially vulnerable web application? 
iii. Is it a default web server page which reveals version information?
iv. View source and check if the server is running an application such as a CMS 

#need to add gobuster nikto dirb etc

Brute force URIs (directories and files) in web sites
```
gobuster -w /usr/share/wordlists/dirb/common.txt -t 30 -x html,asp,php -u http://<IP_ADDRESS>:<PORT> -o gobuster_<IP_ADDRESS>_<PORT>

dirb

nikto -h <IP_ADDRESS>:<PORT>
```

create a password list based on the website
```
cewl -d 2 -m 5 -w passwords.txt <URL>
```
If the server is running WebDAV, connect to it using cadaver
```
cadaver <IP_ADDRESS>

# list the contents
dav:/> ls
```
If the server is running WebDAV, use davtest to to determine which file extensions can be uploaded to the server
```
davtest -url <IP_ADDRESS>
```

Brute force a web portal login page and filter out the pages that contain the message presented when a login is unsuccessful
```
hydra -l <USERNAME> -P <PASSWORD_LIST> <IP_ADDRESS> http-post-form "<URL>:username=^USER^&password=^PASS^&Login=Login:<FAILED_LOGIN_MESSAGE>"
```
### Port 88 - Kerberos

```
python /opt/kerbrute.py -domain EGOTISTICAL-BANK.LOCAL -dc-ip 10.10.10.175 -users users.txt -passwords /usr/share/wordlists/rockyou.txt -outputfile sauna_passwords.txt
```
nmap script to enumerate users
```
nmap -Pn -p 88 --script=krb5-enum-users --script-args krb5-enum-users.realm=’<domain>’,userdb=users.txt <IP_ADDRESS>
```
Metasploit bruteforce users
```
msf > use Auxiliary/gather/Kerberos_enumusers
set DOMAIN <DOMAIN>
set RHOSTS <IP_ADDRESS>
set USER_FILE <USERS_FILE>
```

### Port 110 - POP3

### Port 111 - RPCBind


### Port 135 - MSRPC [1]

*This tool is included in enum4linux, so you do not need to run both

Attempt a null SMB session (no username or password) or a session with the guest account (no password)
```
rpcclient -U "" <IP_ADDRESS> -N

rpcclient -U "guest" <IP_ADDRESS> -N
```
If a login attempt is successful , enumerate the Windows version
```
rpcclient $> srvinfo
```
Grab the current username
```
rpcclient $> getusername
```
Enumerate users
```
rpcclient $> enumdomusers 
```
Enumerate domain groups
```
rpcclient $> enumalsgroups domain
```
Get the minimum password length for the domain
```
rpcclient $> getdompwinfo
```

### Port 139/445 - SMB

nmap scripts

```
nmap --script=smb-double-pulsar-backdoor.nse,smb-enum-sessions.nse,smb-mbenum.nse,smb-security-mode.nse,smb-enum-domains.nse,smb-enum-shares.nse,smb-os-discovery.nse,smb-server-stats,smb-enum-groups.nse,.nse,smb-print-text.nse,smb-system-info.nse,smb-enum-processes.nse,smb-protocols.nse,smb-ls.nse,smb-psexec.nse -sV -Pn -p 139,445 --open <IP_ADDRESS>
```

nmap vulnerability scripts
```
nmap -script=smb-vuln* -sV -Pn -p 139,445 --open <IP_ADDRESS>

nmap -script=smb-vuln* --script-args=unsafe -sV -Pn -p 139,445 --open <IP_ADDRESS>
```
Grab the Samba version using a combimation of ngrep and an smbclient null session
```
#connect to the target with smbclient
echo exit | smbclient -L <IP_ADDRESS> -U "" -N

# dump the Samba version from the connection to the target using smbclient
ngrep -i -d <INTERFACE> 's.?a.?m.?b.?a.*[[:digit:]]' port 139
```
Grab the Samba version using metasploit 
```
msf > use auxillary/scanner/smb/smb_version
set RHOSTS <IP_ADDRESS>
run
```
Enumerate the SMB shares with enum4linux
```
enum4linux -a <IP_ADDRESS>
```

Attempt to list out the SMB shares with a null session or guest account (no password)
```
smbclient -L <IP_ADDRESS> -U "" -N

smbclient -L <IP_ADDRESS> -U "guest"%
```

If you are able to list out the shares, see what permissions you have on each share:
```
smbmap -H <IP_ADDRESS> -u ""

smbmap -H <IP_ADDRESS> -u guest

smbmap -H <IP_ADDRESS> -u <USERNAME> -p ""

smbmap -H <IP_ADDRESS> -u <USERNAME> -p <PASSWORD>
```

If any of the permissions DO NOT say "NO ACCESS", attempt to list out the contents of the remote share
```
smbmap -R <SHARE> -H <IP_ADDRESS> -u <USERNAME> -p <PASSWORD>

smbmap -R <SHARE> -H <IP_ADDRESS> -u <USERNAME> -p ""
```

From the ouput, grab a directory or file of interest that you have full access to:
```
#directory
smbget -R smb:'//<IP_ADDRESS>/<SHARE>/PATH/TO/DIRECTORY' -U "<USERNAME>" -p "<PASSWORD>"

#file
smbget smb:'//<IP_ADDRESS>/<SHARE>/PATH/TO/FILE' -U "<USERNAME>" -p "<PASSWORD>"
```

Connect to SMB share with a limited shell
```
smbclient //<IP_ADDRESS>/<SHARE> -U <USERNAME>

#Shell commands
ls
get
put
```

Impacket scripts**

Brute force smb login with a single username and rockyou.txt password list 
```
hydra -f -V -t 1 -l <USERNAME> -P /usr/share/wordlists/rockyou.txt <IP_ADDRESS> smb
```

### Port 143 - IMAP

### Port 161/162 - SNMP

### 389 - LDAP

LDAP search

### Port 554 - Real-Time Streaming Protocol (RTSP) 

Nmap scripts
```
nmap -sV -Pn -p 554 --script=rstp* <IP_ADDRESS>
```
### Port 1521 / 3339 / 7778 - Oracle 

### Port 1099/1100
### Port 2049 - Network File System (NFS)

View mount information for the NFS server (available shares)
```
showmount -e <IP_ADDRESS>
```
Mount the NFS to the /mnt directory
```
mount <IP_ADDRESS>:/<SHARE> /mnt
mount -t <IP_ADDRESS>:/<SHARE> /mnt
```
### Port 27017 - MongoDB

NoSQL

### Port 3306 – MySQL

nmap scripts
```
nmap -sV -Pn -script=mysql-audit,mysql-databases,mysql-dump-hashes,mysql-empty-password,mysql-enum,mysql-info,mysql-query,mysql-users,mysql-variables,mysql-vuln-cve2012-2122 –p 3306 <IP_ADDRESS>
```
Log into the MySQL service with default credentials 
```
# remote
mysql -h <IP_ADDRESS> -u root -p

# local
mysql -u root -p
```
Show all the databases (Information_schema, mysql, and performance schema are default databases)
```
mysql >show databases;
```
View the current users and their passwords from the mysql.user table (default configuration)
```
mysql >select user, host, password from mysql.user;
```		
Connect to the database that you want to use
```
mysql >use mysql;
```
View the tables within the database you are currently connected to
```
mysql >show tables;
```

### Port 3389 - Remote Desktop Protocol (RDP)


### Port 5800/5900 - VNC

Metasploit vnc_none_auth scanner scans a range for VNC servers that do not have any authentication set on them: 
```
use auxiliary/scanner/vnc/vnc_none_auth
show options
set RHOSTS <IP_ADDRESS>
```
Login to the vnc server with a password
```
vncviewer <IP_ADDRESS>
```

### Port 5985 - Windows Remote Managmement (WinRM) 
Windows Remote Management is one component of the Windows Hardware Management features that manage server hardware locally and remotely. You can gain powershell execution using the ruby win-rm shell if you have credentials for the target machine:
```
require 'winrm'

conn = WinRM::Connection.new(
  endpoint: 'http://<IP_ADDRESS>:5985/wsman',
  user: '<DOMAIN\USERNAME>',
  password: '<PASSWORD>',
)

command=""

conn.shell(:powershell) do |shell|
    until command == "exit\n" do
        print "PS > "
        command = gets        
        output = shell.run(command) do |stdout, stderr|
            STDOUT.print stdout
            STDERR.print stderr
        end
    end    
    puts "Exiting with code #{output.exitcode}"
end
```
*must have the WInRM ruby lirary installed: https://github.com/WinRb/WinRM

## References
1. https://medium.com/secjuice/hackthebox-mantis-writeup-9c2b50c4b30b

