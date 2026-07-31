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
