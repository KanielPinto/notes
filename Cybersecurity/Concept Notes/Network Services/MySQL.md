
MySQL is a relational database management system (RDBMS) based on Structured Query Language (SQL).

MySQL, as an RDBMS, is made up of the server and utility programs that help in the administration of MySQL databases.

The server handles all database instructions like creating, editing, and accessing data. It takes and manages these requests and communicates using the MySQL protocol. This whole process can be broken down into these stages:  

1. MySQL creates a database for storing and manipulating data, defining the relationship of each table.
2. Clients make requests by making specific statements in SQL.
3. The server will respond to the client with whatever information has been requested.

If we find that MySQL is running on the server. We can try to enumerate users using :

```
❯ nmap -v <ip-addr> -p 3306 --script=mysql-enum
```

We can then use [[hydra]] to potentially brute-force the MySQL password.

We can then connect to the database using :

```
❯ mysql -h <ip-addr> -u <user> -p --skip-ssl
```




10.10.157.137