# SQL Injection

## Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

In order to perform the exploitation we only need to intercept the request with Burp and replace the product category with `' or 1=1--` to alter the SQL query which is executed in the server.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-06-02 a las 12.26.51.png" alt="" width="375"><figcaption></figcaption></figure>

## Lab: SQL injection vulnerability allowing login bypass

By accessing to the _My Account_ link to log in as the administrator, we only have to enter the username that in this case is the `administrator` as it is said in the lab description and in the password field the sentence `'or 1=1--` to break the SQL query that is performed and get access to the admin zone.&#x20;

<figure><img src=".gitbook/assets/Captura de pantalla 2023-06-02 a las 12.33.36.png" alt=""><figcaption></figcaption></figure>

## Lab: SQL injection UNION attack, determining the number of columns returned by the query

The most used way to determine how many columns a table has is the use of the union statement with a select with NULL values, as it can be seen in the image below, using the resulting query `' UNION SELECT NULL--`. The server will respond with an error until we use the same null values of the number of columns that the table has.

<figure><img src=".gitbook/assets/Captura de pantalla 2023-06-02 a las 12.40.41.png" alt="" width="375"><figcaption></figcaption></figure>

In this case, the table has three columns as the number of NULL values in the query to have a successful response is three.&#x20;

<figure><img src=".gitbook/assets/Captura de pantalla 2023-06-02 a las 12.45.15.png" alt="" width="375"><figcaption></figcaption></figure>

