# 🖥️ Takeover — TryHackMe

| Field | Details |
|---|---|
| 🏷️ Machine | Takeover |
| 🌐 Platform | TryHackMe |
| 🎯 Difficulty | Easy |
| 🧠 Topics | Subdomain Enumeration, Virtual Host Fuzzing, TLS Certificate Inspection, Subdomain Takeover |
| 👤 Author | [Sandeep Prasad Chowdhury](https://github.com/chowdhurysandeep) |

---

## 📌 Summary

Takeover is a TryHackMe room focused on **subdomain enumeration and subdomain takeover**. The attack involves adding the target to `/etc/hosts`, fuzzing for virtual hosts using `ffuf`, inspecting TLS certificates to find hidden subdomains, and exploiting an unclaimed AWS S3 bucket linked to a subdomain to capture the flag.

---

## 🔍 1. Reconnaissance & Setup

### Add Target to /etc/hosts

The target runs virtual hosts — the domain must be added to `/etc/hosts` first:

```bash
echo -n -e "<TARGET_IP>\tfuturevera.thm" >> /etc/hosts
```

Browsing to `https://<TARGET_IP>` shows a self-signed certificate warning:

```
Error code: MOZILLA_PKIX_ERROR_SELF_SIGNED_CERT
```

Accept the risk and continue. The main site at `futurevera.thm` shows the **FutureVera** homepage.

---

## 🔎 2. Subdomain Enumeration — ffuf (Round 1)

First pass with a smaller wordlist to identify the pattern:

```bash
ffuf -u https://futurevera.thm \
  -H "Host: FUZZ.futurevera.thm" \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

Many results returned with **Size: 4605** — this is the default response size. Filter it out in the next pass.

---

## 🔎 3. Subdomain Enumeration — ffuf (Round 2)

Use a larger wordlist and filter out the default size:

```bash
ffuf -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt \
  -H "Host: FUZZ.futurevera.thm" \
  -u https://<TARGET_IP> \
  -fs 4605
```

**Valid subdomains found:**

| Subdomain | Status | Size |
|---|---|---|
| `blog` | 200 | 3838 |
| `support` | 200 | 1522 |

---

## 🔐 4. TLS Certificate Inspection

Add discovered subdomains to `/etc/hosts`:

```bash
echo -n -e "<TARGET_IP>\tblog.futurevera.thm" >> /etc/hosts
echo -n -e "<TARGET_IP>\tsupport.futurevera.thm" >> /etc/hosts
```

Browse to `https://support.futurevera.thm` and inspect the **TLS certificate**.

Under **X509v3 Subject Alternative Name**, a hidden subdomain is revealed:

```
X509v3 Extended Key Usage:
    TLS Web Server Authentication
X509v3 Subject Alternative Name:
    DNS:secrethelpdesk934752.support.futurevera.thm
```

**Hidden subdomain found:** `secrethelpdesk934752.support.futurevera.thm`

---

## 💥 5. Subdomain Takeover

Add the hidden subdomain to `/etc/hosts`:

```bash
echo -n -e "<TARGET_IP>\tsecretHelpdesk934752.support.futurevera.thm" >> /etc/hosts
```

Browse to `http://secrethelpdesk934752.support.futurevera.thm`.

The subdomain points to an **unclaimed AWS S3 bucket** — the browser redirects to:

```
flag[REDACTED].s3-website-us-west-3.amazonaws.com
```

The S3 bucket does not exist / has not been claimed, showing:

```
Hmm. We're having trouble finding that site.
We can't connect to the server at flag[...].s3-website-us-west-3.amazonaws.com
```

The **flag is embedded in the S3 bucket URL itself** in the browser address bar.

✅ **Flag captured from the subdomain takeover!**

---

## 📚 6. Lessons Learned

- Always add virtual host domains to `/etc/hosts` before enumeration
- Default response sizes must be filtered with `-fs` in `ffuf` to find real results
- TLS certificates often contain **Subject Alternative Names (SANs)** that reveal hidden subdomains — always inspect certificates
- Subdomain takeover occurs when a subdomain's DNS/CNAME points to an external service (like AWS S3) that no longer exists — an attacker can claim it
- AWS S3 bucket names embedded in subdomains are a common source of subdomain takeover vulnerabilities

---

## 🛠️ Tools Used

- `ffuf` — virtual host / subdomain fuzzing
- `/etc/hosts` — manual DNS resolution for virtual hosts
- Firefox certificate inspector — TLS SAN enumeration
- SecLists (`bitquark-subdomains-top100000.txt`) — subdomain wordlist

---

*Writeup by [Sandeep Prasad Chowdhury](https://github.com/chowdhurysandeep) — for educational purposes only.*
