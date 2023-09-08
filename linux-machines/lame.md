---
description: Easy machine OSCP style
---

# Lame

## Reconocimiento

El primer paso es comprobar que la máquina está encendida y conocer ante qué tipo de máquina estamos. En este caso, enviamos un ping a la máquina víctima y en la respuesta podemos observar que el TTL es 63, por lo que se trata de una máquina Linux.&#x20;

Las máquinas Linux tienen TTL 64 pero debido al salto a través de la gateway de HTB se resta un TTL (63). Cuando se trata de una máquina Windows, el TTL será 128.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 12.18.24.png" alt=""><figcaption></figcaption></figure>

Una vez nos hemos asegurado que la máquina está encendida, podemos comenzar con el scanning haciendo uso de nmap. Para ello, escaneamos la máquina víctima en busca de puertos abiertos (que normalmente son los que nos interesan) haciendo uso de una serie de parámetros (ver imagen).

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 12.36.07.png" alt=""><figcaption></figcaption></figure>

Una vez conocidos aquellos puertos que se encuentran abiertos en la máquina víctima, pasamos a escanear dichos puertos haciendo un reconocimiento de versión para obtener información más detallada de los servicios que están corriendo en dichos puertos.&#x20;

Como podemos ver en la imagen, a través de este escaneo nmap obtenemos las versiones y más información sobre los servicios objetivo. **En un primer momento descartamos el uso de ssh debido a que no se dispone de credenciales válidas.**

De la imagen podemos ver que se trata de un Ubuntu Debian. También podemos ver que nos podemos conectar a través de ftp haciendo uso de un usuario anónimo.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 12.42.04.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 12.48.10.png" alt=""><figcaption></figcaption></figure>

Aparentemente haciendo uso del usuario anónimo no se disponen de permisos de escritura por lo que pasamos a buscar posibles exploits para la versión de ftp que esta corriendo en el puerto correspondiente.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 12.54.17.png" alt=""><figcaption></figcaption></figure>

Nos copiamos el fichero en nuestro directorio de trabajo y lo analizamos para aprender qué es lo que hace el script the python para explotar la vulnerabilidad. En la imagen podemos ver que hace simplemente abre una conexión con el puerto FTP haciendo uso del usuario nergal:) y en esta versión se abre el puerto 6200 que es el que va a usar el exploit para abrir una consola interactiva.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 13.01.47.png" alt=""><figcaption></figcaption></figure>

Ahora vamos a comprobar qué está ahciendo el script de forma manual a través de netcat.

![](<../.gitbook/assets/Captura de pantalla 2023-02-17 a las 13.03.22.png>)

Como vemos, no responde al utilizar el usuario y contraseña que utiliza el script, así que vamos a hacer un escaneo para comprobar si ahora el puerto 6200 está abierto. Como vemos, el puerto no está abierto así que por ahora vamos a intentar seguir buscando vulnerabilidades en los otros  servicios encontraos por nmap.

![](<../.gitbook/assets/Captura de pantalla 2023-02-17 a las 13.06.28.png>)

Ahora vamos a buscar exploits para la versión de Samba que está corriendo en la víctima.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 13.08.06.png" alt=""><figcaption></figcaption></figure>

Como vemos, nos interesamos por el Command Execution y aunque sea un exploit de Metasploit y no se pueda usar, si que se puede inspeccionar el contenido para comprender qué es lo que hace para posteriormente explotarlo de forma manual.

Se trata de un archivo en ruby y la seccion interesante es la función exploit que es la que lo lleva a cabo. Como podemos ver, en primer lugar al parecer está abriendo una conexión y está declarando como usuario a la ristra de caracteres entre comillas. El nohup lo que normalmente permite es obtener una consola interactiva sin que te la mate, pero no siempre es así.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 13.11.43.png" alt=""><figcaption></figcaption></figure>

SI accedemos a través de smbclient podemos listar los recursos compartidos, por lo que vamos a intentar acceder al directorio tmp

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 13.45.54.png" alt=""><figcaption></figcaption></figure>

Como vemos, nos hemos podido conectar, pero sin especificar el usuario. Como hemos visto en el exploit, este hace uso de un usuario en concreto.

Una vez accedido al directorio, y haciendo uso del comando help podemos ver que hay una opcion que nos permite loguearnos llamda logon, pasando el username al comando.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 13.48.07.png" alt=""><figcaption></figcaption></figure>

Si tratamos de incrustar en el usuario un comando para intentar ejecutar código en la máquina víctima y abrimos un servidor en escucha por el 443 en el host, podemos ver cómo nos devuelve el resultado de ejecutar el comando en la máquina target.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 14.22.47.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 14.24.17.png" alt=""><figcaption></figcaption></figure>

Si ahora en vez de tratar de ejecutar un comando cualquiera lo que hacemos es enviar una shell al servidor en escucha obtenemos lo siguiente:

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 14.34.39.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 14.34.52.png" alt=""><figcaption></figcaption></figure>

Como podemos ver, ya hemos accedido a la máquina víctima. Con script /dev/null -c bash conseguimos una pseudoconsola pero el CTRL + L no funciona y CTRL + C sale de consola. Para evitarlo, CTRL + Z para suspender la sesión y ejecutamos el comando:

```
stty raw -echo; fg
```

Y a continuación escribimos **reset xterm** y se nos abre una terminal con todas estas funcionalidades, pero la consola sigue sin ser interactiva si por ejemplo abrimos nano. Con stty vemos las dimensiones de nuestra pantalla.

Una vez dentro del sistema y buscando por los distintos directorios, obtenemos el flag tanto de user como de root y pwned.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 14.55.22.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 14.57.50.png" alt=""><figcaption></figcaption></figure>

Una vez hemos finalizado, para borrar cualquier rastro o evuidencia dentro del sistema ejecutamos:

```
(rm -rf /*) 2>/dev/null
```

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 14.58.16.png" alt=""><figcaption></figcaption></figure>
