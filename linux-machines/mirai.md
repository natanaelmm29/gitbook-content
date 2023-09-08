---
description: Easy machine OSCP style
---

# Mirai

## Reconnaissance

Machine is up and it is a Linux due to the TTL equal to 63. Should be 64 but the intermediary node between the host and the target decrements the TTL in one unit.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 12.30.25.png" alt=""><figcaption></figcaption></figure>

We scan for open ports. Ports: 22, 53, 80, 1530, 32400 and 32469 are open.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 12.31.23.png" alt=""><figcaption></figcaption></figure>

We perform a version detection scan over the open ports to get more information about the services running on that ports. After looking for exploits using searchsploit over the different versions we do not get anything interesting.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 12.32.45.png" alt=""><figcaption></figcaption></figure>

We now perform a request to the http server to see the headers and we notice that there is a rare one clalled Pi-Hole.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 12.43.25.png" alt=""><figcaption></figcaption></figure>

Searching on the Internet we get that the Pi Hole header is used commonly by raspberries.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 12.44.11.png" alt=""><figcaption></figcaption></figure>

**THIS I DID NOT KNOW!!! Raspberry has a default user and password and searching on the Internet to get them we got to a page where there were.**

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 12.44.49.png" alt=""><figcaption></figcaption></figure>

Now we try to log via ssh and yes! it was successful.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 12.45.33.png" alt=""><figcaption></figcaption></figure>

## Privilege escalation

Once as pi user, we are able to get the user flag. Moreover, we look for sudo privileges and the user pi has them so we switch to root user.  We look for the root flag but apparently it was removed so we have to look into the backup on the usb stick clue that was given.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 12.49.07.png" alt=""><figcaption></figcaption></figure>

With the command df we look for the path of the usb stick and we get it.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 12.52.34.png" alt=""><figcaption></figcaption></figure>

To get the root flag we use the strings command to get the printable characters, as if we use cat we also have got it but it would print all the content. PWNED!

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 12.53.05.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-23 a las 12.52.06.png" alt=""><figcaption></figcaption></figure>
