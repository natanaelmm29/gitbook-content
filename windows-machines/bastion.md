---
description: >-
  Easy machine OSCP style: SMB, Authentication, Password Cracking, Password
  Dump, Anonymous/Guest access
---

# Bastion

## Reconnaissance

The first step is to ensure that the host is up with a ping request. Moreover, because of the TTL value we can determine that it is a Windows machine (should be 128 but due to the intermediary node it is decremented in one unit).

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 17.37.59.png" alt=""><figcaption></figcaption></figure>

With a first scan we can find the open ports on the target machine.  As we can see from the output, SSH, RPC, SMB, WINRM services are up.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 17.39.22.png" alt=""><figcaption></figcaption></figure>

With a version scan we get further information about the services running on that ports.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 17.42.47.png" alt=""><figcaption></figcaption></figure>

With `smbclient` we can list the shared resources in the target macine. With `smbmap` we should be able to list them as well and also the permissions should be displayed, but an authentication error appears.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 17.44.51.png" alt=""><figcaption></figcaption></figure>

However, using a null user we can list them as we can see, we have read and write permissions on the Backup resource. As we can see, in the Backup resource there is a file called note.txt.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 18.06.56.png" alt=""><figcaption></figcaption></figure>

## Exploitation

This is the content of the file note.txt:

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 18.09.50.png" alt=""><figcaption></figcaption></figure>

Here is another way of listing the content of the Backup shared resource.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 18.08.32.png" alt=""><figcaption></figcaption></figure>

To work in a more comfortable way with the shared resource, we can mount it and then work it from the `/mnt` directory.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 18.24.05.png" alt=""><figcaption></figcaption></figure>

This is the content of the whole Backup resource. Here there are some interesting files with the extension .vhd.

{% hint style="info" %}
`.vhd` files are Virtual Hard Disk that are used to store the entire contents of a computer's hard drive.
{% endhint %}

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 18.24.51.png" alt=""><figcaption></figcaption></figure>

In this image we can see that there is a .vhd file whose size is 5GB... Interesting!!

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 18.28.04.png" alt=""><figcaption></figcaption></figure>

To mount the Virtual Hard Disk we used the utility `guestmount`

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 18.39.19.png" alt=""><figcaption></figcaption></figure>

As the backup file contains the whole system, we can use `impacket-secretsdump` to dump some users's hashes and we got them.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 18.47.00.png" alt=""><figcaption></figcaption></figure>

With crackmapexec we try to access to the some user's account but we are not able to get a shell, for example using `psexec`.

{% hint style="info" %}
If the output of the command says PWNED it means that we could use the hash to get a shell as the user.
{% endhint %}

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 18.52.40.png" alt=""><figcaption></figcaption></figure>

The alternative is to crack the passwords offline and we get the password of the L4mpje user.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 18.55.04.png" alt=""><figcaption></figcaption></figure>

Connecting via SSH, we log as L4mpje and get the flag.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 18.56.42.png" alt=""><figcaption></figcaption></figure>

## Privilege escalation

Now we need to escalate to get administrator privileges but we do not have access to the Admin directory.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 18.58.30.png" alt=""><figcaption></figcaption></figure>

Looking for the user privileges, we do not find any that could be used to escalate.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 18.58.59.png" alt=""><figcaption></figcaption></figure>

Listing the groups we do not see anything interesting.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 18.59.50.png" alt=""><figcaption></figcaption></figure>

We start looking for information within the different directories. The backup directory is the one previosly examined, within the Users directory is any interesting...

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 19.02.35.png" alt=""><figcaption></figcaption></figure>

Listing the Program Files directory, we do not find anything.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 19.02.40.png" alt=""><figcaption></figcaption></figure>

However, inside the Program Files x86 directory there is something interesting called `mRemoteNG`. Looking for information about exploits for this service, we found that there is a file inside every user directory that contains the credentials of the user for this service.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 19.02.57.png" alt=""><figcaption></figcaption></figure>

This is the content of the file. As we can see, it contains the credentials of the user 'Administrator' as well as the rest of the users. But we are interested in the first one. However, the password is not in plaintext.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 19.06.31.png" alt=""><figcaption></figcaption></figure>

Although the password is not in clear text, there is python exploit that decrypts the password.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 19.09.18.png" alt=""><figcaption></figcaption></figure>

At this point we can connect via ssh to the Admin account, get the root flag and PWNED!

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 19.10.31.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 19.10.49.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-17 a las 19.11.13.png" alt=""><figcaption></figcaption></figure>
