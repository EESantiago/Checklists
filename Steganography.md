## WORK IN PROGRESS*

Stegonography challenge workflow for CTF challenges

All tookls verified to work on Kali Linux but may not be included in the base distribution

### File Examination

Determine the file type (compare to the file extension) 
```
file <FILE>
```
Read meta information of the file
```
exiftool <FILE>
```
View the printable characters within non-text files
```
strings <FILE>
```
Examine the hex of the file
```
hexdump -C <FILE>

xxd <FILE>
```
verify that the magic bytes in the hex output match that of the file output / extension:

https://en.wikipedia.org/wiki/List_of_file_signatures

Search for specific magic bytes in a file
```
bgrep <MAGIC_BYTES> <FILE> 
```
### Extracting contents of a file

Verify if there are embedded/appended files
```
binwalk <FILE>
```
Carve out embedded or appended files
```
foremost <FIlE>

steghide --extract <FILE>
# may require a password
```


### Image Files 


### Audio Files 


### Video Files 




