## Work in Progress

### HTTP Logs

HTTP logs sometimes have tab delimited columns, os use awk to sort through them:
```
awk -F '\t' {'print $<FIELD_NUMBER'} <LOG_FILE> 
# \t is used for tab delimited fields
```
### Bro Logs 
