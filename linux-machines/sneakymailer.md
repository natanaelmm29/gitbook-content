---
description: >-
  Medium machine OSCP style: PyPi, FTP, Password Reuse, Password Cracking, SUDO
  exploitation
---

# SneakyMailer

## Reconnaissance

As always, the first step is to ensure that the machine is up, in this case with a simple ping request. Moreover, because of the TTL value we can determine that it is a Linux machine as the value is 63 (should be 64 but due to the intermediary node it is decremented).

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 17.03.22.png" alt=""><figcaption></figcaption></figure>

The first scan for all open ports reports that the FTP, SSH, IMAP, HTTP (80 and 8080) ports are open.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 17.05.02.png" alt=""><figcaption></figcaption></figure>

Through a version scan we get further information about the services running on that ports.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 17.08.51.png" alt=""><figcaption></figcaption></figure>

As it was not reported by the previous scan, the ftp service has the anonymous login diabled.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 17.13.51.png" alt=""><figcaption></figcaption></figure>

To access to the server on port 80, we need to modify the file `/etc/hosts` to specify the IP and domain to resolve.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 17.17.29.png" alt=""><figcaption></figcaption></figure>

This is the main page of the server. As we can see, we are supposedly loged as some user and the projects are related to pypi and POP3 and SMTP.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 17.17.54.png" alt=""><figcaption></figcaption></figure>

Navigating to the Team tab we can see the list of the employees and their names and emails.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 17.19.08.png" alt=""><figcaption></figcaption></figure>

We can also see some messages that does not contain anything important (inspect the source code).

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 17.19.31.png" alt=""><figcaption></figcaption></figure>

With whatweb we can determine some information about the technology that is being used in the server but nothing interesting.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 17.40.59.png" alt=""><figcaption></figcaption></figure>

We got interested in the email list and we get it and paste to a file.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 17.49.52.png" alt=""><figcaption></figcaption></figure>

## Exploitation

Using `swaks` we try to send an email to every email address on the list to test if we are able to do so and we success.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 17.59.34.png" alt=""><figcaption></figcaption></figure>

The emails are received.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 17.59.59.png" alt=""><figcaption></figcaption></figure>

Knowing that, we can send a phishing email indicating an URL (our host) to test if some user clicks on our link.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 18.03.06.png" alt=""><figcaption></figcaption></figure>

Being listening on the correct port, we receive some request to our host.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 18.03.26.png" alt=""><figcaption></figcaption></figure>

If we use `netcat` instead of python, we can see the content of the POST request and as we can see, there is some user and password inside.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 18.06.47.png" alt=""><figcaption></figcaption></figure>

With those credentials, as we can see they are URL encoded so we use an online decoder to get the plain password.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 18.11.20.png" alt=""><figcaption></figcaption></figure>

With those credentials we can connect to the email account of the user paulbyrd. Moreover, we can list his inboxes, emails, etc.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 18.15.49.png" alt=""><figcaption></figcaption></figure>

Examining the inboxes, we notice there is an inbox with some emails inside (Sent Items)

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 18.18.08.png" alt=""><figcaption></figcaption></figure>

If we inspect the content of the first email we can see some credentials for a user called developer.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 18.19.05.png" alt=""><figcaption></figcaption></figure>

Then, inspecting the second email we do not find something really useful, but an email sent to a user called `low`.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 18.22.53.png" alt=""><figcaption></figcaption></figure>

With the credentials of a user called developer, we try to connect via ssh but the access is denied but then we try it with ftp and we success.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 18.24.55.png" alt=""><figcaption></figcaption></figure>



Within the user account, we notice that we are able to upload files as it can be seen in the image below.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 18.26.57.png" alt=""><figcaption></figcaption></figure>

When accessing the file from the browser the resource is not found and that is rare. At this point I got a bit blocked.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 18.28.47.png" alt=""><figcaption></figcaption></figure>

After a time we run a fuzzer to look for subdomains and we find that there is one called dev.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 18.59.01.png" alt=""><figcaption></figcaption></figure>

