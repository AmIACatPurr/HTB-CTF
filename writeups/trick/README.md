
# HTB-Trick

<img width="862" height="793" alt="Screenshot 2025-09-20 214022" src="https://github.com/AmIACatPurr/HTB-CTF/blob/main/writeups/assets/trick.jpg" />


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
