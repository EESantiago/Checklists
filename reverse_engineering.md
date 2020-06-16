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
call - call a function 

REgisters x86
EAX - stores the return values from functions 
EDX
ECX - used as the loop counter, or object in C++
ESI - Source register for string instructions
EDI - destination register for string instructions
ESP - points to the top of the stack
EBP - points to the bottom of the stack
EIP - points to the next instruction the processor should execute

Try to focus purely on the CALLs to begin with. Whenever you see a CALL, try to figure out if
there are arguments being passed to it by use of the PUSH instruction. If the argument
being PUSHed is a register, look above to see what value was most recently MOVed into it.

dword - 32 bits of data (represented in brackets0
