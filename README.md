# Mr. Robot Penetration Testing Report

# Overview

A penetration testing report on MR ROBOT VM CFT, demonstrating credential attack, web exploitation (WordPress), reverse shell and privilege escalation technique.

## Lab Environment

Attacker machine (Kali Linux) ip - 192.168.1.5
Target machine (Mr Robot VM)  ip - 192.168.1.8

Both the machine were configured to same NAT Network.

## Reconnaissance

### Host discovery 

Used arp-scan to find the ip address of the target machine. 

Command used "sudo arp-scan -l"

### Service and OS detection

Used command ```namp -sV -O 192.168.1.8``` to find the running services and underlying operating system.


Scan Results:

PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd 2.4.7
443/tcp open   ssl/http Apache httpd
Device type: general purpose
Running: Linux 3.X | 4.X

### Bruteforcing Directories

Used command ```sudo dirb http://192.168.1.8``` to find the hidden files and directories within the web server. 


## Web Enumeration

After the dirb scan I found a file named ```http://192.168.1.8/robots.txt``` and then I manually inspected the file.

### Key findings 

"fsocity.dic" A massive custom dictionary file (wordlist).
"key-1-of-3.txt" The first flag
 



