# Server-side Request Forgery (SSRF)

SSRF, which stands for Server-Side Request Forgery, is a web application vulnerability that allows an attacker to make arbitrary requests from the targeted server. It occurs when the application accepts user-supplied input and includes it in a server-side request, without properly validating or sanitizing the input. Attackers can exploit SSRF to manipulate the server into making requests to internal resources or external systems that should be inaccessible. This can lead to data leakage, unauthorized access to sensitive information, or even remote code execution. To mitigate SSRF vulnerabilities, it is important to implement strict input validation, enforce server-side access controls, and utilize whitelisting or blacklisting techniques to restrict the allowed target hosts or IP addresses for outgoing requests.

#### Lab: Basic SSRF against the local server

The check stock feature of the application is calling an API using the following URL.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-06-29 a las 17.47.30.png" alt=""><figcaption></figcaption></figure>

By replacing the URL with the URL provided in the lab statement, we can see that we are able to access to the admin panel.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-06-29 a las 17.48.01.png" alt=""><figcaption></figcaption></figure>

Inspecting the raw content of the page, we can see what request is the app using to remove users, which is the goal of this lab.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-06-29 a las 17.49.15.png" alt=""><figcaption></figcaption></figure>

By using that path in the request, we can solve the lab.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-06-29 a las 17.50.57.png" alt=""><figcaption></figcaption></figure>

#### Lab: Basic [SSRF](https://portswigger.net/web-security/ssrf) against another back-end system

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-07-02 a las 11.07.36.png" alt=""><figcaption></figcaption></figure>

Now we know that there is an IP address within the internal network with the admin interface but we do not know it. In this case, we are going to use de Burp Intruder to perform a brute force attack changing the last digit of the IP address to discover which IP address within the network is the one with the admin interface.&#x20;

The first step is to asign the payload variable to the last digit of the IP.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-07-02 a las 11.23.37.png" alt=""><figcaption></figcaption></figure>

Then, we configure the payload set to be a number list from 1 to 254.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-07-02 a las 11.25.11.png" alt="" width="375"><figcaption></figcaption></figure>

Once the attack is started, we can see that all of the response status codes are 500 due to an internal error. However, if we look at the table below, we can see that the status code for the IP ending in 167 is 200 (success). Then the same steps as before are followed.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-07-02 a las 11.31.22.png" alt="" width="375"><figcaption></figcaption></figure>

#### Lab: [SSRF](https://portswigger.net/web-security/ssrf) with blacklist-based input filter

This lab is basically the same as the first one but to exploit the SSRF we need to bypass to weak filters. The first one is related to the IP address: localhost can be also written as 127.0.0.1, but it does not work. Another way of writing 127.0.0.1 is removing the "0" and writing the IP as 127.1. The second filter is related to the "admin" resource name.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-07-02 a las 11.53.35.png" alt="" width="375"><figcaption></figcaption></figure>

To solve the lab we only have to add the URL to delete the required user as in the first lab.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-07-02 a las 12.03.04.png" alt="" width="375"><figcaption></figcaption></figure>

#### Lab: [SSRF](https://portswigger.net/web-security/ssrf) with filter bypass via open redirection vulnerability

In this case we can see that URLs are not used in the stockApi parameter. By trying to perform a request to a different host from this parameter as we did in the other exercises, we notice that we are not able.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-07-02 a las 12.22.41.png" alt="" width="375"><figcaption></figcaption></figure>

We need to find a place in the application that is performing a redirection in order to test if we can redirect to the admin panel provided in the statement. By clicking in the _Next product_ link, we observe that a redirection is made.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-07-02 a las 12.31.41.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-07-02 a las 12.31.24.png" alt=""><figcaption></figcaption></figure>

By using this request but providing the URL of the admin panel to the redirection parameter _path_ we can see that we get access to the admin panel. Finally, we only have to delete the user as in the other labs.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-07-02 a las 12.31.03.png" alt=""><figcaption></figcaption></figure>
