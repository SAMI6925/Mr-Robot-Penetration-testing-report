Mr. Robot Penetration Testing Report

Overview

This report details the end-to-end penetration test of the Mr. Robot VM, covering network reconnaissance, web exploitation (WordPress), credential attacks, and initial access.

1. Lab Environment

To maintain an isolated environment, both machines were configured on a private NAT Network.

Attacker (Kali Linux): 192.168.1.5

Target (Mr. Robot VM): 192.168.1.8

2. Reconnaissance & Host Discovery

Initial Network Scan

I started by using arp-scan to confirm the target's presence on the local network.

sudo arp-scan -l


Service & OS Detection

After identifying the IP, I performed an aggressive Nmap scan to map the attack surface and fingerprint the operating system.

sudo nmap -sV -O 192.168.1.8


Scan Results:

PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd 2.4.7
443/tcp open   ssl/http Apache httpd
Device type: general purpose
Running: Linux 3.X | 4.X


Directory Brute-forcing

To uncover hidden web directories, I utilized dirb.

sudo dirb [http://192.168.1.8](http://192.168.1.8)


3. Web Enumeration

The Breakthrough: Finding robots.txt

The directory scan flagged a sensitive file: http://192.168.1.8/robots.txt. Manually inspecting this file revealed the first major clues.

Key Findings:

fsocity.dic: A massive custom dictionary file (wordlist).

key-1-of-3.txt: The first of three hidden flags!

WordPress Discovery

The scan also identified a WordPress login portal at http://192.168.1.8/wp-login. This confirmed the primary attack vector.

4. User Enumeration

Cracking the Identity with Burp Suite

I used Burp Suite Intruder to test the fsocity.dic wordlist against the login form. By analyzing the server's error messages, I was able to distinguish between valid and invalid users.

Invalid User: Triggered a generic "Invalid" error.

Valid User ("Elliot"): Triggered a specific message: "The password you entered for the username Elliot is incorrect."

5. Password Attack

Wordlist Optimization

To ensure the brute-force attack was efficient, I cleaned the original dictionary by removing over 800,000 duplicate entries.

sort /home/kali/mrrobot/fsocity.dic | uniq > ufsocity.txt


The Attack (Hydra)

With the username Elliot and sorted wordlist ufsocity.txt I launched Hydra to crack the password.

sudo hydra -vv -l Elliot -P ufsocity.txt 192.168.1.8 http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=is Incorrect'


The Result:
Hydra successfully recovered the password, granting full administrative access to the WordPress dashboard.
 



