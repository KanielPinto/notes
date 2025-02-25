
To find valid formats :

```
❯ john --list=formats

❯ john --list=formats | grep sha1
```

Using rockyou :

```
❯ john --format=[Format] --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

