# Service Enumeration

Checklist for enumerating services found while conducting port scans.  All tools verified to work on Kali Linux 2020 but may not be included in the base distribution.

<br />

## Port 21 - File Transfer Protocol (FTP)

Bruteforce with a single username and rockyou password list:
```
hydra -f -V -t 1 -l <USERNAME> -P /usr/share/wordlists/rockyou.txt 192.168.2.62 -s 21 <IP_ADDRESS> ftp
```

Attempt an FTP anonymous login:

```
ftp <IP_ADDRESS>
USER anonymous 
PASS <EMPTY>

# Common FTP commands:
get <FILE>
bin - change mode to upload/download binaries
pasv - upload/download ascii files 
put <FILE> - put a file on the target FTP server 
```

Can also navigate to the FTP server in a web browser to easily browse the contects of the server:
```
ftp://<IP_ADDRESS>
```

Use nmap scripts to check for common FTP vulnerabilities:

```
nmap -sV -Pn -p 21 --script=ftp-vuln* <IP_ADDRESS>
```

<br />

## Port 22 - Secure Shell (SSH)

Bruteforce with a single username and rockyou password list:

```
hydra -f -V -t 1 -l <USERNAME> -P /usr/share/wordlists/rockyou.txt -s 22 <ip addr> ssh

medusa -u <USERNAME> -P /usr/share/wordlists/rockyou.txt -e ns -h <ip addr> - 22 -M ssh

ncrack -vv -p 22 --user <USERNAME> -P /usr/share/wordlists/rockyou.txt <ip addr>
```

