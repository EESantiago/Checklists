	1. Port 21 - FTP
		a. NSE Scripts
			○ nmap --script=ftp-anon.nse,ftp-proftpd-backdoor.nse,ftp-vsftpd-backdoor.nse,tftp-enum.nse,ftp-bounce.nse,ftp-libopie.nse,ftp-syst.nse,ftp-vuln-cve2010-4221.nse -p 21 <ip addr>
			○ nmap -sV -Pn -p 21 --script=ftp-anon,ftp-bounce,ftp-libopie,ftp-proftpd-backdoor,ftp-syst,ftp-vsftpd-backdoor,ftp-vuln-cve2010-4221 -oN '/root/PWK/lab_machines/10.11.1.115/scans/ftp.nse' 10.11.1.115
		b. FTP anonymous login:
			○ ftp IPADDRESS
			○ USER anonymous
			○ PASS
				1) Should now have access to files on the server
			○ GET <filename>
				1) Download file to the local host
			○ PUT <filename>
				1) Upload file to the FTP server
			○ Bin
                                Use when transferring binary files using the ftp server

