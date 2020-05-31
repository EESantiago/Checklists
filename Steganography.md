# Stegonagraphy and Forensics 

Stegonography challenge workflow for CTF challenges

All tookls verified to work on Kali Linux but may not be included in the base distribution

## File Examination

Determine the file type (compare to the file extension) 
```
file <FILE>
```
Read meta information of the file
```
exiftool <FILE>
```
View the printable ASCII characters within non-text files:
```
strings <FILE>
```
Examine the hex of the file:
```
hexdump -C <FILE>

xxd <FILE>
```
Verify that the magic bytes in the hex output match that of the file output / extension:

https://en.wikipedia.org/wiki/List_of_file_signatures

Commone magic byte headers and footers:

| File Extension  | Header | Footer |
| ------------- | ------------- | ------------- |
| jpg/jpeg  | FF D8 FF  | FF D9 |

#

Search for specific magic bytes in a file
```
bgrep <MAGIC_BYTES> <FILE> 
```
## Extracting Contents of a File

Verify if there are embedded/appended files:
```
binwalk -e <FILE>
```
Carve out embedded or appended files:
```
foremost <FIlE>

steghide --extract <FILE>
# may require a password
```
Can also use 'dd' to carve out the file.  Use the decimal offset from binwalk:
```
dd if="<INPUT_FILE>" bs=1 skip=<DECIMAL_OFFSET> of="<OUTPUT_FILE>"
```

### Image Files 


### Audio Files 


### Video Files 






