<img width="509" height="156" alt="image" src="https://github.com/user-attachments/assets/bd9c78fc-4b49-4970-853f-d0484259e66a" />

┌──(kali㉿kali)-[~]
└─$ nmap -p- -A -T4 10.112.181.187
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-02 04:24 -0400
Nmap scan report for 10.112.181.187
Host is up (0.031s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    2 65534    65534        4096 Jul 24  2022 pub
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.141.159
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e2:91:5c:43:c1:81:19:6e:0a:28:e8:16:78:c6:d5:c0 (RSA)
|   256 db:f8:7e:ca:5e:24:31:f9:07:57:8b:8d:74:cb:fe:c1 (ECDSA)
|_  256 40:6e:c3:a8:fb:df:15:d1:2b:9c:0f:c5:60:ba:e0:b6 (ED25519)
80/tcp open  http    Werkzeug httpd 0.9.6 (Python 2.7.9)
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
| http-cookie-flags: 
|   /: 
|     session_id: 
|_      httponly flag not set


After connection to the ftp with Anonymous login, I retrieve the notice.txt and password files.
<img width="771" height="422" alt="image" src="https://github.com/user-attachments/assets/6c368035-2682-43f0-b322-313e18c3360b" />


Looking at the notice, the password file seems to be a password recovery tool. Also we find a domain name: antisoft.thm
<img width="854" height="119" alt="image" src="https://github.com/user-attachments/assets/be1cd47b-f415-44db-a5ae-b725d70bacc3" />


Using strings, we can see the employee id is 971234596. Inputting it into the password program will give us a password.
<img width="528" height="584" alt="image" src="https://github.com/user-attachments/assets/f5e2e67e-7c84-4780-bf9d-6237c17464d4" />
<img width="445" height="111" alt="image" src="https://github.com/user-attachments/assets/0e07e3be-8da1-411c-87f7-5c331a253b29" />


Visiting the webpage will take us to a login screen, where we could possibly use the password, if we had an email for it. 
<img width="704" height="373" alt="image" src="https://github.com/user-attachments/assets/9a309c56-40c7-46a2-a217-ff7ae9282c30" />


Going to the manage databases link, will take us to this page, where we can interact with the db. We can backup the db. It will ask us for a master password. I tried using the password we got, and it worked.
<img width="1555" height="679" alt="image" src="https://github.com/user-attachments/assets/fb0f1e15-f01e-4200-962d-bf833a0ae18b" />


After unzipping the backup, I got dump.sql and a manifest.json files. From the manifest.json, I got the version of Odoo running: 10.0. 
Looking through the dump file a bit, I found a username and a hash (most likely the password we already got).
<img width="539" height="89" alt="image" src="https://github.com/user-attachments/assets/9b0a164d-b1ab-434c-840c-b72ae15b8e1e" />
<img width="1510" height="202" alt="image" src="https://github.com/user-attachments/assets/d090bccc-af14-4999-81fd-59191e2b7f37" />
<img width="1133" height="819" alt="image" src="https://github.com/user-attachments/assets/170f6dfd-d9ed-4ad8-80f5-80747992c2fb" />


After logging in I looked up exploits for this version of Odoo. I've found this one: https://www.exploit-db.com/exploits/44064

<img width="1914" height="428" alt="image" src="https://github.com/user-attachments/assets/669d2f25-df59-4feb-b50b-f270e98597f6" />
<img width="1915" height="721" alt="image" src="https://github.com/user-attachments/assets/4b026679-8cd5-49ed-82d9-86ec800d4774" />
<img width="665" height="465" alt="image" src="https://github.com/user-attachments/assets/87dfb5fe-2aae-4595-9bc6-be3d7df43835" />

It didn't seem to work with the default bash reverse shell that was in the POC. I tried around with a few different ones and this is the one that worked. 
<img width="1053" height="228" alt="image" src="https://github.com/user-attachments/assets/d9d72390-8a92-4676-8839-2e8df6027bab" />


After getting the reverse shell, we are in a docker container. I've found a file named ret.
<img width="612" height="532" alt="image" src="https://github.com/user-attachments/assets/a34fc735-b0f2-4e29-894d-d17d2d15d0e2" />
<img width="546" height="548" alt="image" src="https://github.com/user-attachments/assets/d0c332f6-5a64-4110-a3fd-39043d7b2b94" />
