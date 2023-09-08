---
description: 'Easy machine OSCP style: SAMBA, Outdated software, Metasploit, RCE'
---

# Legacy

## Reconnaissance

The first step is to ensure that the host is up by pinging it. Because of the TTL we know that it is a Windows machine as the value is 128 (it should be 127 but the intermediary node decrements it in one unit).

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-16 a las 19.29.58.png" alt=""><figcaption></figcaption></figure>

With the first scanning for all open ports we can see that there are only three open ports which are 135, 139 and 445. As we know, port 445 corresponds to SMB service.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-16 a las 19.30.30.png" alt=""><figcaption></figcaption></figure>

With a version scan we can determine the version of the services running on that open ports.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-16 a las 19.31.57.png" alt=""><figcaption></figcaption></figure>

With the tool smbclient we try to list the shared resources on the domain but we do not have access.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-16 a las 19.41.39.png" alt=""><figcaption></figcaption></figure>

We try it again with smbmap but again we cannot list them.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-16 a las 19.43.56.png" alt=""><figcaption></figcaption></figure>

With a nmap scanning we are going to run some scripts to find vulnerabilities on the service SMB. As we can see in the image below, there is a Remote Code Execution vulnerability which is called `ms17-010` and as we know, this refers to the EternalBlue vulnerability.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-16 a las 19.48.37.png" alt=""><figcaption></figcaption></figure>

## Exploitation

Using searchsploit we find some exploits some of them suitable for metasploit and some others in raw. We are not going to use Metasploit as part of the preparation for the OSCP. However, the version of Windows is XP and none of the exploits from this list is suitable for this version.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-16 a las 19.49.35.png" alt=""><figcaption></figcaption></figure>

Looking on the Internet for exploits we found a GitHub repo that contains different exploits for different versions. Moreover, there is a checker that checks if the target machine is vulnerable to this EternalBlue. In this case, there is a pipe which enables the exploit which is the browser.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-16 a las 20.38.16.png" alt=""><figcaption></figcaption></figure>

To perform the exploitation, we launch the exploit script specifying the machine and the pipe and the script gives us a shell.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-16 a las 20.44.59.png" alt=""><figcaption></figcaption></figure>

Within this shell, we cannot run some commands but as we can see, here it is the IP address of the target machine. At this point is trivial to get both flags.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-16 a las 20.45.30.png" alt=""><figcaption></figcaption></figure>

PWNED!

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-16 a las 22.10.41.png" alt=""><figcaption></figcaption></figure>
