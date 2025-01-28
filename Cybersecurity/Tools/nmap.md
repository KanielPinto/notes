

```bash
nmap [Scan Type] [Options] {target}
```

### Important Flags:

1. **Scan Types**:
   - `-sS`: TCP SYN scan (stealth scan, default).
   - `-sT`: TCP connect scan (less stealthy).
   - `-sU`: UDP scan.
   - `-sN`, `-sF`, `-sX`: Null, FIN, and Xmas scans (stealthy).

2. **Target Specification**:
   - `{target}`: IP address, hostname, or network range (e.g., `192.168.1.0/24`).

3. **Port Specification**:
   - `-p`: Specify ports (e.g., `-p 22`, `-p 1-100`, `-p 22,80,443`).
   - `--top-ports`: Scan the most common ports (e.g., `--top-ports 100`).

4. **Output**:
   - `-oN`: Save output to a text file.
   - `-oX`: Save output to an XML file.
   - `-oG`: Save output in greppable format.

5. **Timing and Performance**:
   - `-T<0-5>`: Timing template (0: slow, 5: fast).
   - `--min-rate`: Send packets at a minimum rate (e.g., `--min-rate 100`).
   - `--max-rate`: Limit packet sending rate (e.g., `--max-rate 100`).
   - `--scan-delay`: Add a delay between packets (e.g., `--scan-delay 100ms`).

6. **Fragmentation**:
   - `-f`: Fragment packets (evade detection by splitting packets into smaller pieces).

7. **Service and Version Detection**:
   - `-sV`: Probe open ports to determine service/version info.
   - `-O`: Enable OS detection.

8. **Scripting Engine**:
   - `-sC`: Run default NSE scripts.
   - `--script`: Run specific NSE scripts (e.g., `--script vuln`).

9. **Miscellaneous**:
   - `-A`: Aggressive scan (enables OS detection, version detection, script scanning, and traceroute).
   - `-v`: Increase verbosity level.
   - `-Pn`: Treat all hosts as online (skip host discovery).
   - `-n`: Disable DNS resolution.


Nmap stores its scripts on Linux at `/usr/share/nmap/scripts`.

We can search through NSE scripts using the following : 

```bash
$ grep "ftp" /usr/share/nmap/scripts/script.db

or

$ ls -l /usr/share/nmap/scripts/*ftp*
```

The same techniques can also be used to search for categories of script. For example:  
`grep "safe" /usr/share/nmap/scripts/script.db`

We can update the scriptdb with : 

```bash
$ sudo wget -O /usr/share/nmap/scripts/<script-name>.nse https://svn.nmap.org/nmap/scripts/<script-name>.nse


$ nmap --script-updatedb
```


## [NSE Scripts Docs](https://nmap.org/nsedoc/)


## 1. **Basic Network Discovery**

   ```bash
   nmap -sn 192.168.1.0/24 -oN nmap
   ```
   
   - **Flags**:
     - `-sn`: Ping scan (no port scan).

---

## 2. **Quick Service and OS Detection**

   ```bash
   nmap -sV -O -T3 192.168.1.10 -oN nmap
   ```
   
   - **Flags**:
     - `-sV`: Service version detection.
     - `-O`: OS detection.
     - `-T3`: Default timing template.

---

## 3. **Stealthy Scan with Service Detection**

   ```bash
    nmap -sS -sV -O -p 1-1000 -T1 192.168.1.10 -oN nmap
   ```

SYN scans are the default scans used by Nmap _if run with sudo permissions_. If run **without** sudo permissions, Nmap defaults to the TCP Connect scan.

   - **Flags**:
     - `-sS`: TCP SYN scan.
     - `-sV`: Service version detection.
     - `-O`: OS detection.
     - `-p 1-1000`: Scan ports 1 to 1000.
     - `-T1`: Slow and stealthy timing template.

---

## 4. **Comprehensive Scan with Scripts**

   ```bash
   nmap -A -p- -T4 192.168.1.10 -oN nmap
   ```
   
   - **Flags**:
     - `-A`: Aggressive scan (OS detection, version detection, script scanning, traceroute).
     - `-p-`: Scan all 65535 ports.
     - `-T4`: Faster scan timing.

---

## 5. **UDP Scan with Service Detection**

   ```bash
   nmap -sU -sV --top-ports 20 -T2 192.168.1.10 -oN nmap
   ```
   
   - **Flags**:
     - `-sU`: UDP scan.
     - `-sV`: Service version detection.
     - `-p 53,67,68,123,161`: Scan specific UDP ports (DNS, DHCP, NTP, SNMP).
     - `-T2`: Polite timing template to avoid overwhelming the target.

