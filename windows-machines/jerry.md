---
description: >-
  Easy machine OSCP style: Tomcat, SILENTTRINITY, mimikatz, Java, Brute Force,
  Password dump, RCE, AFU, default credentials.
---

# Jerry

## Reconnaissance

The first step is to ensure that the host is up by making a ping request. Through the TTL value we can determine that it is a Windows machine as it is 127 (should be 128 but due to the intermediary node it is decremented in one unit).

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 22.25.53.png" alt=""><figcaption></figcaption></figure>

With the first scan we can determine that there is only a port opened which is the port 8080.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 22.28.25.png" alt=""><figcaption></figcaption></figure>

Then we perform a version scan to get further information about the service running on port 8080. As we can see in the image below, it is a tomcat server on version 7.0.88.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 22.29.34.png" alt=""><figcaption></figcaption></figure>

Apparently, using the utility searchsploit, this version is vulnerable to Remote Code Execution.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 22.30.33.png" alt=""><figcaption></figcaption></figure>

Tomcat servers has a manager panel which is normally under the `/manager` directory, and normally in the `/manager/html` page. When trying to access this panel, a login page is shown but we have not credentials. However, there is a panel which shows information, and as we can see in the image below, it is proposed some default credentials.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 23.02.58.png" alt=""><figcaption></figcaption></figure>

By using these credentials, we get access to the panel.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 23.12.12.png" alt=""><figcaption></figcaption></figure>

From this admin panel, we are able to upload a war file to deploy an application. We could use this upload privilege to upload a malicious war file which gives us a shell.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 23.12.28.png" alt=""><figcaption></figcaption></figure>

Using `msfvenom` we list the available payloads but in this case we are interested in java payloads, as they are used to distribute Java servlets and JavaServer pages.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 23.15.58.png" alt=""><figcaption></figcaption></figure>

We use the jsp reverse shell stating the IP address of our host and the port in which we are going to be listening.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 23.19.00.png" alt=""><figcaption></figcaption></figure>

Once uploaded the war file, we can see in the table of applications the shell app which has been uploaded. Having set up the corresponding listener on port 443, once the `/shell` is clicked, the shell is obtained.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 23.20.33.png" alt=""><figcaption></figcaption></figure>

As the administrator, we are able to get both flags and PWNED!

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 23.23.31.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-04-13 a las 23.23.25.png" alt=""><figcaption></figcaption></figure>
