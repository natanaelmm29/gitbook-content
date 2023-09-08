---
description: Medium machine OSCP style
---

# SolidState

## Recoinnassance&#x20;

Ensuring the host is up and viewing that due to the TTL = 64 it is a Linux machine.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-18 a las 23.25.09.png" alt=""><figcaption></figcaption></figure>

First scan for open ports.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-18 a las 23.27.13.png" alt=""><figcaption></figcaption></figure>

Version scan over the open ports. From this scan we can deduce that the machine has an emal server running and we notice that when scanning the port 4555 it is trying to log in. Lookin for information in Google, we get to default credentials of **James Remote Administration Tool** and we try to log in and we got access.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-18 a las 23.32.24.png" alt=""><figcaption></figcaption></figure>

## Exploiting

Once logged in, we start looking at the possible commands that can eb executed and we run `listusers` to get all the existing accounts. With all the email account names, we notice that there is a command called `setpassword` that can be used to change the password of the email accounts, so we change all the passwords for all the user to then access their email inbox.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-18 a las 23.56.01.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 0.00.12.png" alt=""><figcaption></figcaption></figure>

To view all the users emails we can use telnet to access the port 110 belonging to POP3 and then logging as the user and password that we changed before. As we can see, the user james does not have any email pending.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 0.01.18.png" alt=""><figcaption></figcaption></figure>

At this point we log as the mailadmin as I thought it could have something interesting as it was the admin account, but the account did not have any email.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 0.03.17.png" alt=""><figcaption></figcaption></figure>

Then we proceed logging as the user john and we notice that there is a message pending so we retrieve the content and after reading it we deduce that it is something happening with mindy.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 0.04.25.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 0.05.59.png" alt=""><figcaption></figcaption></figure>

After logging with mindy's account, we notice that whithin one of the two messages available there are ssh credentials so maybe we can connect to mindy's account using ssh.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 0.06.17.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 0.07.07.png" alt=""><figcaption></figcaption></figure>

Connecting mindy's account via ssh. We are in.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 0.07.55.png" alt=""><figcaption></figcaption></figure>

At this point there are only three commands that can be executed because mindy has a rbash (restricted bash) but anyway we could get the user flag as cat was one of the available commands.&#x20;

## Privilege escalation

Now we look for any permission of the user that could enable us to escalate privileges but in a simple view there is nothing really interesting.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 0.31.13.png" alt=""><figcaption></figcaption></figure>

At this point, we could scan the process that are running to know if a command is being executed by root at any time so we proceed to create a procmon bash script that monitors the command that are being executed. Instead of this bash script PSPY can be used at this point.&#x20;

{% hint style="info" %}
`/dev/shm` is a directory "similar" to `/tmp` (shm --> Shared Memory. In this directory the user can write, and here is being used to write the procmon.sh&#x20;
{% endhint %}

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 0.41.01.png" alt=""><figcaption></figcaption></figure>

The content of the script is the following. The script is running the `ps -eo command` to know what commands are being executed and using the while, it keeps monitoring. Using the command `diff` we evaluate the difference between the variables containing the result of the execution of the command to know the differences in the time window between the declaration and the while call. Moreover, we are using grep to filter some words to not to be showed.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 0.57.27.png" alt=""><figcaption></figcaption></figure>

When running the script, we notice that there is a file called `/opt/tmp.py` which is being executed as root at some point. Looking at the file, we have access to it and even we can modify it.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 1.03.57.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 1.04.13.png" alt=""><figcaption></figcaption></figure>

So we modify the python script so instead of removing all below the `/tmp` directory the script is going to give **suid** privileges to the `/bin/bash` so any user can run a bash as root temporarily.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 1.10.49.png" alt=""><figcaption></figcaption></figure>

Waiting for the script to be executed.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 1.21.11.png" alt=""><figcaption></figcaption></figure>

With a bash as root, we can get the root flag and PWNED!

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 1.23.23.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 1.22.55.png" alt=""><figcaption></figcaption></figure>
