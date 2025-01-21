
SMTP stands for "Simple Mail Transfer Protocol". It is utilized to handle the sending of emails. In order to support email services, a protocol pair is required, comprising of SMTP and POP/IMAP. Together they allow the user to send outgoing mail and retrieve incoming mail, respectively.

The SMTP server performs three basic functions:

-  It verifies who is sending emails through the SMTP server.
-  It sends the outgoing mail
-  If the outgoing mail can't be delivered it sends the message back to the sender.

## **POP and IMAP**

POP, or "Post Office Protocol" and IMAP, "Internet Message Access Protocol" are both email protocols who are responsible for the transfer of email between a client and a mail server. The main differences is in POP's more simplistic approach of downloading the inbox from the mail server, to the client. Where IMAP will synchronize the current inbox, with new mail on the server, downloading anything new. This means that changes to the inbox made on one computer, over IMAP, will persist if you then synchronize the inbox from another computer. The POP/IMAP server is responsible for fulfilling this process.

## **Flow**

1. The mail user agent, which is either your email client or an external program. connects to the SMTP server of your domain, e.g. smtp.google.com. This initiates the SMTP handshake. This connection works over the SMTP port- which is usually 25. Once these connections have been made and validated, the SMTP session starts.  

2. The process of sending mail can now begin. The client first submits the sender, and recipient's email address- the body of the email and any attachments, to the server.  

3. The SMTP server then checks whether the domain name of the recipient and the sender is the same.

4. The SMTP server of the sender will make a connection to the recipient's SMTP server before relaying the email. If the recipient's server can't be accessed, or is not available- the Email gets put into an SMTP queue.  

5. Then, the recipient's SMTP server will verify the incoming email. It does this by checking if the domain and user name have been recognized. The server will then forward the email to the POP or IMAP server, as shown in the diagram above.  

6. The E-Mail will then show up in the recipient's inbox.

## **Enumeration**

Poorly configured or vulnerable mail servers can often provide an initial foothold into a network. We begin by fingerprinting the server. 

We can use [[nmap]] and the `auxiliary(scanner/smtp/smtp_version)` module in Metasploit to accomplish this.

We can then use the `auxiliary(scanner/smtp/smtp_enum)` module in Metasploit to enumerate users.

Alternatively, `smtp-user-enum` can be used as follows : 

```bash
❯ smtp-user-enum -t <ip-addr> -U /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -m 100
```

If a username is obtained, then we can potentially brute-force SSH with [[hydra]] if the SSH port is open.