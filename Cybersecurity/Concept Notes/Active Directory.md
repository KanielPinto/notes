
A **Windows domain** is a group of users and computers under the administration of a given business. The main idea behind a domain is to centralise the administration of common components of a Windows computer network in a single repository called **Active Directory (AD)**. The server that runs the Active Directory services is known as a **Domain Controller (DC)**.

The main advantages of having a configured Windows domain are:

- **Centralized identity management:** All users across the network can be configured from Active Directory with minimum effort.
- **Managing security policies:** You can configure security policies directly from Active Directory and apply them to users and computers across the network as needed.

## **AD DS**
The core of any Windows Domain is the **Active Directory Domain Service (AD DS)**. This service acts as a catalogue that holds the information of all of the "objects" that exist on your network.
### Users
Users are one of the most common object types in Active Directory. Users are one of the objects known as **security principals**, meaning that they can be authenticated by the domain and can be assigned privileges over **resources** like files or printers. You could say that a security principal is an object that can act upon resources in the network. Users generally represent both people and services.

### Machines
Machines are another type of object within Active Directory; for every computer that joins the Active Directory domain, a machine object will be created. Machines are also considered "security principals" and are assigned an account just as any regular user. This account has somewhat limited rights within the domain itself.

The machine accounts themselves are local administrators on the assigned computer, they are generally not supposed to be accessed by anyone except the computer itself, but as with any other account, if you have the password, you can use it to log in.

**Note:** Machine Account passwords are automatically rotated out and are generally comprised of 120 random characters.

Identifying machine accounts is relatively easy. They follow a specific naming scheme. The machine account name is the computer's name followed by a dollar sign. For example, a machine named `DC01` will have a machine account called `DC01$`.

### Security Groups

You can define user groups to assign access rights to files or other resources to entire groups instead of single users. Security groups are also considered security principals and, therefore, can have privileges over resources on the network.

Groups can have both users and machines as members. If needed, groups can include other groups as well.

Several groups are created by default in a domain that can be used to grant specific privileges to users. As an example, here are some of the most important groups in a domain:

| **Security Group** | **Description**                                                                                                                                           |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Domain Admins      | Users of this group have administrative privileges over the entire domain. By default, they can administer any computer on the domain, including the DCs. |
| Server Operators   | Users in this group can administer Domain Controllers. They cannot change any administrative group memberships.                                           |
| Backup Operators   | Users in this group are allowed to access any file, ignoring their permissions. They are used to perform backups of data on computers.                    |
| Account Operators  | Users in this group can create or modify other accounts in the domain.                                                                                    |
| Domain Users       | Includes all existing user accounts in the domain.                                                                                                        |
| Domain Computers   | Includes all existing computers in the domain.                                                                                                            |
| Domain Controllers | Includes all existing DCs on the domain.                                                                                                                  |

### Organizational Units
Objects are organised in **Organizational Units (OUs)** which are container objects that allow you to classify users and machines. OUs are mainly used to define sets of users with similar policing requirements. Keep in mind that a user can only be a part of a single OU at a time. It is very typical to see the OUs mimic the business' structure, as it allows for efficiently deploying baseline policies that apply to entire departments.

To configure users, groups or machines in Active Directory, we need to log in to the Domain Controller and run "Active Directory Users and Computers" from the start menu.

Some containers are created by Windows automatically and contain the following:
- **Builtin:** Contains default groups available to any Windows host.
- **Computers:** Any machine joining the network will be put here by default. You can move them if needed.
- **Domain Controllers:** Default OU that contains the DCs in your network.
- **Users:** Default users and groups that apply to a domain-wide context.
- **Managed Service Accounts:** Holds accounts used by services in your Windows domain.


### Security Groups vs OUs

- **OUs** are handy for **applying policies** to users and computers, which include specific configurations that pertain to sets of users depending on their particular role in the enterprise. Remember, a user can only be a member of a single OU at a time, as it wouldn't make sense to try to apply two different sets of policies to a single user.
- **Security Groups**, on the other hand, are used to **grant permissions over resources**. For example, you will use groups if you want to allow some users to access a shared folder or network printer. A user can be a part of many groups, which is needed to grant access to multiple resources.

### Delegation

One of the nice things you can do in AD is to give specific users some control over some OUs. This process is known as **delegation** and allows you to grant users specific privileges to perform advanced tasks on OUs without needing a Domain Administrator to step in.

Following are the commands to change a user's password and subsequently require them to change their password on next sign-in :

```
> Set-ADAccountPassword user1 -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New Password') -Verbose

> Set-ADUser -ChangePasswordAtLogon $true -Identity user1 -Verbose
```

## **Managing Computers**

By default, all the machines that join a domain (except for the DCs) will be put in the container called "Computers"

In general, you'd expect to see devices divided into at least the three following categories:

**1. Workstations**
Workstations are one of the most common devices within an Active Directory domain. Each user in the domain will likely be logging into a workstation. This is the device they will use to do their work or normal browsing activities. These devices should never have a privileged user signed into them.  

**2. Servers**
Servers are the second most common device within an Active Directory domain. Servers are generally used to provide services to users or other servers.

