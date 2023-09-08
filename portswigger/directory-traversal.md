# Directory Traversal

Directory traversal, also known as path traversal, is a vulnerability that occurs when an application fails to properly validate user input and allows attackers to access files or directories beyond the intended boundaries. By exploiting this vulnerability, attackers can navigate up the directory structure and access restricted files or directories. This can lead to unauthorized disclosure of sensitive information or even remote code execution. To mitigate this vulnerability, proper input validation and sanitization techniques should be implemented, access to directories outside the intended scope should be restricted, and secure coding practices should be followed. Using frameworks or libraries with built-in protection against directory traversal can also help prevent such attacks.

### Laboratories

#### Lab: [File path traversal](https://portswigger.net/web-security/file-path-traversal), traversal sequences blocked with absolute path bypass

The vulnerability remains when loading an image, which is using the URL shown in the image, using the _filename_ parameter to state the name of the image to load.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-06-29 a las 16.41.20.png" alt="" width="375"><figcaption></figcaption></figure>

Using the typical ../ to move up in the filesystem to reach the desired file is not working due to sanitization. As a result, the obtained response returns back "no such file".

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-06-29 a las 16.43.55.png" alt=""><figcaption></figcaption></figure>

As it is using absolute paths, by simply using the absolute path of the desired file in the parameter we obtain the content of the _/etc/passwd_ file.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-06-29 a las 16.46.05.png" alt=""><figcaption></figcaption></figure>

#### Lab: [File path traversal](https://portswigger.net/web-security/file-path-traversal), traversal sequences stripped non-recursively

In this case, the server is sanitizing the path by non-recursively stripping the input. It is apparently removing the _../_ but only once. In this case, we can bypass the filter by using \[ ....// ] instead of \[ ../ ] so when the filter is applied the resulting input would be ../

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-06-29 a las 16.56.47.png" alt=""><figcaption></figcaption></figure>

#### Lab: [File path traversal](https://portswigger.net/web-security/file-path-traversal), traversal sequences stripped with superfluous URL-decode

In this case the ../ are removed but using URL-encoding we can bypass the filter. It is worth noting that to solve this lab we used double-URL-encoding to encode both the "/" and then the "%" symbol of the resulting encoding. It works only encoding the "/" or the entire "../".

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-06-29 a las 17.08.29.png" alt=""><figcaption></figcaption></figure>

#### Lab: [File path traversal](https://portswigger.net/web-security/file-path-traversal), validation of start of path

In this case there is a start directory validation by which only resources starting with _/var/www/images_ are allowed. It is very simple to bypass this filter as the "../" can be used in any part of the path, solving the lab as shown in the following image.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-06-29 a las 17.15.09.png" alt=""><figcaption></figcaption></figure>

#### Lab: [File path traversal](https://portswigger.net/web-security/file-path-traversal), validation of file extension with null byte bypass

In this case the application is checking for the extension of the file, looking for the image extension before loading the resource. The NULL BYTE technique is used to isolate the path of the _/etc/passwd_ to the _.jpg_ extension. The solution is the following:

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-06-29 a las 17.20.47.png" alt=""><figcaption></figcaption></figure>