---

## 6. **Vulnerability Detection Scan**

   ```bash
   nmap --script=vuln -sV -p 80,443 -T3 192.168.1.10 -oN nmap
   ```
   
   - **Flags**:
     - `--script=vuln`: Run vulnerability detection scripts.
     - `-sV`: Service version detection.
     - `-p 80,443`: Scan HTTP and HTTPS ports.
     - `-T3`: Default timing template.

---

## 7. **Scan for Common Web Vulnerabilities**

   ```bash
   nmap --script=http-vuln*,http-enum -p 80,443 -T4 192.168.1.10 -oN nmap
   ```
   
   - **Flags**:
     - `--script=http-vuln*`: Run all HTTP vulnerability scripts.
     - `--script=http-enum`: Enumerate web directories and applications.
     - `-p 80,443`: Scan HTTP and HTTPS ports.
     - `-T4`: Aggressive timing template for faster results.

---

## 8. **Scan for SMB Vulnerabilities**

   ```bash
   nmap --script=smb-vuln* -p 445 -T1 192.168.1.10 -oN nmap
   ```
   
   - **Flags**:
     - `--script=smb-vuln*`: Run all SMB vulnerability scripts.
     - `-p 445`: Scan SMB port.
     - `-T1`: Slow and stealthy timing template to avoid detection.

---

## 9. **Scan for SQL Server Vulnerabilities**

   ```bash
   nmap --script=ms-sql* -p 1433 -T3 192.168.1.10 -oN nmap
   ```
   
   - **Flags**:
     - `--script=ms-sql*`: Run all Microsoft SQL Server-related scripts.
     - `-p 1433`: Scan SQL Server port.
     - `-T3`: Default timing template.

---

## 10. **Scan for SSL/TLS Vulnerabilities**

   ```bash
   nmap --script=ssl* -p 443 -T5 192.168.1.10 -oN nmap
   ```
   
   - **Flags**:
     - `--script=ssl*`: Run all SSL/TLS-related scripts.
     - `-p 443`: Scan HTTPS port.
     - `-T5`: Insane timing template for the fastest possible scan.

---

## 11. **Scan for FTP Vulnerabilities**

   ```bash
   nmap --script=ftp* -p 21 -T2 192.168.1.10 -oN nmap
   ```
   
   - **Flags**:
     - `--script=ftp*`: Run all FTP-related scripts.
     - `-p 21`: Scan FTP port.
     - `-T2`: Polite timing template to reduce network load.

---

## 12. **Scan for SSH Vulnerabilities**

   ```bash
   nmap --script=ssh* -p 22 -T3 192.168.1.10 -oN nmap
   ```
   
   - **Flags**:
     - `--script=ssh*`: Run all SSH-related scripts.
     - `-p 22`: Scan SSH port.
     - `-T3`: Default timing template.

---

## 13. **Scan for DNS Vulnerabilities**

   ```bash
   nmap --script=dns* -p 53 -T4 192.168.1.10 -oN nmap
   ```
   
   - **Flags**:
     - `--script=dns*`: Run all DNS-related scripts.
     - `-p 53`: Scan DNS port.
     - `-T4`: Aggressive timing template for faster results.

---

## 14. **Scan for SNMP Information**

   ```bash
   nmap --script=snmp* -p 161 -T1 192.168.1.10 -oN nmap
   ```
   
   - **Flags**:
     - `--script=snmp*`: Run all SNMP-related scripts.
     - `-p 161`: Scan SNMP port.
     - `-T1`: Slow and stealthy timing template to avoid detection.

---

## 15. **Full Network Audit**

   ```bash
   nmap -A -p- -T5 192.168.1.0/24 -oN nmap
   ```
   
   - **Flags**:
     - `-A`: Aggressive scan.
     - `-p-`: Scan all 65535 ports.
     - `-T5`: Insane timing template for the fastest possible scan.


---

## 16. **Scan for Heartbleed Vulnerability**

   ```bash
   nmap -p 443 --script=ssl-heartbleed -T3 192.168.1.10 -oN nmap
   ```
   
   - **Flags**:
     - `-p 443`: Scan HTTPS port.
     - `--script=ssl-heartbleed`: Run the Heartbleed detection script.
     - `-T3`: Default timing template.

---