**3. Domain Controllers**
Domain Controllers are the third most common device within an Active Directory domain. Domain Controllers allow you to manage the Active Directory Domain. These devices are often deemed the most sensitive devices within the network as they contain hashed passwords for all user accounts within the environment.

## **Group Policies**

Windows manages such policies through **Group Policy Objects (GPO)**. GPOs are simply a collection of settings that can be applied to OUs. GPOs can contain policies aimed at either users or computers, allowing you to set a baseline on specific machines and identities. Something important to have in mind is that any GPO will apply to the linked OU and any sub-OUs under it.

### GPO distribution

GPOs are distributed to the network via a network share called `SYSVOL`, which is stored in the DC. All users in a domain should typically have access to this share over the network to sync their GPOs periodically. The SYSVOL share points by default to the `C:\Windows\SYSVOL\sysvol\` directory on each of the DCs in our network.

Once a change has been made to any GPOs, it might take up to 2 hours for computers to catch up. If you want to force any particular computer to sync its GPOs immediately, you can always run the following command on the desired computer:

```shell
PS C:\> gpupdate /force
```

## **Authentication**

When using Windows domains, all credentials are stored in the Domain Controllers. Whenever a user tries to authenticate to a service using domain credentials, the service will need to ask the Domain Controller to verify if they are correct. Two protocols can be used for network authentication in windows domains:

- **Kerberos:** Used by any recent version of Windows. This is the default protocol in any recent domain.
- **NetNTLM:** Legacy authentication protocol kept for compatibility purposes.

While NetNTLM should be considered obsolete, most networks will have both protocols enabled.

### Kerberos Authentication

Kerberos authentication is the default authentication protocol for any recent version of Windows. Users who log into a service using Kerberos will be assigned tickets. Think of tickets as proof of a previous authentication. Users with tickets can present them to a service to demonstrate they have already authenticated into the network before and are therefore enabled to use it.

Kerberos authentication follows these steps:

1. **User Login:** The client sends an encrypted authentication request to the Key Distribution Center (KDC).
2. **TGT Issuance:** The KDC verifies credentials and issues a **Ticket Granting Ticket (TGT)**.
3. **Service Request:** The client presents the TGT to the KDC’s Ticket Granting Server (TGS) to request access to a service.
4. **Service Ticket Issuance:** The TGS verifies the TGT and provides a **Service Ticket**.
5. **Access Request:** The client presents the Service Ticket to the target server.
6. **Access Granted:** The server verifies the ticket and allows secure access.

### NetNTLM Authentication

NetNTLM works using a challenge-response mechanism.

1. The client sends an authentication request to the server they want to access.
2. The server generates a random number and sends it as a challenge to the client.
3. The client combines their NTLM password hash with the challenge (and other known data) to generate a response to the challenge and sends it back to the server for verification.
4. The server forwards the challenge and the response to the Domain Controller for verification.
5. The domain controller uses the challenge to recalculate the response and compares it to the original response sent by the client. If they both match, the client is authenticated; otherwise, access is denied. The authentication result is sent back to the server.
6. The server forwards the authentication result to the client.

Note that the user's password (or hash) is never transmitted through the network for security.

**Note:** The described process applies when using a domain account. If a local account is used, the server can verify the response to the challenge itself without requiring interaction with the domain controller since it has the password hash stored locally on its SAM.


## **Trees, Forests, Trusts**

### Tree
A **tree** is a hierarchical structure of one or more **domains** that share a **common namespace** and a **contiguous DNS name**.

Example: A company might have a main domain `company.com` and subdomains like `sales.company.com` and `hr.company.com`.

All domains in a tree automatically trust each other through **transitive trust** relationships, allowing seamless authentication and resource sharing.

### Forest
A **forest** is a collection of one or more **trees** that share:
- A **common schema** (defines object classes and attributes across AD).
- A **global catalog** (stores information about all objects for quick searches).
- A **trust relationship** between trees.

Unlike a tree, a **forest allows multiple domain trees with different namespaces**.

Example: A company with multiple subsidiaries might have `company.com` in one tree and `subsidiary.net` in another, both under the same forest.

**Forests serve as security boundaries**, meaning policies, users, and resources are typically isolated from other forests unless trust relationships are established.

A new security group needs to be introduced when talking about trees and forests. The **Enterprise Admins** group will grant a user administrative privileges over all of an enterprise's domains. Each domain would still have its Domain Admins with administrator privileges over their single domains and the Enterprise Admins who can control everything in the enterprise.
### Trust
**Trusts** define how authentication and resource access work between domains and forests. They can be:
- **Transitive:** Automatically extend trust within a forest (e.g., `A → B`, `B → C` means `A → C` is also trusted).
- **Non-Transitive:** Limited to only the specified domains (e.g., `A ↔ B`, but `A` does not trust `C`).
- **One-Way:** Only one domain trusts the other (e.g., `Domain A` trusts `Domain B`, but `B` does not trust `A`).
- **Two-Way:** Both domains trust each other equally.

Trusts allow users from one domain or forest to access resources in another while maintaining security and control.