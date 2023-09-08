---
description: Easy machine OSCP style
---

# OpenAdmin

## Recoinnassance

First we are going to ensure that the machine is up by pinging it. Moreover, from the TTL we can determine that it is a Linux machine as it is 63. In fact, it should be 64 but there is an intermediary node between the machine and the host from the HTB VPN so that is because it is 63.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-20 a las 19.05.18.png" alt=""><figcaption><p>machine pinging</p></figcaption></figure>

Now we can continue to scan all the target machine ports which are open to know what ports and services are running. From the scan below, we can determine that there are 2 open ports (22 and 80) so now we can start a version detection scan to learn more information about the services that are running in those ports on the target machine.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-20 a las 19.09.55.png" alt=""><figcaption></figcaption></figure>

From the version detection scan we get the version of the services running. There is an apache server httpd 2.4.29 and a ssh server OpenSSH 7.6p1.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-20 a las 19.15.43.png" alt=""><figcaption></figcaption></figure>

Firstly we are going to concentrate on the HTTP server which is running on port 80. With the **whatweb** utility we are going to get more information of the server running, for example, what content management system is implementes, Joompla, Drupal, etc.

In this case, no additional information was given apart from the yet known.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-20 a las 19.18.49.png" alt=""><figcaption></figcaption></figure>

We can also use the Wappalyzer extension to know what kind of server is running on the target machine.&#x20;

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-20 a las 19.22.24.png" alt=""><figcaption></figcaption></figure>

As it is not any interesting information, we now want to retrieve the existing paths using a fuzzer.

In this case, we are going to apply a fuzzer using -c to colorize the response, -t to use an amount of threads, --hc to hide 404 responses as they are not interesting, a wordlist of common used paths and the URL of the server, and the response is going to be stored in the FUZZ directory of the server.&#x20;

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-20 a las 19.28.39.png" alt=""><figcaption></figcaption></figure>

When finished, we can now explore the different paths returned by the execution. By examining the content of the music path, we get access to a web page with multiple links, including a login path.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-20 a las 19.35.44.png" alt=""><figcaption></figcaption></figure>

By clicking on the login link, we are redirected to a, what it looks like, an administrative panel of **OpenNetAdmin**.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-20 a las 19.37.43.png" alt=""><figcaption></figcaption></figure>

Now, we are going to search for vulnerabilities associated with the OpenNetAdmin service with searchexploit.

From the results, we notice that there is a remote code execution vulnerability for the version 18.1.1 and YES!, our OpenNetAdmin panel is running over v18.1.1.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-20 a las 19.42.11.png" alt=""><figcaption></figcaption></figure>

Taking a look at the exploit code, we notice that it is a one line command execution which makes a request with some parameters to the URL. As we can see, there is a parameter called **cmd** and we can suppose that it is used to run some commands on the target machine, so we are not going to use the exploit provided but the command in order to better understand what it does.&#x20;

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-20 a las 19.49.53.png" alt=""><figcaption></figcaption></figure>

In this case, we are going to set up an HTTP server on port 80 on the host machine to then make requests from the command to prove the performing of the exploit.&#x20;

As we can see in the image below, we can make a request to our host from the request to the target machine, so the command was executed.&#x20;

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-20 a las 22.34.50.png" alt=""><figcaption></figcaption></figure>

Now, from this command, we are going to create a file so that when performing the request it loads the content of the file as we can see it in the second image.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-20 a las 22.44.57.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-20 a las 22.58.13.png" alt=""><figcaption></figcaption></figure>

Now, by modifying the request to the server so that the content is interpreted in bash, when performing the request it executes the code and returns a shell on the HTTPS server which was setup on the port 443.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-20 a las 22.59.29.png" alt=""><figcaption></figcaption></figure>

Now we are into the system.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-20 a las 23.02.58.png" alt=""><figcaption></figcaption></figure>

To obtain a pseudo-console we just execute the following:

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-20 a las 23.05.45.png" alt=""><figcaption></figcaption></figure>

Then to get an interactive shell we have to modify some value of the terminal and shell values.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-20 a las 23.07.25.png" alt=""><figcaption></figcaption></figure>

As we can see, when trying to retrieve the flag, the user www-data has no permissions to access the another users directory.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-20 a las 23.46.34.png" alt=""><figcaption></figcaption></figure>

Now we are going to look for interesting data into the **www** directory, especially for those configuration files. As we can see in the image below, there is an interesting file called database settings that could have sensitive data.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-20 a las 23.57.44.png" alt=""><figcaption></figcaption></figure>

Finally, we know the credentials for the ona\_sys user.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-21 a las 0.03.25.png" alt=""><figcaption></figcaption></figure>

With that credentials, we now can try to change to another user using that password, for example jimmy or joanna.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-21 a las 0.14.37.png" alt=""><figcaption></figcaption></figure>

Now, by looking into the paths we notice that the user jimmy belongs to the group internal, so we can access the directory to look for sensitive information. &#x20;

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-21 a las 0.18.40.png" alt=""><figcaption></figcaption></figure>

In the directory, if we inspect the file called main.php we can see that there is a function called shell\_exec which gets the content of the file containing the private key of the user joanna.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-21 a las 0.29.30.png" alt=""><figcaption></figcaption></figure>

Aparently, it is showing the content of the id\_rsa of the user joanna in some place so by inspecting the configuration file for that site we notice that it is using the port 52846 for internal requests.&#x20;

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-21 a las 0.37.01.png" alt=""><figcaption></figcaption></figure>

Now, simply by making a request to localhost to that port we should be looking at the private key of the user Joanna, as we can see in the image below.&#x20;

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-21 a las 0.40.44.png" alt=""><figcaption></figcaption></figure>

Although now we could try to connect via ssh using this private key, from its header we notice that it is encrypted, so it would be necessary to use a passphrase to connect to joanna, as it is shown below.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-21 a las 0.46.19.png" alt=""><figcaption></figcaption></figure>

So now, we can use john to compute the hash of the id\_rsa to then perform a brute force attack to get the passphrase.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-21 a las 0.47.28.png" alt=""><figcaption></figcaption></figure>

Using john we cracked the password of the id\_rsa so now we should be able to connect via ssh to the user joanna, using her private key and the passphrase returned by john.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-21 a las 0.49.18.png" alt=""><figcaption></figcaption></figure>

We are in as joanna.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-21 a las 0.50.44.png" alt=""><figcaption></figcaption></figure>

Getting the user hash.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-21 a las 0.53.45.png" alt=""><figcaption></figcaption></figure>

Now, to root privileges, we notice that joanna can execute only nano as root to load the resource /opt/priv.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-21 a las 0.54.51.png" alt=""><figcaption></figcaption></figure>

So that, we could change the suid bash privileges using this vulnerability by executing sudo commands using nano. To execute a command with root privileges from nano we open nano and then CTRL + R and CTRL + X and insert the command + ENTER. Now the privileges of the /bin/bash have changed.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-21 a las 0.58.33.png" alt=""><figcaption></figcaption></figure>

Getting root flag.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-21 a las 1.00.30.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/Captura de pantalla 2023-02-21 a las 1.01.15.png" alt=""><figcaption></figcaption></figure>
