---
description: Medium machine OSCP style
---

# Ready

## Recoinnassance

First we are going to ensure that the machine is up by pinging it. Moreover, from the TTL we can determine that it is a Linux machine as it is 63. In fact, it should be 64 but there is an intermediary node between the machine and the host from the HTB VPN so that is because it is 63.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-21 a las 19.58.52.png" alt=""><figcaption></figcaption></figure>

This is the route the packet does from the host to the target machine and the reason of the TTL decrementing in one.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-21 a las 20.00.08.png" alt=""><figcaption></figcaption></figure>

Now we are going to scan the target machine to locate which ports are open to later find possible service vulnerabilities to exploit. In this case, the scan returned that the ports 22 and 5080 are open.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-21 a las 20.05.40.png" alt=""><figcaption></figcaption></figure>

With the open ports, we perform a service version scan in order to get more information about the services that are running in the open ports in the target machine. To do that, we must use the parameters -sC and -sV to get the versions and more detailed information. As we can see, in the port 5080 there is an HTTP service running

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-21 a las 20.08.40.png" alt=""><figcaption></figcaption></figure>

With the **whatweb** utility we can know if a content manage system is being used in that server. As we can learn from the response, there is a redirect directive which redirects the request to http://10.10.10.220:5080/users/sign\_in and as we can see in the result, it is mentioning something about GitLab.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-21 a las 20.12.31.png" alt=""><figcaption></figcaption></figure>

If we now access via the browser to the http server we notice that we are redirected to the sign in page of gitlab.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-21 a las 20.21.48.png" alt=""><figcaption></figcaption></figure>

At this point, we could try to look for exploits for GitLab so we start inspecting the source code and some paths that are available.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-21 a las 20.23.15.png" alt=""><figcaption></figcaption></figure>

As we can see in the image above, there is a path called api but we have not privileges to access to it, so we can now search on the Internet for ways of getting the version of the GitLab running using the api. We found that to get the version we must be logged and then we would have to make a request including a private token, so the first thing to do is to register.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-21 a las 20.25.05.png" alt=""><figcaption></figcaption></figure>

Once registered, we can generate a private token to grant access to the api using the GitLab dashboard panel.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-21 a las 20.28.17.png" alt=""><figcaption></figcaption></figure>

Finally, to get the GitLab version, we only have to make a request especifying the access token, simply using curl command. Now we have the version 11.4.7.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-21 a las 20.31.00.png" alt=""><figcaption></figcaption></figure>

Now using searchsploit we can look for exploits for that specific version, and yes, there are two Remote Code Execution.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-21 a las 20.33.19.png" alt=""><figcaption></figcaption></figure>

Inspecting the content of the exploit we notice that there are some required parameters that are used to perform it, like the username, the token, the local port, the local IP and some others, so we might think that it gets a reverse shell in some way, so we proceed to modify the parameters.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-21 a las 20.37.39.png" alt=""><figcaption></figcaption></figure>

When modifying the corresponding parameters, we notice that we do not have the cookie and the authenticity token, so we have to look for them, for example, in the source code.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-21 a las 20.42.40.png" alt=""><figcaption></figcaption></figure>

As we are logged as a user, inspecting the storage we can get the session cookie value as well.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-21 a las 20.45.52.png" alt=""><figcaption></figcaption></figure>

The parameters end up looking like this:

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-21 a las 20.48.50.png" alt=""><figcaption></figcaption></figure>

After 10 hours trying to perform the exploit, I never got the reverse shell back, SAD me!

## Exploiting

## Privilege escalation

