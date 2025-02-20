
[[TryHackMe]] Room

**Date**: 17/02/25
**Target IP**: 10.10.110.61

---

> [!NOTE] Prompt
>This is a relatively easy machine that tries to teach you a lesson, but perhaps you've already learned the lesson? Let's find out.
>
>Treat this box as if it were a real target and not a CTF.  
>
>Get past the login screen and you will find the flag. There are no rabbit holes, no hidden files, just a login page and a flag. Good luck!

---

All we get is a login page. So the logical conclusion would be that the vulnerability will be brute-force or SQLi.

The "lesson" here is apparently not to use **OR 1=1** in an SQLi payload.

> [!NOTE] Quote
> Using **OR 1=1** is risky and should rarely be used in real world engagements. Since it loads all rows of the table, it may not even bypass the login, if the login expects only 1 row to be returned. Loading all rows of a table can also cause performance issues on the database. However, the real danger of **OR 1=1** is when it ends up in either an UPDATE or DELETE statement, since it will cause the modification or deletion of every row.

Exploitation is fairly straight forward. Username enumeration is possible due to the error message and can be used to find a valid username. Then an SQLi payload can be used to bypass the login.

```
username=martin%27%3B+--+-&password=
OR
username=martin'; -- -&password=
```

Flag : THM{aab02c6b76bb752456a54c80c2d6fb1e}