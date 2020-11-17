# Windows Survey and Privilege Escalation

System Information:
```
systeminfo

ver 
```

Networking:
```
type C:\windows\system32\drivers\etc\networks
```

View the current user's privileges:
```
whoami /all
whoami /privs
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



