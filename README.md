Linux, Security Tools & Reconnaissance: Learning Notes
Name: Abbas Khan  
Reg No: 25849  
Module 2 (Day 6 to Day 10)
Note: the lab results below are sample values typical of a Kali Linux + Metasploitable 2 setup. Replace them with your own command outputs and screenshots before you submit.
Part A: Linux Fundamentals
1. Files and Directories
1.1 Filesystem Layout
Directory
Purpose
/
Root of the entire filesystem
/home
Personal folders of normal users
/root
Home folder of the root (admin) user
/etc
System-wide configuration files (e.g., /etc/passwd, /etc/shadow)
/var
Variable data such as logs (/var/log)
/tmp
Temporary files, cleared often
/bin, /usr/bin
Command binaries
/dev
Device files
/proc
Live information about processes and the kernel
1.2 Essential Commands
Command
What it does
pwd
Show current directory
ls -la
List all files with details and hidden files
cd <dir>
Change directory
mkdir, rmdir
Create / remove directories
touch <file>
Create an empty file
cp, mv, rm
Copy, move/rename, delete
cat, less, head, tail
View file contents (tail -f follows a log live)
grep <text> <file>
Search for text inside files
find / -name <file>
Search for files by name
man <command>
Open the manual for a command
history
Show previous commands
Handy tricks: pipes (ls | grep txt), redirection (> overwrite, >> append), and wildcards (*.txt).
2. Users, Groups and Permissions
2.1 Users and Groups
• Every user has a UID; every group has a GID.
• /etc/passwd lists users, and /etc/shadow stores password hashes (readable only by root).
• root (UID 0) has unlimited power. Normal users use sudo for admin tasks.
Command
Purpose
whoami, id
Show current user and group IDs
useradd <name> / adduser <name>
Create a user
passwd <name>
Set or change a password
usermod -aG <group> <user>
Add a user to a group
groups <user>
Show a user's groups
su - <user>
Switch user
sudo <command>
Run a command as root
2.2 File Permissions
Running ls -l shows something like -rwxr-xr-- 1 alice staff 120 Oct 12 notes.sh.
• First character: file type (- file, d directory, l link).
• Next nine characters: three sets of r (read), w (write), x (execute) for owner, group, others.
Permission
Number
Meaning
r
4
Read
w
2
Write
x
1
Execute
Octal examples:
• 755 = rwxr-xr-x (owner full; group and others read/execute)
• 644 = rw-r--r-- (typical text file)
• 600 = rw------- (private file, e.g., SSH keys)
• 777 = everyone can do everything (dangerous, avoid)
Commands:
• chmod 754 file or chmod u+x file changes permissions.
• chown user:group file changes ownership.
Special permissions: SUID (runs with the file owner's rights), SGID, and the sticky bit (e.g., /tmp). Misconfigured SUID binaries are a classic privilege escalation path (find them with find / -perm -4000 2>/dev/null).
3. Processes
A process is a running program, identified by a PID.
Command
Purpose
ps aux
List all running processes
top / htop
Live view of CPU and memory use
kill <PID>
Ask a process to stop (SIGTERM)
kill -9 <PID>
Force stop (SIGKILL)
jobs, bg, fg, &
Manage background and foreground jobs
ss -tulpn / netstat -tulpn
Show listening ports and the process using them
systemctl status <service>
Check a service
Security relevance: unexpected processes or unknown listening ports can be a sign of compromise.
4. Linux Security Basics
• Keep the system updated: sudo apt update && sudo apt upgrade.
• Follow least privilege: avoid using root for daily work.
• Disable unused services and close unused ports.
• Use SSH keys and disable password login and root login over SSH.
• Use a firewall (ufw, iptables).
• Review logs: /var/log/auth.log (logins), /var/log/syslog.
• Check for risky permissions (world-writable files, SUID binaries).
• Use strong password policies and consider tools like fail2ban against brute force.
Part B: Reconnaissance
5. What is Reconnaissance?
Reconnaissance (recon) is the information-gathering phase of a security assessment. The more you learn about a target, the better you can understand its attack surface and its risks.
5.1 Passive vs Active Recon
Feature
Passive Recon
Active Recon
Interaction with target
None (uses public sources)
Direct (sends traffic to the target)
Detection risk
Very low
Higher, can be logged or blocked
Examples
WHOIS, DNS records, search engines, certificate transparency, public code repos
Ping sweeps, port scans, banner grabbing, web probing
Permission needed?
Generally public data
Yes, always
6. Passive Recon and DNS / Domain Information Gathering
6.1 What to Look For
• Domain ownership and registration details
• IP ranges and hosting providers
• Subdomains and technologies used
• Email formats and public employee information
• Exposed files, leaked credentials or public repositories
6.2 Tools and Commands
Tool
Example
Purpose
WHOIS
whois example.com
Registrar, creation and expiry dates, name servers
nslookup
nslookup example.com
Basic DNS lookup
dig
dig example.com ANY, dig MX example.com, dig NS example.com
Detailed DNS records
host
host example.com
Quick lookup
Reverse lookup
dig -x <IP>
Find the name for an IP
Certificate search
crt.sh (website)
Find subdomains from SSL certificates
Search engines
Google dorks such as site:example.com filetype:pdf
Find exposed information
theHarvester / Recon-ng
theHarvester -d example.com -b all
Collect emails and subdomains
6.3 DNS Record Types (Recap)
• A / AAAA: IPv4 / IPv6 address
• MX: mail servers
• NS: name servers
• CNAME: alias
• TXT: text data (SPF, domain verification)
• SOA: zone authority information
Zone transfer (AXFR): a misconfigured DNS server may hand over all its records. Test only on your own domain or lab: dig axfr example.com @<nameserver>.
7. Active Recon and Network Enumeration
Enumeration means actively extracting detailed information about live hosts, open ports, services, users and shares.
7.1 Typical Workflow
1. Find live hosts: nmap -sn <subnet>
2. Find open ports: nmap <target>
3. Identify services and versions: nmap -sV <target>
4. Detect OS and run default scripts: nmap -A <target>
5. Enumerate specific services: web (directories, headers), SMB, FTP, SSH banners.
6. Document everything with timestamps and saved output.
7.2 Useful Commands
Command
Purpose
ping -c 4 <target>
Check if host responds
nmap -sn 192.168.56.0/24
Host discovery
nmap -p- <target>
Scan all 65535 ports
nmap -sV -sC <target>
Version detection plus default scripts
nmap -oN scan.txt <target>
Save output to a file
nc -nv <target> 80
Manual banner grab with netcat
curl -I http://<target>
View HTTP response headers
Part C: Essential Security Tools
8. Nmap
Purpose: network discovery and port/service scanning.
Option
Meaning
-sn
Ping scan only (no ports)
-sS
TCP SYN ("stealth") scan
-sT
TCP connect scan
-sU
UDP scan
-sV
Service version detection
-O
OS detection
-A
Aggressive: OS, version, scripts, traceroute
-p 80,443 / -p-
Specific ports / all ports
-T0 to -T5
Timing (speed)
--script <name>
Run NSE scripts
-oN, -oX
Save as normal text / XML
Port states: open, closed, filtered (blocked by firewall), unfiltered, open|filtered.
9. Wireshark
Purpose: capture and analyze packets.
• Select a network interface and start the capture.
• Capture filters limit what is captured. Display filters limit what is shown.
Display filter
Shows
ip.addr == 192.168.56.101
Traffic to or from an IP
tcp.port == 80
Traffic on a port
http
HTTP traffic
dns
DNS queries and replies
tcp.flags.syn == 1 && tcp.flags.ack == 0
SYN packets (useful to see scans)
tls
TLS handshakes
Useful features: Follow TCP Stream, Statistics > Conversations, Protocol Hierarchy, saving captures as .pcapng.
Security note: HTTP, FTP and Telnet send data in plain text, so credentials can be read in a capture. Only capture traffic on networks you own or are authorized to monitor.
10. Burp Suite
Purpose: intercept and test web application traffic through a proxy.
• Proxy: intercepts browser requests so you can view and modify them. Browser proxy is set to 127.0.0.1:8080.
• Repeater: resend and edit a single request to see how the server responds.
• Intruder: automate many request variations (rate-limited in the Community edition).
• Target / Site map: shows the structure of the application.
• Decoder, Comparer: encode/decode data and compare responses.
Basic workflow:
1. Configure the browser to use the Burp proxy.
2. Install Burp's CA certificate to inspect HTTPS.
3. Turn Intercept on and browse the lab app (e.g., DVWA).
4. Inspect requests (cookies, parameters, headers) and send interesting ones to Repeater.
11. OWASP ZAP
Purpose: free, open-source web application vulnerability scanner and proxy.
• Quick Start / Automated Scan: enter the target URL and click Attack.
• Spider / AJAX Spider: crawls the site to map it.
• Active Scan: actively tests for vulnerabilities (only on your own lab target).
• Passive Scan: analyzes traffic without attacking (safe).
• Alerts: results ranked High, Medium, Low, Informational.
• Reports: export HTML, XML or JSON reports.
Typical findings: missing security headers, insecure cookies, XSS, SQL injection, information disclosure.
12. Tool Comparison
Tool
Layer / Target
Best for
Active or passive
Nmap
Network
Finding hosts, ports, services
Active
Wireshark
Network traffic
Seeing what is on the wire
Passive (capture)
Burp Suite
Web app
Manual web testing and request tampering
Both
OWASP ZAP
Web app
Automated web vulnerability scanning
Both
Part D: Controlled Reconnaissance Lab Record
Safety rule: Only test systems you own or have written permission to test. Keep the lab on an isolated Host-Only or NAT Network. Never point these tools at real websites, school systems or other people's devices.
Lab 1: Linux Practice
Objective: Get comfortable with the shell, permissions, users and processes.
Tasks:
1. Create a folder recon-lab with files and change permissions (chmod 600, chmod 755).
2. Create a test user and add it to a group; verify with id and groups.
3. List running processes and find which process is listening on a port (ss -tulpn).
4. Read the last 20 lines of /var/log/auth.log.
Command run
Output / screenshot
What I learned
mkdir recon-lab && cd recon-lab && touch notes.txt
Folder and empty file created
Basic navigation and file creation
chmod 600 notes.txt then ls -l notes.txt
-rw------- 1 kali kali 0 notes.txt
600 means only the owner can read and write
sudo useradd -m tester then id tester
uid=1001(tester) gid=1001(tester) groups=1001(tester)
Every user has a UID and GID; groups control shared access
ss -tulpn
tcp LISTEN 0 128 0.0.0.0:22 users:(("sshd",pid=612))
Shows which process is listening on which port
sudo tail -n 20 /var/log/auth.log (or journalctl -u ssh on newer Kali)
Recent sudo and SSH login events
Logs show who logged in and when
Lab 2: Lab Environment Setup
Components:
• Attacker machine: Kali Linux VM
• Target machine: Metasploitable 2, DVWA or OWASP Juice Shop VM
• Network: Host-Only / NAT Network (no exposure to the internet)
• Snapshots taken before testing: yes
Addresses:
• Kali IP: 192.168.56.102
• Target IP: 192.168.56.101
• Subnet: 192.168.56.0/24
Screenshot: (insert network settings and ip a output)
Lab 3: Passive Recon
Target: a domain I own, or scanme.nmap.org (explicitly allowed for testing).
Technique
Command / tool
Key findings
WHOIS
whois scanme.nmap.org
Registrar, creation and expiry dates, and name servers; owner details are often privacy-protected
DNS records
dig scanme.nmap.org ANY, dig MX, dig NS
A record gives the IP address; NS lists the authoritative name servers; MX shows whether the domain handles mail
Reverse DNS
dig -x <IP>
PTR record maps the IP back to a hostname
Subdomains
crt.sh / theHarvester
Certificate logs and public sources can reveal subdomains without touching the target
Search engine dorks
site:<domain>
Reveals indexed pages and exposed files
Analysis: What can an attacker learn without ever touching the target? Without sending a single packet to the target, an attacker can learn who owns the domain, its IP addresses, name and mail servers, subdomains and indexed files. That is enough to map the attack surface before any scan.
Lab 4: Active Recon and Enumeration with Nmap
Commands run on the lab target:
nmap -sn <subnet>
nmap -sV -sC <target IP>
nmap -p- <target IP>
nmap -A -oN scan.txt <target IP>
Results:
Port
State
Service
Version
Risk note
21
open
ftp
vsftpd 2.3.4
Version has a known backdoor; remove or upgrade
22
open
ssh
OpenSSH 4.7p1
Outdated; update and restrict access
23
open
telnet
Linux telnetd
Plain-text login; replace with SSH
25
open
smtp
Postfix smtpd
Check for open relay and user enumeration
80
open
http
Apache httpd 2.2.8
Outdated web server hosting vulnerable test apps
139/445
open
netbios-ssn
Samba smbd 3.x
Old Samba; enumerate shares and patch
3306
open
mysql
MySQL 5.0.51a
Database reachable from the network; bind to localhost
5900
open
vnc
VNC
Weak or no authentication risk
Screenshot: (insert Nmap output)
Lab 5: Traffic Analysis with Wireshark
Tasks:
1. Start a capture on the lab interface.
2. Run an Nmap scan and spot the SYN packets with tcp.flags.syn == 1.
3. Browse an HTTP page on the lab target and use Follow TCP Stream to read the plain-text request.
4. Save the capture as lab-capture.pcapng.
Observations: The Nmap scan appeared as many SYN packets from one source to different ports in a short time. Closed ports answered with RST and open ports with SYN-ACK. Following the TCP stream of the HTTP page showed the request and response in readable plain text.
Screenshot: (insert capture and filter)
Lab 6: Web Testing with Burp Suite
Tasks:
1. Set the browser proxy to 127.0.0.1:8080.
2. Intercept a login request on the lab web app.
3. Send the request to Repeater and change a parameter.
4. Note the cookies, headers and parameters used.
Observations: The intercepted login request showed the username, password and session cookie as plain parameters. In Repeater, changing a parameter changed the server's response, which shows why input must be validated on the server.
Screenshot: (insert intercepted request)
Lab 7: Web Scanning with OWASP ZAP
Tasks:
1. Run a passive scan, then an active scan, on the lab web app only.
2. Review the alerts and export the report.
Alert
Risk level
Description
Suggested fix
SQL Injection
High
User input in a parameter reaches the database query without sanitizing
Use parameterized queries
Cross Site Scripting (Reflected)
High
Script typed into a field is returned in the page and runs in the browser
Encode output and validate input
Missing Anti-clickjacking Header
Medium
Page can be loaded inside a frame on another site
Add X-Frame-Options or frame-ancestors
Content Security Policy Header Not Set
Medium
No policy limiting where scripts and content can load from
Add a Content-Security-Policy header
Cookie Without HttpOnly / SameSite
Low
Session cookie can be read by scripts or sent cross-site
Set HttpOnly, Secure and SameSite flags
Server Leaks Version Information
Low
Response header reveals the web server version
Hide version details in the server config
Screenshot: (insert ZAP alerts)
Findings Summary
#
Finding
Evidence (tool / screenshot)
Severity
Recommendation
1
Outdated vsftpd 2.3.4 with a known backdoor
Nmap -sV scan (Lab 4)
High
Remove or upgrade the FTP service
2
Telnet enabled, credentials sent in plain text
Nmap scan and Wireshark TCP stream (Labs 4 and 5)
High
Disable Telnet and use SSH
3
Web app vulnerable to SQL injection and XSS
ZAP alerts and Burp Repeater (Labs 6 and 7)
High
Use parameterized queries and output encoding
4
Missing security headers and weak cookie flags
ZAP report (Lab 7)
Medium
Add CSP and X-Frame-Options; set HttpOnly, Secure, SameSite
5
Unnecessary open ports (MySQL, VNC, SMB)
Nmap all-ports scan (Lab 4)
Medium
Close unused ports and add a firewall
Key Takeaways
• Linux permissions and least privilege are the foundation of system security.
• Passive recon reveals a lot without ever touching the target, so public information must be managed carefully.
• Active recon is detectable and always needs permission.
• Nmap finds the doors, Wireshark shows the traffic, and Burp Suite and ZAP test the web application behind them.
• Good documentation (commands, outputs, screenshots, fixes) is as important as the tools.
Challenges & Reflection
• Challenges I faced: Getting both VMs onto the same Host-Only network and finding the right IP range.
• How I solved them: Checked the adapter settings, then confirmed connectivity with `ip a`, `ping` and `nmap -sn`.
• What I want to learn next: Burp Intruder and ZAP scripting, then basic exploitation of known vulnerabilities in the lab.
