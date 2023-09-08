---
description: Easy machine OSCP style
---

# Traverxec

## Reconnaissance

Host is up and it is a Linux machine.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 14.40.59.png" alt=""><figcaption></figcaption></figure>

First scan, ports 22 and 80 are open.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 14.40.30.png" alt=""><figcaption></figcaption></figure>

Service scan

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 14.42.24.png" alt=""><figcaption></figcaption></figure>

With `whatweb` tool we can determine which technologies and framworks are the server using and we notice that the server is a nostromo webserver v1.9.6. With `searchsploit` we look for exploits for that version of the server and we finda RCE for that specific version.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 14.49.29.png" alt=""><figcaption></figcaption></figure>

Scanning for interesting paths on the server using the `http-enum` script.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 15.01.58.png" alt=""><figcaption></figcaption></figure>

## Exploit

The exploit is a python script that looks like the following images. The script returns a shell on the ip and port that are specified when the command is executed.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 15.16.16.png" alt=""><figcaption></figcaption></figure>

Having a look at the payload and then on the NVD from the NIST to understand the vulnerability that is being exploited, the vulnerability is a Directory traversal in the function `http_verify` in nostromo that allows  an attacker to achieve remote code execution via a crafter HTTP request.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 15.15.47.png" alt=""><figcaption></figcaption></figure>

We start listening on the port to which the shell is going to be sent. After achieving the shell, we get to an interactive shell.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 15.17.35.png" alt=""><figcaption></figcaption></figure>

## Privilege escalation

Then we are logged as **www-data** but we do not have any sudo privileges neither suid privileges on something that could enable us to escalate.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 15.25.32.png" alt=""><figcaption></figcaption></figure>

We start looking for interesting files in some directories and we notice that inside the server configuration directory there is a suspicious file which contains a hash and a user.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 15.32.50.png" alt=""><figcaption></figcaption></figure>

Then, in the configuration file we notice that the public directory is called public\_www for the users, so this could be a clue.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 16.11.37.png" alt=""><figcaption></figcaption></figure>

If we try to list the directories inside the user david directory, we have no permission, but knowing that there might be a directory called public\_www we try to access it and got in. Inside the directory, there is another directory called protected-file-area so we access it.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 16.12.46.png" alt=""><figcaption></figcaption></figure>

Inside that directory there is a tar file which is supposed to have the ssh identity of some user so we send it to our host to look inside it.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 17.02.06.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 17.01.47.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 17.01.21.png" alt=""><figcaption></figcaption></figure>

After decompressing the file we look inside and there is an id\_rsa file.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 17.04.09.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 17.04.58.png" alt=""><figcaption></figcaption></figure>

Having a look at the private key we notice that it is encrypted so now the brute-force beggins.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 17.05.27.png" alt=""><figcaption></figcaption></figure>

With `ssh2john` and `john` we crack the password for the private key.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 17.07.17.png" alt=""><figcaption></figcaption></figure>

Then with `openssl` we decrypt the private key using the password obtained.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 17.10.24.png" alt=""><figcaption></figcaption></figure>

Now we can access via ssh to the user david and his private key.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 17.21.50.png" alt=""><figcaption></figcaption></figure>

Getting the user flag.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 17.23.48.png" alt=""><figcaption></figcaption></figure>

Looking for the privileges.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 17.30.40.png" alt=""><figcaption></figcaption></figure>

Looking for permissions to escalate to root. Nothing interesting.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 17.31.47.png" alt=""><figcaption></figcaption></figure>

After browsing through directories, we reached a file that is running a command as root, which is `journalctl.`

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 17.36.40.png" alt=""><figcaption></figcaption></figure>

Proving that the user david could run this command as script which includes sudo commands.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 17.37.23.png" alt=""><figcaption></figcaption></figure>

At this point we start looking how to exploit this. We only can execute this command with -n5 lines.&#x20;

{% hint style="info" %}
I looked for this on the Internet, I had no idea.

The command `journalctl` normally outputs loads of logs so `less` is executed. Using `less` the user without privileges can run  **!`/bin/sh`**`and get a shell as root.`
{% endhint %}

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 17.39.34.png" alt=""><figcaption></figcaption></figure>

As we were only allowed to show 5 logs, we made the terminal as tiny as possible to force the `less` execution.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 17.46.36.png" alt=""><figcaption></figcaption></figure>

Then we got the shell as root.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 17.46.01.png" alt=""><figcaption></figcaption></figure>

Getting the root flag and PWNED!

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 17.50.21.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-19 a las 17.49.58.png" alt=""><figcaption></figcaption></figure>

