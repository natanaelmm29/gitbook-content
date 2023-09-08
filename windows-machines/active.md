---
description: >-
  Easy machine OSCP style: Kerberos, Active Direcory, SMB, Impact, Password
  Cracking, Kerberoasting
---

# Active

## Reconnaissance

First we perform a ping request to ensure that the host is up and by the TTL value we can determine that it is a Windows machine (TTL = 128). In reality, here the TTL is 127 due to the intermediary node between the host and the target machine.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 18.02.11.png" alt=""><figcaption></figcaption></figure>

Then we make the first scan to get all open ports on the target machine. As we can see, there are so many ports opened. There are some interesting ports related to Kerberos, SMB, etc.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 18.04.35.png" alt=""><figcaption></figcaption></figure>

After the first scan, we perform a version scan to get further information about the services that are running on those ports on the target machine.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 18.28.38.png" alt=""><figcaption></figcaption></figure>

From the image below we can view that the name is DC which means Domain Controller and the Domain name is active.htb.

{% hint style="info" %}
For some attacks such as kerberoasting where the domain name is useful it is important to determine to which IP address corresponds the domain name. In this case, modifying the file `/etc/hosts` to add the specific line.
{% endhint %}

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 18.23.24.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 18.28.18.png" alt=""><figcaption></figcaption></figure>

Now, getting back to SMB on port 445, we list the shared resources to determine if we have some kind of access to any of them.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 18.30.50.png" alt=""><figcaption></figcaption></figure>

By taking a look at the content of the resource Replication we can see that it has a directory inside and looking into that direcory we can see some others.&#x20;

{% hint style="info" %}
These direcory names are similar to those included in SYSVOL which is a set of folders and files which are stored in the DC hard drive. In this case, DfsrPrivate is a directory used as a staging folder to act as a cache for new and changed files to be replicated from sending members to receiving members.
{% endhint %}

## Exploitation

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 18.33.26.png" alt=""><figcaption></figcaption></figure>

Apparently, the resource Replication could be a copy of the SYSVOL resource. SYSVOL resource has some Group Policies under the Policies directory and sometimes there can be found plaintext passwords. In this case, following the path under the policies direcory, it is found a file called Groups.xml.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 18.43.40.png" alt=""><figcaption></figcaption></figure>

Then we download the file in order to inspect the content.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 18.48.02.png" alt=""><figcaption></figcaption></figure>

As we can see in the image below, there is a cpassword for the user SVC\_TGS.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 18.48.55.png" alt=""><figcaption></figcaption></figure>

There is a tool called `gpp-decrypt` which basically decrypts these hashes to get the passwords.&#x20;

{% hint style="info" %}
The password is encrypted using AES-256. However, at some time Microsoft decided to publish the AES key so nowadays these hashes are so easy to attack.
{% endhint %}

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 18.50.51.png" alt=""><figcaption></figcaption></figure>

Now with crackmapexec we ensure that the password is valid for the specific user.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 18.54.42.png" alt=""><figcaption></figcaption></figure>

At this point we are able to list the shared resources to determine which are the permissions of the user that we have the password. Then, with smbmap we are going to list the content of different directories to look for insteresting information.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 18.59.06.png" alt=""><figcaption></figcaption></figure>

With smbmap and inspecting the different directories we are able to get the user flag.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 19.00.06.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 19.02.08.png" alt=""><figcaption></figcaption></figure>

Then, using the utility rpcclient and the grabbed credentials we can list the users in the domain as well as the groups, administrators, description of the users and groups, etc.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 19.09.03.png" alt=""><figcaption></figcaption></figure>

## Privilege escalation

Here we are displaying which users belong to the group called Domain Admins which is the most privileged group.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 19.12.56.png" alt=""><figcaption></figcaption></figure>

At this point two attacks could be performed

{% hint style="info" %}
It is important to have the datetime synchronized with the domain controller in order to perform some of the possible attacks.
{% endhint %}

### Possible attacks

#### ASREPROASTING attack (no password needed)

It is a similar technique as the Kerberoasting which tries to crack the passwords offline for those users that have the attribute DONT\_REQ\_PREAUTH. This attribute means that the user do not have to be authenticated before the kerberos authentication.

This is not recommended as an attacker could send a malicious authentication request without knowing the password of the user and as the KDC sends back an encrypted TGT, the attacker could crack the administrator password offline.&#x20;

In this case, it is not possible as the user has the previously commented attribute disabled.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 19.24.37.png" alt=""><figcaption></figcaption></figure>

#### KERBRUTE (no password needed)

Basically is a brute force attack which sends authentication requests to the Authentication Server to enumerate valid accounts in the Active Direcory.

In this case, we only show how the attack could be performed. The command would be the following:

```
kerbrute userenum --dc 10.10.10.100 -d active.htb /user/share/SecLists/Usernames/Names/names.txt
```

#### Kerberoasting

This attack basically consists of:

1. With `impacket-GetUserSPNs` we get the SPN of the target service that is Kerberos authenticated. In this case, teh SPN is active/CIFS:445

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 19.31.36.png" alt=""><figcaption></figcaption></figure>

2. Then, with `impacket-GetUserSPNs` and using the option `-request` we perform a service ticket request to the target service. This ticket has a password hash that can be cracked offline and in this case is the administrator's password.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 19.33.18.png" alt=""><figcaption></figcaption></figure>

To crack the password, we copy the hash into a file and use the tool john with the rockyou dictionary to crack it.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 19.34.26.png" alt=""><figcaption></figcaption></figure>

With the password, we can check that the credentials are valid with crackmapexec.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 19.35.43.png" alt=""><figcaption></figcaption></figure>

With the utility `impacket-psexec` we can get a shell as the NT Authority System account and grab the flag.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 19.42.17.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 19.41.48.png" alt=""><figcaption></figcaption></figure>
