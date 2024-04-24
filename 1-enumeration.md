# Kioptrix

## Identify target ip

`netdiscover -r 192.168.1.0/24`

```
Currently scanning: Finished!   |   Screen View: Unique Hosts                 
                                                                               
 4 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 168               
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.1.1     84:93:b2:22:67:76      2      84  zte corporation             
 192.168.1.107   08:00:27:74:8c:bd      1      42  PCS Systemtechnik GmbH      
 192.168.1.105   c8:89:f3:bd:f9:bb      1      42  Apple, Inc. 
```

So I can know that `192.168.1.107` is the target

## Scanning with nmap

`nmap -T4 -p- -A 192.168.1.107`

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-24 22:24 +07
Nmap scan report for 192.168.1.107
Host is up (0.00031s latency).
Not shown: 65529 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 2.9p2 (protocol 1.99)
|_sshv1: Server supports SSHv1
| ssh-hostkey: 
|   1024 b8:74:6c:db:fd:8b:e6:66:e9:2a:2b:df:5e:6f:64:86 (RSA1)
|   1024 8f:8e:5b:81:ed:21:ab:c1:80:e1:57:a3:3c:85:c4:71 (DSA)
|_  1024 ed:4e:a9:4a:06:14:ff:15:14:ce:da:3a:80:db:e2:81 (RSA)
80/tcp    open  http        Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
|_http-title: Test Page for the Apache Web Server on Red Hat Linux
111/tcp   open  rpcbind     2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1          32768/tcp   status
|_  100024  1          32768/udp   status
139/tcp   open  netbios-ssn Samba smbd (workgroup: MYGROUP)
443/tcp   open  ssl/https   Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC4_64_WITH_MD5
|     SSL2_DES_64_CBC_WITH_MD5
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|_    SSL2_RC4_128_WITH_MD5
|_http-server-header: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2009-09-26T09:32:06
|_Not valid after:  2010-09-26T09:32:06
|_ssl-date: 2024-04-24T19:25:03+00:00; +3h59m59s from scanner time.
|_http-title: 400 Bad Request
32768/tcp open  status      1 (RPC #100024)
MAC Address: 08:00:27:74:8C:BD (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 2.4.X
OS CPE: cpe:/o:linux:linux_kernel:2.4
OS details: Linux 2.4.9 - 2.4.18 (likely embedded)
Network Distance: 1 hop

Host script results:
|_nbstat: NetBIOS name: KIOPTRIX, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
|_smb2-time: Protocol negotiation failed (SMB2)
|_clock-skew: 3h59m58s

TRACEROUTE
HOP RTT     ADDRESS
1   0.31 ms 192.168.1.107

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.75 seconds
parrot# nmap -T4 -p- -A -sV -O 192.168.1.107
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-24 22:27 +07
```

## Focus on web server

I used `nikto` to scan the web server

`nikto -h http://192.168.1.107`

```
+ Target IP:          192.168.1.107
+ Target Hostname:    192.168.1.107
+ Target Port:        80
+ Start Time:         2024-04-24 22:40:12 (GMT7)
---------------------------------------------------------------------------
+ Server: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
+ /: Server may leak inodes via ETags, header found with file /, inode: 34821, size: 2890, mtime: Thu Sep  6 10:12:46 2001. See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2003-1418
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ /: Apache is vulnerable to XSS via the Expect header. See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2006-3918
+ Apache/1.3.20 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
+ OpenSSL/0.9.6b appears to be outdated (current is at least 3.0.7). OpenSSL 1.1.1s is current for the 1.x branch and will be supported until Nov 11 2023.
+ mod_ssl/2.8.4 appears to be outdated (current is at least 2.9.6) (may depend on server version).
+ OPTIONS: Allowed HTTP Methods: GET, HEAD, OPTIONS, TRACE .
+ /: HTTP TRACE method is active which suggests the host is vulnerable to XST. See: https://owasp.org/www-community/attacks/Cross_Site_Tracing
+ Apache/1.3.20 - Apache 1.x up 1.2.34 are vulnerable to a remote DoS and possible code execution.
+ Apache/1.3.20 - Apache 1.3 below 1.3.27 are vulnerable to a local buffer overflow which allows attackers to kill any process on the system.
+ Apache/1.3.20 - Apache 1.3 below 1.3.29 are vulnerable to overflows in mod_rewrite and mod_cgi.
+ mod_ssl/2.8.4 - mod_ssl 2.8.7 and lower are vulnerable to a remote buffer overflow which may allow a remote shell.
+ ///etc/hosts: The server install allows reading of any system file by adding an extra '/' to the URL.
+ /usage/: Webalizer may be installed. Versions lower than 2.01-09 vulnerable to Cross Site Scripting (XSS). See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2001-0835
+ /manual/: Directory indexing found.
+ /manual/: Web server manual found.
+ /icons/: Directory indexing found.
+ /icons/README: Apache default file found. See: https://www.vntweb.co.uk/apache-restricting-access-to-iconsreadme/
+ /test.php: This might be interesting.
+ /wp-content/themes/twentyeleven/images/headers/server.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /wordpress/wp-content/themes/twentyeleven/images/headers/server.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /wp-includes/Requests/Utility/content-post.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /wordpress/wp-includes/Requests/Utility/content-post.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /wp-includes/js/tinymce/themes/modern/Meuhy.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /wordpress/wp-includes/js/tinymce/themes/modern/Meuhy.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /assets/mobirise/css/meta.php?filesrc=: A PHP backdoor file manager was found.
+ /login.cgi?cli=aa%20aa%27cat%20/etc/hosts: Some D-Link router remote command execution.
+ /shell?cat+/etc/hosts: A backdoor was identified.
+ /#wp-config.php#: #wp-config.php# file found. This file contains the credentials.
+ 8908 requests: 0 error(s) and 30 item(s) reported on remote host
+ End Time:           2024-04-24 22:40:22 (GMT7) (10 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

```


## Brute force directory to find the protential folders

I used `dirbuster` with `/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt`
and I knew from previous process that web server run with Apache so it might be PHP language as a primary backend engine, then I found an interested URL `http://192.168.1.107/usage/usage_200909.html` which I could identified this page was generated by Generated by `Webalizer Version 2.01`


## Enumerate SMB protocol
As the previous nmap scan result I found SMB protocol is enabled

```
139/tcp   open  netbios-ssn Samba smbd (workgroup: MYGROUP)
```

But I'm not sure which SMB version was using

```
Host script results:
|_nbstat: NetBIOS name: KIOPTRIX, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
|_smb2-time: Protocol negotiation failed (SMB2)
|_clock-skew: 3h59m58s
```

So I used `metasploit framework` to enumerate it by using `auxiliary/scanner/smb/smb_version` to identify SMB protocol version

```
msf](Jobs:0 Agents:0) auxiliary(scanner/smb/smb_version) >> info

       Name: SMB Version Detection
     Module: auxiliary/scanner/smb/smb_version
    License: Metasploit Framework License (BSD)
       Rank: Normal

Provided by:
  hdm <x@hdm.io>
  Spencer McIntyre
  Christophe De La Fuente

Check supported:
  No

Basic options:
  Name     Current Setting  Required  Description
  ----     ---------------  --------  -----------
  RHOSTS                    yes       The target host(s), see https://docs.metasploit.com/docs/using-metasplo
                                      it/basics/using-metasploit.html
  THREADS  1                yes       The number of concurrent threads (max one per host)

Description:
  Fingerprint and display version information about SMB servers. Protocol
  information and host operating system (if available) will be reported.
  Host operating system detection requires the remote server to support
  version 1 of the SMB protocol. Compression and encryption capability
  negotiation is only present in version 3.1.1.


View the full module info with the info -d command.

[msf](Jobs:0 Agents:0) auxiliary(scanner/smb/smb_version) >> options

Module options (auxiliary/scanner/smb/smb_version):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS                    yes       The target host(s), see https://docs.metasploit.com/docs/using-metaspl
                                       oit/basics/using-metasploit.html
   THREADS  1                yes       The number of concurrent threads (max one per host)


View the full module info with the info, or info -d command.

[msf](Jobs:0 Agents:0) auxiliary(scanner/smb/smb_version) >> set RHOSTS 192.168.1.107
RHOSTS => 192.168.1.107
[msf](Jobs:0 Agents:0) auxiliary(scanner/smb/smb_version) >> run

[*] 192.168.1.107:139     - SMB Detected (versions:) (preferred dialect:) (signatures:optional)
[*] 192.168.1.107:139     -   Host could not be identified: Unix (Samba 2.2.1a)
[*] 192.168.1.107:        - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
[msf](Jobs:0 Agents:0) auxiliary(scanner/smb/smb_version) >> 
```
it response with what software version is used for SMB protocol which is `Unix (Samba 2.2.1a)`

### try to connect with SMB client

`smbclient -L \\\\192.168.1.107\\`

```
smbclient -L \\\\192.168.1.107\\
Server does not support EXTENDED_SECURITY  but 'client use spnego = yes' and 'client ntlmv2 auth = yes' is set
Anonymous login successful
Password for [WORKGROUP\annibuliful]:

	Sharename       Type      Comment
	---------       ----      -------
	IPC$            IPC       IPC Service (Samba Server)
	ADMIN$          IPC       IPC Service (Samba Server)
Reconnecting with SMB1 for workgroup listing.
Server does not support EXTENDED_SECURITY  but 'client use spnego = yes' and 'client ntlmv2 auth = yes' is set
Anonymous login successful

	Server               Comment
	---------            -------
	KIOPTRIX             Samba Server

	Workgroup            Master
	---------            -------
	MYGROUP              KIOPTRIX
```

the interesting is `ADMIN$` sharename so I tried to connect with this sharename `smbclient \\\\192.168.1.107\\ADMIN$` but I failed to connect 

```
parrot# smbclient \\\\192.168.1.107\\ADMIN$
Password for [WORKGROUP\root]:
Server does not support EXTENDED_SECURITY  but 'client use spnego = yes' and 'client ntlmv2 auth = yes' is set
Anonymous login successful
tree connect failed: NT_STATUS_WRONG_PASSWORD
```

so I went back to `IPC$` instead `smbclient \\\\192.168.1.107\\IPC$` which is I logged in as anonymous account

```
parrot# smbclient \\\\192.168.1.107\\IPC$
Password for [WORKGROUP\root]:
Server does not support EXTENDED_SECURITY  but 'client use spnego = yes' and 'client ntlmv2 auth = yes' is set
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> help
?              allinfo        altname        archive        backup         
blocksize      cancel         case_sensitive cd             chmod          
chown          close          del            deltree        dir            
du             echo           exit           get            getfacl        
geteas         hardlink       help           history        iosize         
lcd            link           lock           lowercase      ls             
l              mask           md             mget           mkdir          
more           mput           newer          notify         open           
posix          posix_encrypt  posix_open     posix_mkdir    posix_rmdir    
posix_unlink   posix_whoami   print          prompt         put            
pwd            q              queue          quit           readlink       
rd             recurse        reget          rename         reput          
rm             rmdir          showacls       setea          setmode        
scopy          stat           symlink        tar            tarmode        
timeout        translate      unlock         volume         vuid           
wdel           logon          listconnect    showconnect    tcon           
tdis           tid            utimes         logoff         ..             
!              
smb: \> 
```
but I still not run any command to on SMB protocol

```
smb: \> ls
NT_STATUS_NETWORK_ACCESS_DENIED listing \*
smb: \>
```

## Enumterate SSH protocol

the result from nmap which is related to SSH protocol

```
22/tcp    open  ssh         OpenSSH 2.9p2 (protocol 1.99)
|_sshv1: Server supports SSHv1
| ssh-hostkey: 
|   1024 b8:74:6c:db:fd:8b:e6:66:e9:2a:2b:df:5e:6f:64:86 (RSA1)
|   1024 8f:8e:5b:81:ed:21:ab:c1:80:e1:57:a3:3c:85:c4:71 (DSA)
|_  1024 ed:4e:a9:4a:06:14:ff:15:14:ce:da:3a:80:db:e2:81 (RSA)
```

I was trying to connect directly to target but it failed

`ssh 192.168.1.107`

```
ssh 192.168.1.107
Unable to negotiate with 192.168.1.107 port 22: no matching key exchange method found. Their offer: diffie-hellman-group-exchange-sha1,diffie-hellman-group1-sha1
```

I tried again wth diffie-helllman-group1-sha1 `ssh 192.168.1.107 -oKexAlgorithms=+diffie-hellman-group1-sha1` but it failed again

```
ssh 192.168.1.107 -oKexAlgorithms=+diffie-hellman-group1-sha1 
Unable to negotiate with 192.168.1.107 port 22: no matching host key type found. Their offer: ssh-rsa,ssh-dss
```

## Researching potential vulnerabilities

these are the list of potential software which might be vulnerable

- OpenSSH 2.9p2 (protocol 1.99)
  - https://www.exploit-db.com/exploits/21402

- Unix (Samba 2.2.1a)
  - https://www.rapid7.com/db/modules/exploit/linux/samba/trans2open/
  - https://www.exploit-db.com/exploits/22468
- Webalizer Version 2.01 /usage/: Webalizer may be installed. Versions lower than 2.01-09 vulnerable to Cross Site Scripting (XSS). See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2001-0835
  - https://www.cvedetails.com/cve/CVE-2002-0180/
- Apache/1.3.20 - Apache 1.3 below 1.3.29 are vulnerable to overflows in mod_rewrite and mod_cgi.
  - https://www.exploit-db.com/exploits/19975
- mod_ssl/2.8.4 - mod_ssl 2.8.7 and lower are vulnerable to a remote buffer overflow which may allow a remote shell
  - https://www.exploit-db.com/exploits/47080
  - https://www.exploit-db.com/exploits/40347

I also can use `searchsploit` tools to search directly from exploit db eg. `searchsploit Samba2`