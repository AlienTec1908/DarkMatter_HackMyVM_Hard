# HackMyVM: DarkMatter - Hard

![DarkMatter Icon](DarkMatter.png)

*   **Difficulty:** Hard 🔴
*   **Author:** DarkSpirit
*   **Date:** 19. Juni 2025
*   **VM Link:** [https://hackmyvm.eu/machines/machine.php?vm=DarkMatter](https://hackmyvm.eu/machines/machine.php?vm=DarkMatter)
*   **Full Report (HTML):** [Link zum vollständigen Pentest-Bericht](https://alientec1908.github.io/DarkMatter_HackMyVM_Hard/)

## Overview

This report documents the penetration testing process of the "DarkMatter" virtual machine from HackMyVM, rated as a Hard difficulty challenge. The objective was to identify and exploit vulnerabilities to gain root access to the system. The challenge involved thorough web enumeration, leveraging information disclosure for initial access, and a complex privilege escalation vector using a known Linux Kernel exploit.

## Methodology

The approach involved detailed reconnaissance and web enumeration to map the attack surface, exploiting critical web application vulnerabilities for an initial foothold, and finally escalating privileges through a Kernel-level exploit.

### Reconnaissance & Web Enumeration

1.  **Host Discovery:** Identified the target IP (192.168.2.54) using `arp-scan` and configured the hostname `darkmatter.hmv` in `/etc/hosts`.
2.  **Port Scanning (Nmap):** Discovered open ports 22 (SSH - OpenSSH 8.4p1) and 80 (HTTP - Apache 2.4.51). Identified the OS as Debian Linux.
3.  **Web Application Analysis (Gobuster, Nikto):** Enumerated the webserver (Apache 2.4.51). Found `index.html`, `robots.txt`, and critically, the `phpmyadmin` directory (phpMyAdmin 4.8.1 found in README). Nikto reported missing security headers and an outdated Apache version.
4.  **Information Disclosure (`robots.txt`, `/p4ssw0rd.txt`):** Found an entry in `robots.txt` pointing to `/p4ssw0rd.txt`. Accessed this file directly via HTTP, which contained the cleartext password `th3-!llum!n@t0r`.

### Initial Access

Initial access was achieved by leveraging an unauthenticated Remote Code Execution vulnerability in the identified phpMyAdmin version.

1.  **phpMyAdmin Login Attempt:** Attempted to log into phpMyAdmin with the found password `th3-!llum!n@t0r` using standard usernames like `root` (failed).
2.  **Exploit Search:** Used `searchsploit` to find known vulnerabilities for phpMyAdmin 4.8.1. Identified an unauthenticated RCE exploit (CVE-2018-12613) (`php/webapps/50457.py`).
3.  **Remote Code Execution (phpMyAdmin RCE):** Downloaded the Python exploit script and executed it, specifying the target details and a command (`whoami`). Successfully executed commands as the `www-data` user.
4.  **Obtaining a Shell:** Modified the exploit script command to execute a bash reverse shell payload, connecting back to a Netcat listener on the attacker machine. Successfully obtained an interactive shell as the `www-data` user.

### Privilege Escalation

From the `www-data` shell, privilege escalation was achieved by exploiting a known Linux Kernel vulnerability.

1.  **System Enumeration (as `www-data`):** Explored the file system. Found the user `darkenergy` in `/home/`. Could not access `darkenergy`'s home directory or other standard restricted areas. `sudo -l` required a password. No immediately exploitable SUID binaries were found.
2.  **Information Gathering (`/opt/`):** Discovered the `/opt/` directory contained `note.txt` and `website.zip`. Both were readable by `www-data`.
3.  **Hint from `note.txt`:** Read `/opt/note.txt`, finding the hint "`www-data can read root's important.txt file but idk how ;(`". This suggested a method for `www-data` to bypass file permissions to read a specific root-owned file (`/root/important.txt`).
4.  **OS & Kernel Version Check:** Identified the system as Debian 11 (bullseye) with Linux Kernel `5.10.0-9-amd64` (from `/etc/*rel*` and `uname -a`).
5.  **Kernel Vulnerability Identification:** The Kernel version 5.10.x is known to be vulnerable to the **"Dirty Pipe" Local Privilege Escalation** (CVE-2022-0847), which allows overwriting arbitrary files in the page cache, enabling permission bypass for reading and writing.
6.  **Dirty Pipe Exploit:** Located a C-language Proof-of-Concept (PoC) exploit for Dirty Pipe (e.g., Exploit-DB 50808).
7.  **Exploit Compilation & Transfer:** Compiled the C exploit code on the attacker machine (`gcc exploit.c -o expl -static` for static linking to avoid GLIBC issues). Hosted the compiled binary on a local HTTP server and downloaded it to the target system's `/tmp/` directory using the `www-data` shell (`wget`).
8.  **Executing the Exploit:** Gave execute permissions to the downloaded binary (`chmod +x expl-static`). Executed the exploit (`./expl-static /usr/bin/passwd`), targeting a standard SUID binary (`/usr/bin/passwd`) to inject a small SUID shell into `/tmp/sh`.
9.  **Root Access:** The exploit successfully ran. Executing the injected SUID shell (`/tmp/sh`) resulted in obtaining a root shell.

### Flags

Both the userFlag.txt and root.txt flags were successfully retrieved after gaining the respective privileges.

*   User Flag: `4811162d4b5326c7432d29429ca6491b` (Found in `/home/darkenergy/userFlag.txt` after gaining root)
*   Root Flag: `b1946ac92492d2347c6235b4d2611184` (Found in `/root/flag.txt` after gaining root)

---

[Link zum vollständigen Pentest-Bericht](https://alientec1908.github.io/DarkMatter_HackMyVM_Hard/)
