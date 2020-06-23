### Work in progress

## Wireshark 
Filter for a specific port:
```
tcp.port == <PORT>
udp.port == <PORT>
```
filter for a specific IP address
```
ip.addr == <IP_ADDRESS>
```
Extract files from a stream
```
File > Export Objects > HTTP/IMF/SMB/TFTP
```
Filter HTTP methods:
```
http.request.method==GET
http.request.method==POST
```

<br />

## tcpdump
Filter for a specific port:
```
tcpdump -r <PCAP_FILE> "port <PORT>" -n -t
```

## tshark

