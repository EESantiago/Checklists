# WORK IN PROGRESS

### Static Analysis

First step is to determine the file type.  This helps determine what environment the binary was designed to run and operate in:
```
file <FILE>
```
Look for plaintext (human-readable) strings in the binary:
```
strings <FILE>
```

Use wineconsole to test windows executables:
```
wineconsole
```
Use a dissaembler for static and dynamic analysis, specifically finding and analyzing strings:


IDA Pro
Radare 2
Objdump

## Binary Ninja

In Binary Ninja, open the file to display the GUI representation of the assembly code.  We can then use the strings function in the bootom right to view the strings and the address in memory where the strings are stored


## Assembly Notes

mov - move (load) data to a register prior to pushing it onto the stack for execution 
push - push onto the stack to be executed
cmp
je
add
test
shl
call

REgisters
ECX 
EAX
EDX
EBP


dword - 32 bits of data (represented in brackets0
