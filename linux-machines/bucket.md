---
description: 'Cloud Pentesting Seminar Write-up: Cloud AWS'
---

# Bucket

## Reconnaissance

First of all, ensuring that the host is up by pinging it. It is a linux machine as the TTL is 63 (should be 64 as the TTL is decreased in one because of the intermediate node between the host and the machine).

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-28 a las 12.57.09.png" alt=""><figcaption></figcaption></figure>

First scan looking for open ports using nmap. Only ports 22 and 80 are open so we are going to focus on the port 80 as 22 is for SSH and we do not have credentials yet.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-28 a las 12.59.13.png" alt=""><figcaption></figcaption></figure>

Through the version scan we can determine which version of the server is running.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-28 a las 13.01.32.png" alt=""><figcaption></figcaption></figure>

If we try to connect to the web server via the browser, we cannot reach it because the machine does not know the IP address of `bucket.htb.`

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-28 a las 13.03.29.png" alt=""><figcaption></figcaption></figure>

To solve this, it only has to be added to the `/etc/hosts`. Having added this line, we can now connect to the web server which looks like this.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-28 a las 19.03.42.png" alt=""><figcaption></figcaption></figure>

Looking to the source code of the page we notice that the images that are being loaded are stored in a AWS bucket.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-28 a las 13.08.02.png" alt=""><figcaption></figcaption></figure>

After adding the corresponding line to the`/etc/hosts` file, we try to connect to that server and the respond is a JSON with the status of the s3 bucket which is running.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-28 a las 13.11.48.png" alt=""><figcaption></figcaption></figure>

With `wfuzz` we look for interesting directories.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-28 a las 18.59.42.png" alt=""><figcaption></figcaption></figure>

If we inspect these directories, we notice that in the `/health` directory there is some interesting information showing the service which are running. There is a service called **dynamodb**. This service refers to the NoSQL database which is used by Amazon.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-28 a las 19.12.28.png" alt=""><figcaption></figcaption></figure>

Now, if we look into the /shell directory we got access to a web shell to the dynamodb service. To learn how to interact with the database we look on the Internet for documentation.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-28 a las 19.18.01.png" alt=""><figcaption></figcaption></figure>

With the `aws-cli` we can connect to the endpoint and interact with the database. At this point, we can list the existing tables and as we can see in the following image, there is a table called users.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-28 a las 20.01.13.png" alt=""><figcaption></figcaption></figure>

After spending some time getting used to aws cli, and as we know that there is a table called users, we want to list every item inside the table so that we could now some information about the users that are stored. As we can see in the following image, we can see the users and the passwords of some accounts.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-28 a las 20.12.16.png" alt=""><figcaption></figcaption></figure>

Knowing such information, we can try to connect with these users and passwords via ssh. After those attempts, we could not access it. At this point we want to list the active buckets.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-28 a las 20.23.58.png" alt=""><figcaption></figcaption></figure>

With this information we want to know what kind of operations can we do on the available buckets looking for a way of accessing it and exploiting it. With the command `s3 ls` we can list the directories of the bucket and the stored files. As we can list the different files inside, we now want to try if we are capable of uploading stuff into the bucket so that there is a way of exploiting it.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-28 a las 20.34.11.png" alt=""><figcaption></figcaption></figure>

## Exploitation

After some time spent on the Internet learning the commands to interact with buckets, we achieved to upload a file to the bucket.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-28 a las 20.44.23.png" alt=""><figcaption></figcaption></figure>

Accessing the file content via the browser.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-28 a las 20.51.24.png" alt=""><figcaption></figcaption></figure>

With this file upload vulnerability, we got to upload a php reverse shell unless the server does not seem to be a php server. By making a request to that url and listening on the port to which the reverse shell is supposed to be sent, the shell is obtained.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-28 a las 21.19.10.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-28 a las 21.19.35.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-28 a las 21.19.54.png" alt=""><figcaption></figcaption></figure>

Got a shell as www-data.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-28 a las 21.38.21.png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

After getting access as www-data, the next step was to switch to the user roy. The first thing to do is to use the previous passwords that were obtained in previous steps and try to log in as roy. Finnaly, we achieved to log in using the password for sysadmin user.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-29 a las 10.53.58.png" alt=""><figcaption></figcaption></figure>

Now we want to escalate to root. After a time spent looking for directories, root privileges for the user roy and so on, we got to the folder of the bucket called /bucket-app which is owned by root but it is readable and executable by roy.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-29 a las 11.22.28.png" alt=""><figcaption></figcaption></figure>

Moreover, by taking a look at the apache configuration file we notice that the bucket application is listening on localhost port 8000 and  the `AssignUserId root root` means that it is run by root.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-29 a las 11.23.53.png" alt=""><figcaption></figcaption></figure>

Looking at the content of the index.php file which is included in the bucket application folder, we notice that there is some php code. This code is basically managing the POST requests to the server with the parameter action as `get_alerts` to read from the table alerts all the entries with the word "Ransomware" included in the title. For each entry the program creates a random file which is stored under the folder /files whose content is the content of the column **data**. Finally it converts it to a pdf called result.pdf.

To test if this could give us some clue to escalate privileges, the first step is to create the table alerts as it does not exists yet.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-29 a las 11.29.16.png" alt=""><figcaption></figcaption></figure>

To create the table alerts, the AWS cli is used.

```bash
aws dynamodb create-table \
    --table-name alerts \
    --attribute-definitions \
        AttributeName=title,AttributeType=S \
        AttributeName=data,AttributeType=S \
    --key-schema \
        AttributeName=title,KeyType=HASH \
        AttributeName=data,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=5,WriteCapacityUnits=5 \
    --table-class STANDARD \
    --endpoint-url http://s3.bucket.htb
```

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-29 a las 11.45.23.png" alt=""><figcaption></figcaption></figure>

Adding the entry to the alerts table.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-29 a las 11.53.09.png" alt=""><figcaption></figcaption></figure>

Verifying the entry of the database.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-29 a las 12.19.45.png" alt=""><figcaption></figcaption></figure>

Verifying that the content of the entry is exported to the result.pdf.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-29 a las 12.32.23.png" alt=""><figcaption></figcaption></figure>

The commands used are the following (we have to repeat the process every time as the bucket is erased in periods of around a minute).

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-29 a las 12.38.59.png" alt=""><figcaption></figcaption></figure>

Making the request to the server to export the entries to the pdf.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-29 a las 12.39.32.png" alt=""><figcaption></figcaption></figure>

Retrieving the pdf to the host.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-29 a las 12.39.53.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-29 a las 12.37.39.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-29 a las 12.46.59.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-29 a las 12.47.32.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-29 a las 12.47.52.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-29 a las 12.48.15.png" alt=""><figcaption></figcaption></figure>

Accessing via ssh to the root user using the rsa private key.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-29 a las 12.46.25.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-03-29 a las 12.45.59.png" alt=""><figcaption></figcaption></figure>