Secure copy files between machines:
```
#Local > Remote
scp <FILE> <USERNAME>@1<IP_ADDRESS>:/<DIRECTORY>

#Remote > Local
scp <USERNAME>@<IP_ADDRESS>:/<REMOTE_DIRECTORY>/<FILE> /<LOCAL_DIRECOTRY>
```
[SSH Port Forwarding:](https://help.ubuntu.com/community/SSH/OpenSSH/PortForwarding)

```
#Local
ssh -L <LOCAL_PORT>:<TARGET_IP>:<TARGET_PORT> <PIVOT_USERNAME>@<PIVOT_IP> -NT

#Remote
ssh -R <LOCAL_PORT>:<TARGET_IP>:<TARGET_PORT> <REMOTE_USERNAME>@<REMOTE_IP> -NT

#Dynamic (9050 is standard proxychains port)
ssh -D 9050 <PIVOT_USERNAME>@<PIVOT_IP> -NT

proxychains nmap <TARGET_IP>
```

<br />

## Port 23 - Telnet

Bruteforce with a single username or username list and rockyou password:
```
hydra -f -V -t 1 -l <USERNAME> -P /usr/share/wordlists/rockyou.txt -s 23 <IP_ADDRESS> telnet

nmap -Pn -p 23 --script=telnet-brute --script-args userdb=myusers.lst,passdb=/usr/share/wordlists/rockyou.txt,telnet-brute.timeout=8s <IP_ADDRESS>
```
  
<br />  
  
## Port 25 - Simple Mail Transfer Protocol (SMTP)

nmap scripts:;
```
nmap --script=smtp-ntlm-info.nse,smtp-vuln-cve2010-4344.nse,smtp-commands.nse,smtp-open-relay.nse,smtp-vuln-cve2011-1720.nse,smtp-enum-users.nse,smtp-strangeport.nse,smtp-vuln-cve2011-1764.nse -p 25 <IP_ADDRESS>
```
Brute force login;;
```
hydra -P /usr/share/wordlistsnmap.lst x.x.x.x smtp –V
```

Enumerate user accounts via the SMTP service: 
```
smtp-user-enum -M VRFY -U /usr/share/metasploit-framework/data/wordlists/unix_users.txt -t <IP_ADDRESS>
```
Connect to the SMTP server:
```
nc -nv <IP_ADDRESS> 25
telnet <IP_ADDRESS> 25

# list available commands:
HELP

# verify usernames once connected:
VRFY root
EXPN root

# login to view emails
USER <username@example.com>
PASS <PASSWORD>

# list and view emails
LIST
RETR <EMAIL #>
```


<br />

## Port 53 - Domain Name Service (DNS)

Use the tartget machine as the DNS server and see if it will resolve the hostname:
```
nslookp

> SERVER <IP_ADDRESS>

> 127.0.0.1

> <IP_ADDRESS>
```
Specify to use the target DNS server as your nameserver
```
vim /etc/resolv.conf

nameserver <IP_ADDRESS>

# now ping the domain name if you have it to test that you have configured the DNS server correctly
```

Attempt a zone tranfer:;
```
fierce -dns 10.10.10.123

dig axfr @<IP_ADDRESS>

dig axfr <ZONE> @<IP_ADDRESS>
# zone is the domain name
```

Enumerate DNS records for the domain
```
dnsrecon -r <SUBNET> -n <DC_IP_ADDRESS>
```

Enumerate subdomains:;
```
gobuster -m dns -t 100 -u test.com -w /usr/share/wordlists/metasploit/namelist.txt
```

nmap scripts:
```
nmap --script=broadcast-dns-service-discovery.nse,dns-blacklist.nse,dns-zone-transfer.nse,dns-zeustracker.nse,dns-cache-snoop.nse,dns-check-zone.nse,dns-client-subnet-scan.nse,dns-fuzz.nse,dns-ip6-arpa-scan.nse,dns-nsec3-enum.nse,dns-nsec-enum.nse,dns-nsid.nse,dns-random-srcport.nse,dns-random-txid.nse,dns-recursion.nse,dns-service-discovery.nse,dns-srv-enum.nse,dns-update.nse -sV -Pn -p T:53,U:53 --open <IP_ADDRESS>
```
<br />

## Port 69 - Trivial File Transfer Protocol (TFTP)

Nmap scripts:
```
nmap -sU -Pn -p 69 --script tftp-enum.nse <IP_ADDRESS>
```
Connect to the tftp server:
```
tftp <TARGET_IP>
```
Standard tftp commands:
```
#modes
ascii
binary

#transfer files
get
put 
```

<br />

## Port 79 - Finger

Finger is a program you can use to find information about computer users.
<br/>

Use the finger command to check the information of any currently logged in users:
```
finger <IP_ADDRESS>

# get login name, real name, terminal name, write status, and home directory:
finger -l -s <IP_ADDRESS>
```
Pipe commands through finger:
```
finger "|/bin/id@<IP_ADDRESS>"
```
Can also use the nmap script to gather users on the target machine:
```
nmap -Pn -p 79 --script=finger.nse <IP_ADDRESS>
```

<br />

## Port 80/443 - HTTP/HTTPS (really a work in progress, might create another page)

nmap vuln scan:
```
nmap -sV -Pn -p 80 --script=http-vuln* <IP_ADDRESS>
```
a. Open the ip in web browser 
i. What does it display?
ii.  Is it a potentially vulnerable web application? 
iii. Is it a default web server page which reveals version information?
iv. View source and check if the server is running an application such as a CMS 

Virtual host routing - instead of navigating to the ip address in a web browser, navigate to the domaian name (if running a DNS server) to see if you get a different page
--only examines the "host filedin the GET request and send us to a different page

Burpsuite
-send the page to interceptor and change the verbs / HTTP code to se what we get 


If the page containes many files many files, do a wget -r
#need to add gobuster nikto dirb etc

Brute force URIs (directories and files) in web sites


```
gobuster -w /usr/share/wordlists/dirb/common.txt -t 30 -x html,asp,php -u http://<IP_ADDRESS>:<PORT> -o gobuster_<IP_ADDRESS>_<PORT>

dirb

nikto -h <IP_ADDRESS>:<PORT>

nikto -h 198.51.100.35 -p 443


dirsearch -u <IP_ADDRESS> -e <EXTENTIONS> -w <WORDLIST> -f -t 20

# GUI version
dirbuster&


# new gobuster
gobuster dir --wordlist /usr/share/wordlists/dirb/common.txt --url <IP_ADDRESS>

```
check for a robots.txt file:

```
w3m -dump 172.30.77.104/robots.txt
```

create a password list based on the website
```
cewl -d 2 -m 5 -w passwords.txt <URL>
```

#### WebDAV

If the server is running WebDAV, connect to it using cadaver (reference mail from PWK!!!)
```
cadaver <IP_ADDRESS>

# list the contents
dav:/> ls
```
If the server is running WebDAV, use davtest to to determine which file extensions can be uploaded to the server
```
davtest -url <IP_ADDRESS>
```

#### Wordpress
```
wpscan --url <TARGET_SITE>
```

Brute force a web portal login page and filter out the pages that contain the message presented when a login is unsuccessful
```
hydra -l <USERNAME> -P <PASSWORD_LIST> <IP_ADDRESS> http-post-form "<URL>:username=^USER^&password=^PASS^&Login=Login:<FAILED_LOGIN_MESSAGE>"
```

#### Drupal

Droopescan

For Drupal we can use a tool called droopescan. We can install using python pip installer.

    Apt-get install python-pip
    pip install droopsescan

Now we can run droopescan against the remote server. This will take a while to finish.

    droopescan scan drupal -u http://10.10.10.9

<br />

## Port 88 - Kerberos

[Kerberos cheatsheet](https://gist.github.com/TarlogicSecurity/2f221924fef8c14a1d8e29f3cb5c5c4a)

Kerberos bruteforce usernames and passwords to enumerate valid Active Directory accounts through Kerberos Pre-Authentication.  When an invalid username is requested the server will respond using the Kerberos error code KRB5KDC_ERR_C_PRINCIPAL_UNKNOWN, allowing us to determine that the user name was invalid. Valid user names will illicit either the TGT in a AS-REP response or the error KRB5KDC_ERR_PREAUTH_REQUIRED, signaling that the user is required to perform pre authentication:
```
python kerbrute.py -domain <DOMAIN> -dc-ip <IP_ADDR> -users <USER_FILE> -passwords /usr/share/wordlists/rockyou.txt -outputfile passwords.txt
```
If any of the users return '[NOT PREAUTH]', harvest the non-preauth AS_REP responses by sending an AS_REQ request to the KDC on behalf of the user:
```
python GetNPUsers.py <DOMAIN>/ -dc-ip <IP_ADDR> -usersfile <USER_FILE> -format john -outputfile hashes.txt
```
If we get a TGT, we can decrypt the user's password hash used to encrypt the session key:
```
john --format:krb5asrep --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

nmap script to enumerate users:
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

<br/> 

## Port 110 - POP3

Nmap scripts:
```
nmap -Pn -p 110 --script=pop3-ntlm-info, pop3-brute <IP_ADDRESS>
```
Connect to the POP3 server:
```
nc -nv <IP_ADDRESS> 110
telnet <IP_ADDRESS> 110

# login
USER <USERNAME>
PASS <PASSWORD>

# List and retrieve emails:
LIST
RETR
```

<br/>

## Port 111 - RPCBind

Grab the RPC information:
```
rpcinfo –p <IP_ADDRESS>
```

<br/>

## Port 135 - MSRPC

*This tool is included in enum4linux, so you do not need to run both

Attempt a connection:
```
# no username or password
rpcclient -U "" <IP_ADDRESS> -N

# guest account
rpcclient -U "guest" <IP_ADDRESS> -N
```
If a login attempt is successful, enumerate with native commands:
```
# windows version
rpcclient $> srvinfo

# current username
rpcclient $> getusername

# domain usernames
rpcclient $> enumdomusers 

# domain groups
rpcclient $> enumalsgroups domain

# min password length
rpcclient $> getdompwinfo

# privileges
rpcclient $> enumprivs
```

## Port 139/445 - Server Message Block (SMB)

nmap scripts:
```
# standard
nmap --script=smb-double-pulsar-backdoor.nse,smb-enum-sessions.nse,smb-mbenum.nse,smb-security-mode.nse,smb-enum-domains.nse,smb-enum-shares.nse,smb-os-discovery.nse,smb-server-stats,smb-enum-groups.nse,.nse,smb-print-text.nse,smb-system-info.nse,smb-enum-processes.nse,smb-protocols.nse,smb-ls.nse,smb-psexec.nse -sV -Pn -p 139,445 --open <IP_ADDRESS>

# vulnerability scripts
nmap -script=smb-vuln* -sV -Pn -p 139,445 --open <IP_ADDRESS>
nmap -script=smb-vuln* --script-args=unsafe -sV -Pn -p 139,445 --open <IP_ADDRESS>
```
Grab the Samba version using a combimation of ngrep and an smbclient null session:
```
#connect to the target with smbclient
echo exit | smbclient -L <IP_ADDRESS> -U "" -N

# dump the Samba version from the connection to the target using smbclient
ngrep -i -d <INTERFACE> 's.?a.?m.?b.?a.*[[:digit:]]' port 139
```
Grab the Samba version using metasploit:
```
msf > use auxillary/scanner/smb/smb_version
set RHOSTS <IP_ADDRESS>
run
```
Enumerate the SMB shares with enum4linux:
```
enum4linux -a <IP_ADDRESS>
```
Attempt to list out the SMB shares:
```
# try this first
smbclient -L \\\\<IP_ADDRESS>\\

# no username or password
smbclient -L <IP_ADDRESS> -U "" -N

# guest account no password
smbclient -L <IP_ADDRESS> -U "guest"%

# with valid credentials
smbclient -L <IP_ADDRESS> -U "<USERNAME>"%"<PASSWORD>"
```

If you are able to list out the shares, see what permissions you have on each share:
```
#crackmapexec
crackmapexec smb <IP_ADDRESS>
crackmapexec smb <IP ADDR> --shares
crackmapexec smb <IP ADDR> --shares -u '' -p ''

#smbap
smbmap -H <IP_ADDRESS> -u ""
smbmap -H <IP_ADDRESS> -u guest
smbmap -H <IP_ADDRESS> -u <USERNAME> -p ""
smbmap -H <IP_ADDRESS> -u <USERNAME> -p '<PASSWORD>'
```

If any of the permissions DO NOT say "NO ACCESS", attempt to list out the contents of the remote share
```
smbmap -R <SHARE> -H <IP_ADDRESS> -u <USERNAME> -p <PASSWORD>
smbmap -R <SHARE> -H <IP_ADDRESS> -u <USERNAME> -p ""
```

From the ouput, grab a directory or file of interest that you have full access to:
```
#directory
smbget -R smb:'//<IP_ADDRESS>/<SHARE>/PATH/TO/DIRECTORY' -U "<USERNAME>%<PASSWORD>"

#file
smbget smb:'//<IP_ADDRESS>/<SHARE>/PATH/TO/FILE' -U "<USERNAME>%<PASSWORD>"
```

Connect to SMB share with a limited shell:
```
smbclient //<IP_ADDRESS>/<SHARE> -U <USERNAME>

#Shell commands
help
ls
get
put
cd
```

Connect using winexe with credentials:
```
winexe -U <USERNAME>%<PASSWORD> //<IP_ADDRESS> "cmd.exe"
python psexec.py <USERNAME>:<PASSWORD>@<IP_ADDRESS>
python smbexec.py <USERNAME>:<PASSWORD>@<IP_ADDRESS>
```

#### Impacket Scripts

Brute force smb login:
```
hydra -f -V -t 1 -l <USERNAME> -P /usr/share/wordlists/rockyou.txt <IP_ADDRESS> smb
```

### Port 143 - IMAP

Nmap scripts:
```
nmap -Pn -p 143 --script=imap-brute <IP_ADDRESS>
```

<br />

## Port 161/162 (UDP) - Simple Network Managment Protocol (SNMP)
[SNMP](https://www.greycampus.com/opencampus/ethical-hacking/snmp-enumeration#:~:text=LDAP%20Enumeration-,SNMP%20Enumeration,devices%20on%20an%20IP%20network.&text=SNMP%20enumeration%20is%20used%20to,devices%20on%20a%20target%20system.)  is an application layer protocol which uses UDP protocol to maintain and manage routers, hubs and switches other network devices on an IP network.

<br />

The SNMP Management Information Base (MIB) is a database containing a formal description of all the network objects identified by a specific object identifier (OID) that can be managed using SNMP

<br />

The SNMP Community string is like a user id or password that allows access to a router's or other device's statistics.  When sending a request to an SNMP agent, if the community string is correct, the device responds with the requested information.  SNMP Community strings are used only by devices which support SNMPv1 and SNMPv2c protocol. SNMPv3 uses username/password authentication, along with an encryption key.  Two types of community strings:
* Read only: This mode permits querying the device and reading the information, but does not permit any kind of changes to the configuration. The default community string for this mode is “public.”
* Read Write: In this mode, changes to the device are permitted; hence if one connects with this community string, we can even modify the remote device ’s configurations. The default community string for this mode is “private.”

<br />

Brute force the community string:
```
nmap --script=snmp-brute -Pn -sU -p 161 <IP_ADDRESS> 
onesixtyone -c <WORDLIST> <IP_ADDRESS>
```
Once you know the community string, use snmpwalk to extract the OIDs and values from the MIB:
```
snmpwalk -c <COMMUNITY_STRING> -v<SNMP_VERSION_#> <IP_ADDRESS>
```
If you already know the OIDs of the target device, use snmpwalk to grap specific OIDs.  Some common OIDs:

**Windows**

OID | Value
------------ | -------------
Users | 1.3.6.1.4.1.77.1.2.25
Processes | 1.3.6.1.2.1.25.4.2.1.2
Open TCP Ports | 1.3.6.1.2.1.6.13.1.3
Installed Software | 1.3.6.1.2.1.25.6.3.1.2

https://resources.infosecinstitute.com/snmp-pentesting/#gref

Other tools:
```
snmp-check
snmp-set
```

<br />

## 389 - LDAP

LDAP search

<br/>

## Port 554 - Real-Time Streaming Protocol (RTSP) 

Nmap scripts:
```
nmap -sV -Pn -p 554 --script=rstp* <IP_ADDRESS>
```
## Port 1521 / 1560 / 3339 / 7778 - Oracle 

Nmap scripts:
```
# brute force SIDs
nmap -sV -Pn --script=oracle-sid-brute -p <PORT> <IP_ADDRESS>
 
# Brute force to check for default credentials
nmap -sV -Pn --script=oracle-brute -p <PORT> <IP_ADDRESS>
```
Enumerate Oracle Transparent Network Substrate (TNS)
```
tnscmd10g version -h <IP_ADDRESS>

tnscmd10g status -h <IP_ADDRESS>
```

<br/>

## Port 1433 - Microsoft SQL Server (MSSQL)

https://book.hacktricks.xyz/pentesting/pentesting-mssql-microsoft-sql-server

<br/>

## Port 2049 - Network File System (NFS)

View mount information for the NFS server (available shares)
```
# standard
showmount -e <IP_ADDRESS>

# nmap script
nmap -sV --script=nfs-showmount <IP_ADDRESS>
```
Mount the NFS to the /mnt directory
```
mount <IP_ADDRESS>:/<SHARE> /mnt
mount -t <IP_ADDRESS>:/<SHARE> /mnt
```

<br/>

## Port 27017 - MongoDB

NoSQL commands - TO DO

<br/>

## Port 3306 – MySQL

Nmap scripts:
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
View the structure of a table (fields)
```
describe <TABLE>;
```
View the records from a specific field
```
SELECT <FIELD> FROM <TABLE>;
```

## Port 3389 - Remote Desktop Protocol (RDP)

nmap scripts:
```
nmap -sV -Pn -script=rdp* –p 3389 <IP_ADDRESS>
```
Login to the RDP:
```
#Username and password
rdesktop -u <USERNAME> -p <USERNAME> <IP_ADDRESS> -g 94%

#Guest account
rdesktop -u guest -p guest <IP_ADDRESS> -g 94%
```
Brute force RDP login with a wordlist:
```
ncrack -vv --user <USERNAME> -P /usr/share/wordlists/rockyou.txt rdp://<IP_ADDRESS>
```

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

## Port 5985 - Windows Remote Managmement (WinRM)
Windows Remote Management is one component of the Windows Hardware Management features that manage server hardware locally and remotely. If you have credentials for the target machine, you can test if winrm will work with crackmapexec:
```
crackmapexec winrm <IP_ADDRESS> -u <USERNAME> -p <PASSWORD>
```
You can gain powershell execution using the [ruby win-rm shell](https://alionder.net/winrm-shell/) if you have credentials for the target machine (must have the [WinRM ruby library](https://github.com/WinRb/WinRM) installed):
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

Another method by which you can leverage WinRM is through the [evil-winrm tool](https://github.com/Hackplayers/evil-winrm). Using this, you can log in with a username and password, so long as the user is in the "Remote Management User" group.  Conveniently, you can also pass a hash instead of a password.


## References
1. https://medium.com/secjuice/hackthebox-mantis-writeup-9c2b50c4b30b

***Good article on usage here: http://pentest.tonyng.net/enumerating-user-accounts-on-linux-and-os-x-with-rpcclient/
***Great article on usage here: https://www.blackhillsinfosec.com/password-spraying-other-fun-with-rpcclient/

Good articles for what to do after logging in:
1) http://www.samlogic.net/articles/smtp-commands-reference.htm
2) https://mediatemple.net/community/products/dv/204404584/sending-or-viewing-emails-using-telnet

## Checklists Used:
1. https://bitvijays.github.io/LFF-IPS-P2-VulnerabilityAnalysis.html#pop3-port-110




