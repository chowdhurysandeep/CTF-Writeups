# 📓 CTF-Writeups

> Documented penetration testing writeups for intentionally vulnerable machines from **TryHackMe** and **VulnHub**.

![Writeups](https://img.shields.io/badge/Writeups-7-blue?style=flat-square)
![Platform](https://img.shields.io/badge/Platform-TryHackMe%20%7C%20VulnHub-red?style=flat-square)
![Category](https://img.shields.io/badge/Category-CTF%20%7C%20Pentesting-informational?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)

> Made by [Sandeep Prasad Chowdhury](https://github.com/chowdhurysandeep) | 🎓 Ethical Hacking Student  
> 🔗 [LinkedIn](https://www.linkedin.com/in/sandeep-chowdhury-661a54397)

---

## 📌 About

This repository contains hands-on writeups for CTF machines and vulnerable labs. Each writeup follows a real-world penetration testing methodology — from initial recon to root — and is documented for learning and reference.

---

## 🗂️ Writeups Index

### 🔹 TryHackMe

| # | Machine | Difficulty | Topics | Writeup |
|---|---|---|---|---|
| 1 | Mr. Robot | Medium | WordPress, Reverse Shell, PrivEsc | [📄 Read](TryHackMe/Mr.Robot.md) |
| 2 | Simple CTF | Easy | FTP, SQLi, sudo PrivEsc | [📄 Read](TryHackMe/SimpleCTF.md) |
| 3 | Takeover | Easy | Subdomain Enumeration | [📄 Read](TryHackMe/Takeover.md) |

### 🔹 VulnHub

| # | Machine | Difficulty | Topics | Writeup |
|---|---|---|---|---|
| 1 | MyExpense:1 | Medium | XSS, IDOR, Session Hijacking | [📄 Read](Vulnhub/MyExpense:1.md) |
| 2 | SickOS 1.1 | Medium | Shellshock, SMTP, PrivEsc | [📄 Read](Vulnhub/SickOS1.1.md) |
| 3 | Vulnix | Medium | NFS, SSH, sudo PrivEsc | [📄 Read](Vulnhub/Vulnix.md) |
| 4 | Skytower | Medium | SQLi, SSH Tunneling, PrivEsc | [📄 Read](Vulnhub/Skytower.md) |
| 5 | Mercy | Medium | Tomcat, FTP, PrivEsc | [📄 Read](Vulnhub/Mercy.md) |

---

## 🛠️ Tools Used

| Category | Tools |
|---|---|
| Recon | Nmap, Netdiscover, Gobuster, FFUF |
| Exploitation | Metasploit, SQLMap, Manual |
| Web Attacks | XSS, SQLi, IDOR, Session Hijacking |
| PrivEsc | LinPEAS, sudo abuse, SUID, NFS |
| Shells | Netcat, Python, Bash reverse shells |

---

## 📋 Methodology

Every writeup follows this standard workflow:

```
1. Reconnaissance & Enumeration
2. Vulnerability Identification
3. Exploitation
4. Privilege Escalation
5. Post-Exploitation & Flags
6. Summary & Lessons Learned
```

Aligned with **OWASP Testing Guide** and real-world penetration testing practices.

---

## 📁 Repository Structure

```
CTF-Writeups/
├── README.md
├── TryHackMe/
│   ├── Mr.Robot.md
│   ├── SimpleCTF.md
│   └── Takeover.md
└── VulnHub/
    ├── MyExpense1.md
    ├── SickOS.md
    ├── Vulnix.md
    ├── Skytower.md
    └── Mercy.md

```

---

## ⚠️ Disclaimer

All machines documented here are **intentionally vulnerable** and hosted on legal training platforms (TryHackMe, VulnHub). This content is shared **strictly for educational purposes**. No real-world systems were targeted or harmed.

---

## 👤 Author

**Sandeep Prasad Chowdhury**  
🔗 [GitHub](https://github.com/chowdhurysandeep) · [LinkedIn](https://www.linkedin.com/in/sandeep-chowdhury-661a54397)
