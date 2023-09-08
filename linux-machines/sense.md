---
description: >-
  Easy machine OSCP style: PfSense, Remote Code Execution, Outdated Software,
  clear credentials
---

# Sense

## Reconnaissance

The first step as always is to ensure that the host is up by performing a ping request to the target source. As we can determine from the TTL value, it is a Linux machine as it is 64 (in fact it is 63 but due to the intermediary node between the target and the host the TTL value is decremented in one unit).

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-21 a las 10.40.49.png" alt=""><figcaption></figcaption></figure>

Through the first scan we find the open ports, which in this case are http and https ports (80 and 443).

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-21 a las 10.42.37.png" alt=""><figcaption></figcaption></figure>

With a version detection scan we look for further information about the services running on that ports to then look if there are some vulnerabilities availables for those versions.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-21 a las 10.43.35.png" alt=""><figcaption></figcaption></figure>

Accessing the website we only find a login panel for the PfSense service. After looking on the Internet for default credentials we find that the default credentials for pfsense are admin:pfsense but in this case it seems that have been changed.&#x20;

Through a directory search we try to list all the directories on the target host but using the dirbuster search we do not find anything useful.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-21 a las 11.02.12.png" alt=""><figcaption></figcaption></figure>

With `whatweb` we try to list the technologies of the webserver to look if there is something missing.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-21 a las 11.07.39.png" alt=""><figcaption></figcaption></figure>

As we have not find anything, we laungh a nmap scan with a script for http enumeration and it finds a hidden text file called changelog.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-21 a las 11.12.34.png" alt=""><figcaption></figcaption></figure>

The content of the hidden file is shown in the following image. As we can see, there is a sentence which says that there are 2 out of 3 vulnerabilities that have been patched, which means that there is a vulerability that remains active.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-21 a las 11.14.25.png" alt=""><figcaption></figcaption></figure>

At this point I suppose that if there is a hidden text file with that information, it must be some other information hidden in text files. With wfuzz we launch a fuzz scan to detect all the text files on the webserver and WE GOT IT! there is another text file inside called system-users.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-21 a las 17.38.38.png" alt=""><figcaption></figcaption></figure>

Looking at the content of the file we find some credentials. As we supposed, there is a user called rohit whose password seems to be the default one, which is pfsense.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-21 a las 17.37.28.png" alt=""><figcaption></figcaption></figure>

## Exploitation

Introducing the credentials obtained we got in. Once inside, we find that the pfsense version is the 2.1.3, so now we can look for specific exploits for that version.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-21 a las 17.38.20.png" alt=""><figcaption></figcaption></figure>

Using searchsploit we find a vulnerability that enables an attacker to inject commands.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-21 a las 18.30.26.png" alt=""><figcaption></figcaption></figure>

With that exploit is relatively easy to get access to the target machine and as it can be seen in the following image, we are in as root.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-21 a las 17.46.37.png" alt=""><figcaption></figcaption></figure>

Getting the user flag and root flag very easy.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-21 a las 17.48.12.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-21 a las 17.48.21.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-21 a las 17.48.36.png" alt=""><figcaption></figcaption></figure>

## Understanding the vulnerability and the exploit

Apparently, there is a vulnerability on the generation of the RDD status graph, as the exec() function allows to inject code simply by using semicolons just after the request URL.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-21 a las 18.03.52.png" alt=""><figcaption></figcaption></figure>

This is the URL that can be used to exploit the vulnerability.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-21 a las 18.04.15.png" alt=""><figcaption></figcaption></figure>

To test it manually, we open Burpsuite and get a request to then send a crafted one. As it can be seen in the image, we just try to execute the command whoami and send the response to our machine.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-21 a las 18.12.48.png" alt=""><figcaption></figcaption></figure>

Being previously listening on the port specified in the URL, we receive the response of the request.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-21 a las 18.12.38.png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
To get a shell it is not that easy as there is some kind of sanitization, but it is relatively easy to evade this sanitization by trial and error.
{% endhint %}
