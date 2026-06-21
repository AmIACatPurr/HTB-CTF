
# HTB-Trick

<img width="862" height="793" alt="Screenshot 2025-09-20 214022" src="https://github.com/AmIACatPurr/HTB-CTF/blob/main/writeups/assets/trick.jpg" />


---
## Intro

Welcome to Trick. This machine isn’t a sci-fi anomaly; it’s a monument to corporate complacency and the illusion of perimeter defense. 
It perfectly illustrates the fundamental law of the digital underground: you don’t always need a flashy zero-day exploit to burn a system down; 
you just need to read the manual better than the person who built it..

Target IP 10.129.228.112
My IP 10.10.14.138

## nmap & enumeration
We started by mapping the target's digital footprint.


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
