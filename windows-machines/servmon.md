---
description: >-
  Easy machine OSCP style: Apache, SSH, FTP, Tunneling, Password Spraying, Port
  Forwarding, LFI, Clear text credentials, Anonymous access
---

# Servmon

## Reconnaissance

The first step is to ensure that the host is up. By the TTL we can determine that it is a Windows machine as its value is 128 (in this case 127 due to the intermediary node).

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 17.49.21.png" alt=""><figcaption></figcaption></figure>

The first scan is a fast scan to get the open ports on the target machine. As we can see in the image below, there are several ports opened but we are going to focus on those which are more interesting. Apparently, there is an http server on port 80 and an https server on port 8443, the SMB server on port 445, the SSH service on port 22 and FTP service on port 21.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 17.50.34.png" alt=""><figcaption></figcaption></figure>

With the open ports we perform a version scan to obtain more information about the services running on that ports.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 17.55.28.png" alt=""><figcaption></figcaption></figure>

The first step is to try to access via anonymous user within the FTP service and we get in.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 17.58.41.png" alt=""><figcaption></figcaption></figure>

We can determine that there are two users.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 17.59.28.png" alt=""><figcaption></figcaption></figure>

In the Nadine user directory we find a file with an interesting text. Apparently, it should exist a file called Passwords.txt on the path `C:\Users\Nathan\Desktop\Passwords.txt`

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 18.00.51.png" alt=""><figcaption></figcaption></figure>

Looking into the Nathan user directory, we do not find anything interesting yet.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 18.01.46.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 18.02.21.png" alt=""><figcaption></figcaption></figure>

After analysing the FTP service, we try to list the shared resources with smbclient and smbmap but we are not allowed.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 18.31.06.png" alt=""><figcaption></figcaption></figure>

Accessing the server on port 80, we notice that it is something called NVMS-1000. After looking for exploits to this service we find that it is vulnerable to Directory Traversal.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 19.12.56.png" alt=""><figcaption></figcaption></figure>

Indeed, what is shown in the exploit PoC is that the service is vulnerable to LFI, as we could get the data of a file in the target machine via the URL.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 19.12.47.png" alt=""><figcaption></figcaption></figure>

With Burpsuite we ensure that the response of the PoC request is the one that it should be so the service is vulnerable.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 19.15.26.png" alt=""><figcaption></figcaption></figure>

At this point we could try to get the passwords file within the user Nathan directory and as it can be seen in the image, the response is a list of passwords.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 19.17.26.png" alt=""><figcaption></figcaption></figure>

With crackmapexec we check if any password corresponds to any of the users using the two lists shown in the image and one works for the user Nadine.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 19.19.43.png" alt=""><figcaption></figcaption></figure>

Then we can log in as Nadine via SSH

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 19.23.01.png" alt=""><figcaption></figcaption></figure>

Getting the user flag.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 19.24.10.png" alt=""><figcaption></figcaption></figure>

## Privilege escalation

To escalate privileges we list the privileges of the user Nadine but there is nothing interesting here.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 19.26.21.png" alt=""><figcaption></figcaption></figure>

To list more information about the user, we use the command `net user` to list the groups, etc. but again there is nothing interesting.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 19.26.51.png" alt=""><figcaption></figcaption></figure>

As we stated in the reconnaissance process, there is an https server running on port 8443. Accessing it we find that a server called NSClient++ is running there.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 19.30.30.png" alt=""><figcaption></figcaption></figure>

Looking for exploits to this service we find that there is a vulnerability which enables privilege escalation via this service.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 19.30.51.png" alt=""><figcaption></figcaption></figure>

As we can see in the exploit explanation, there are some steps that must be performed to gain access as the administrator.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 19.50.59.png" alt=""><figcaption></figcaption></figure>

The first step is to get the password of the admin of the service by the command shown in the exploit.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 19.51.40.png" alt=""><figcaption></figcaption></figure>

When trying to access with that credentials, we find an error that states that we are not allowed but it is because it is restricting the access from remote locations.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 19.53.42.png" alt=""><figcaption></figcaption></figure>

To fix it, we perform a Port Forwarding executing the following command:

```
ssh Nadine@10.10.10.184 -L 8443:127.0.0.1:8443
```

As we can see in the image below, now we can access as the admin from the login panel.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 19.56.40.png" alt=""><figcaption></figcaption></figure>

The second step was to ensure that some modules were enabled (we did not have to change anything as all required modules were enabled already).

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 19.58.25.png" alt=""><figcaption></figcaption></figure>



<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 19.57.21.png" alt=""><figcaption><p>Dashboard when log in as the admin</p></figcaption></figure>

The third step was to create a file with the reverse shell called `evil.bat` and to download netcat.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 20.02.43.png" alt=""><figcaption></figcaption></figure>

After that, with the utility of impacket called smbserver we create a shared resource to then access it from the target machine.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 20.08.16.png" alt=""><figcaption></figcaption></figure>

However, the policies block unauthenticated guest access.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 20.10.09 (1).png" alt=""><figcaption></figcaption></figure>

However, this utility enable the creation of a user and password and the command that was executed was:

```bash
impacket-smbserver smbFolder $(pwd) -smb2support -username Natanael -password Natanael123
```

With the command net use we connect to the shared resource.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 20.12.16.png" alt=""><figcaption></figcaption></figure>

We bring the necessary files to the temp directory.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 20.15.34.png" alt=""><figcaption></figcaption></figure>

Ensuring that the files were successfully copied.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 20.19.59.png" alt=""><figcaption></figcaption></figure>

As the PoC states, the next step was to go to Settings > External Scripts > Scripts and create a new script with any key (reverse in this case, but it does not mind) and the value as c:\temp\evil.bat.

Once created the script, it was necessary to save the configuration and reload the service. Ath that point, a shell was obtained when clicked the Queries tab.

Setting up the listener and receiving the shell.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 22.06.44.png" alt=""><figcaption></figcaption></figure>

Getting the root flag and PWNED!

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 22.07.19.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-14 a las 22.07.35.png" alt=""><figcaption></figcaption></figure>
