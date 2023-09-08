---
description: Easy machine OSCP style
---

# Cap

## Reconnaissance

Ensuring the target machine is up. Due to the TTL we know it is a Linux machine.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 18.37.34.png" alt=""><figcaption></figcaption></figure>

Discovery of open ports on the target machine. Port 21, 22 and 80 are open.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 18.39.47.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 18.44.39.png" alt=""><figcaption></figcaption></figure>

Nothing interesting.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 18.48.02.png" alt=""><figcaption></figcaption></figure>

Now acces to the http server via the browser to inspect there. We notice that there is a supossedly loogged user called nathan as it can be seen in the right corner when accessing. Then, there are three sections that can be accessed so we begin with the first one.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 19.06.09.png" alt=""><figcaption></figcaption></figure>

The Security Snapshot is supposed to be doing a network snapshot that can be downloaded. If we look the URL, it is using numbers to identify the current snapshot so we begin trying random numbers to look for insteresting information. By downloading the current pcap for the id = 1 we cannot see anything interesting.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 19.08.48.png" alt=""><figcaption></figcaption></figure>

Then, by trying with different ids, we get that using id 0 there are different results for the number of packets, TCP, etc. so we donwload the capture again.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 19.11.06.png" alt=""><figcaption></figcaption></figure>

Now, inspecting the contents of the capture we can look for useful information and BOOM! As we can see in the image below, there is an FTP connection request and the user and password are in clear.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 19.14.39.png" alt=""><figcaption></figcaption></figure>

Here are the credentials. Now we can open an ftp connection.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 19.15.08.png" alt=""><figcaption></figcaption></figure>

Getting the user flag. Now we have to escalate privileges

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 19.16.50.png" alt=""><figcaption></figcaption></figure>

## Privilege escalation

As we know the credentials for the user nathan for ftp, we try to use the same for ssh so maybe they are the same. Once logged in, we look for the user privileges and nathan has no permission to run sudo.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 19.22.08.png" alt=""><figcaption></figcaption></figure>

Then we look for the files with SUID permissions as shown in the image below. However, we did not get any binary that could be use to escalate.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 19.26.17.png" alt=""><figcaption></figcaption></figure>

Now we look for the user capabilities to look for anyone that could give us a chance. As we can see in the image, the user has the capability to set uids using python, so now we can have a bash.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 19.28.25.png" alt=""><figcaption></figcaption></figure>

We use the capability to set uid to give us root privileges and then running a bash.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 19.30.37.png" alt=""><figcaption></figcaption></figure>

Getting the root flag.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 19.33.30.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 19.33.04.png" alt=""><figcaption></figcaption></figure>
