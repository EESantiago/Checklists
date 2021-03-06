# Stegonagraphy and Forensics 

Stegonography challenge workflow for CTF challenges

All tookls verified to work on Kali Linux but may not be included in the base distribution

<br />

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
Verify that the [magic bytes](https://en.wikipedia.org/wiki/List_of_file_signatures) in the hex output match that of the file output / extension:


Common magic byte headers and footers:

| File Extension  | Header | Footer |
| ------------- | ------------- | ------------- |
| jpg/jpeg  | FF D8 FF  | FF D9 |
| png | 89 50 4E 47 0D 0A 1A 0A | |
| mbr |  | 55 aa |
| zip| 50 4B 03 04 |  |
#

Search for specific magic bytes in a file:
```
bgrep <MAGIC_BYTES> <FILE> 
```

<br />

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

<br />

## Image Files 

Verify the integrity of PNG, JNG and MNG files:
```
pngcheck  -vtp7f <IMAGE_FILE>
```
Use tools like Fiji or GIMP to view the lower bits of an image

Stego tools:

```
stegsolve
```
<br />

## Audio Files 

sonic visualizer

## Video Files 

ffmpeg







