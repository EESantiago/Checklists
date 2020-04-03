First step is to determine the file type:
```
file <FILE>
```
Use wineconsole to test windows executables:
```
wineconsole
```
Use a dissaembler for further analysis, specifically finding and analyzing strings:

Binary Ninja
IDA Pro
Radare 2
Objdump

In Binary Ninja, open the file to disokay the GUI representation of the assembly code.  We can then use the strings function inthe bootom right to view the strings and the address in memory where the string is stored