We uploaded again the file and in this time we are able to access it from the browser.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 19.05.28.png" alt=""><figcaption></figcaption></figure>

At this point we created a php file to upload it and execute commands from the browser.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 19.09.12.png" alt=""><figcaption></figcaption></figure>

Here we ensure that we are able to run commands on the target machine in the browser.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 19.09.41.png" alt=""><figcaption></figcaption></figure>

At this point we execute a reverse shell to send a shell as www-data to our host.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 19.13.56.png" alt=""><figcaption></figcaption></figure>

Here it can be seen the received shell as www-data.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 19.14.05.png" alt=""><figcaption></figcaption></figure>

As www-data we have no permission to get the user flag so we think that we must switch to low to get it.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 19.18.03.png" alt=""><figcaption></figcaption></figure>

The www-data user has no sudo privileges to abuse.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 19.18.32.png" alt=""><figcaption></figcaption></figure>

Looking for suid privileges but there is nothing interesting.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 19.19.16.png" alt=""><figcaption></figcaption></figure>

With crontab we ensure that there is nothing interesting and looking at the running processes we notice that there is a process refering to pypi with some command running. Apparently, it is taking the credentials from a file and inspecting the file we find some credentials of a user pypi.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 19.23.19.png" alt=""><figcaption></figcaption></figure>

With JtR we crack the password of the user pypi.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 19.25.36.png" alt=""><figcaption></figcaption></figure>

Here we needed to add the subdomain to the /etc/hosts file to access the pypi server. Looking for information on the Internet, it is some kind of server where python packages can be uploaded that can be created privately.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 19.30.35.png" alt=""><figcaption></figcaption></figure>

If we click on the link to view the installed packages we need to input the pypi user credentials but there is no package installed.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 19.38.33.png" alt=""><figcaption></figcaption></figure>

At this point we think that maybe we could create a python package and upload it to the server and abuse it to get a shell. Looking on the Internet we find the process to setup and upload the package.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 19.42.40.png" alt=""><figcaption></figcaption></figure>

This is the structure of the package that must contain to be correctly interpreted.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 19.43.55.png" alt=""><figcaption></figcaption></figure>

We create the same structure on our host and copy the setup.py example code from the website to our setup file.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 19.46.00.png" alt=""><figcaption></figcaption></figure>

At this point, to abuse it we could insert a python reverse shell so when we upload the package to the server, a reverse shell will be established in our host.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 19.49.09.png" alt=""><figcaption></figcaption></figure>

This is the content of the setup file, with the example code and the reverse shell code (the word python before the second import must be removed).

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 19.52.19.png" alt=""><figcaption></figcaption></figure>

After that, we must create a file called .pypirc to specify the information about the package and the repository server that is going to be uploaded to, in this case the pypi server. Moreover, we must specify the credentials of the user pypi to access that repo.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 19.57.08.png" alt=""><figcaption></figcaption></figure>

At the time of executing the setup, we receive a shell but on our host. However, when exiting this shell, the reverse shell as low is sent to our host, so we needed to set up another listener at the time of obtaining the shell as our root user.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 20.03.02.png" alt=""><figcaption></figcaption></figure>

As low user, we get the flag.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 20.06.16 (1).png" alt=""><figcaption></figcaption></figure>

## Privilege escalation

Here we need to escalate to root to obtain admin privileges. We start using id to inspect the groups which the user low belongs to. Then, with sudo -l we look for sudo privileges and there is one called pip.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 20.12.47.png" alt=""><figcaption></figcaption></figure>

Looking on the Internet we find a way of exploiting pip to get a shell.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 20.17.34.png" alt=""><figcaption></figcaption></figure>

We simply need to run the commands in the image above specifying that the version of pip is pip3 and executed with sudo. At this point we get the root flag and PWNED!

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 20.18.56.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-18 a las 20.18.49.png" alt=""><figcaption></figcaption></figure>
