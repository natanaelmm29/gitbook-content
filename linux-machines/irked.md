---
description: Easy machine OSCP style
---

# Irked

## Reconocimiento

Con el comando ping nos aseguramos que la máquina está up y de esta ping request podemos ver del TTL que se trata de una máquina Linux (TTL 63), cuyo TTL debería ser 64 pero se resta uno debido al salto que realiza la solicitud por la propia gateway de HTB.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 22.10.04.png" alt=""><figcaption></figcaption></figure>

A través de un escaneo de puertos obtenemos aquellos que están abiertos en la máquina víctima para posteriormente buscar servicios que estén corriendo en dichos puertos y que puedan ser vulnerables.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 22.18.45.png" alt=""><figcaption></figcaption></figure>

Ahora, a través de un escaneo de versión sobre los puertos abiertos, obtenemos información detallada sobre los servicios que se encuentran corriendo en la máquina.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 22.23.02.png" alt=""><figcaption></figcaption></figure>

Como podemos ver, el puerto 80 está abierto y si accedemos a través del browser podemos ver lo siguiente:

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 22.25.40.png" alt=""><figcaption></figcaption></figure>

Como vemos en los puertos abiertos, hay un servicio llamado UnrealIRCd así que procedemos a buscar algún exploit por internet para este servicio.&#x20;

Tras hacer una pequeña búsqueda, damos con un repositorio de GitHub con un exploit de backdoor para este servicio en una versión determinada. Aunque no sabemos que versión está corriendo en nuestra máquina, podemos realizar una prueba del exploit para ver si funciona.&#x20;

Echando un vistazo al exploit, podemos ver que el script necesita de dos parámetros para funcionar: nuestra IP y puerto donde se estará escuchando para la reverse shell. Además, podemos ver que existen varios tipos de payload: python, bash y netcat.&#x20;

Una vez modificado el script, abrimos el 443 en escucha y ejecutamos el script, pasando como parámetro el payload como bash e indicando la IP y el puerto donde está corriendo el servicio.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 22.50.19.png" alt=""><figcaption></figcaption></figure>

Como vemos en la siguiente imagen, hemos obtenido una terminal y si ejecutamos el comando whoami, podemos observar que está logueado como el usuario ircd.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 22.50.36.png" alt=""><figcaption></figcaption></figure>

Una vez dentro del sistema, vamos a hacer que la shell tenga más funcionalidades, ejecutamos:

```
script /dev/null -c bash
CTRL + Z
stty raw -echo; fg
    reset xterm
    
export TERM=xterm
export SHELL=bash
```

Con stty colocamos la shell para que tenga el tamaño de pantalla de nuestro host.

Buscamos la flag con el siguiente comando:

```
find \-name user.txt
```

Como no tenemos permisos, buscamos otras opciones.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 23.30.10.png" alt=""><figcaption></figcaption></figure>

Inspeccionando los directorios, vemos que hay un fichero llamado .backup.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 23.31.32.png" alt=""><figcaption></figcaption></figure>

Como vemos, nos da una serie de pistas donde se menciona la esteganografía. Con la herramienta exiftool podemos leer, escribir y manipular los metadatos de, en este caso, la imagen. Como vemos en las siguientes imágenes, no observamos nada interesante.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 23.36.29.png" alt=""><figcaption></figcaption></figure>

Con el comando strings mostramos las cadenas de caracteres imprimibles que contenga la imagen y de nuevo, nada interesante.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 23.38.00.png" alt=""><figcaption></figcaption></figure>

Con la herramienta steghide **info** obtenemos la información sobre la imagen. Como passphrase introducimos la ristra de caracteres que habíamos obtenido anteriormente y nos devuelve que contiene un fichero incrustado llamado pass.txt.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 23.39.54.png" alt=""><figcaption></figcaption></figure>

Para extraer el fichero incrustado usamos la herramienta steghide de nuevo con la opción extract y la passhphrase utilizada anteriormente.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 23.41.02.png" alt=""><figcaption></figcaption></figure>

Si mostramos el contenido del fichero pass.txt podemos ver una contraseña que podemos intuir que es la contraseña del otro usuario.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 23.41.41.png" alt=""><figcaption></figcaption></figure>

Si cambiamos de usuario ahora e introducimos la contraseña, logramos acceder a la cuenta del usuario djmardov.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 23.46.17.png" alt=""><figcaption></figcaption></figure>

Ahora ya podemos mostrar el contenido del fichero user.txt para obtener la flag del user.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 23.46.47.png" alt=""><figcaption></figcaption></figure>

## Escalada de privilegios

Ahora, para obtener el flag del usuario root probamos si existe, en primer lugar, el comando sudo. Ahora vamos a tratar de buscar algún privilegio suid para poder abusar del mismo.&#x20;

En una primera instancia, podemos ver que hay algo fuera de lo común como es el comando **viewuser.**&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 23.50.33.png" alt=""><figcaption></figcaption></figure>

Si lo ejecutamos, nos dice que el fichero /tmp/listusers no existe, así que lo creamos y le damos permisos de ejecución para ver que sucede al ejecutarlo.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 23.56.28.png" alt=""><figcaption></figcaption></figure>

El contenido del fichero es un script en bash que lo que hace es ejecutar un comando para otorgar privilegios de suid a la bash, conseguiremos ejecutar una bash de forma temporal con privilegios del propietario, que es root.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 23.58.00.png" alt=""><figcaption></figcaption></figure>

Aquí podemos observar como la /bin/bash se torna a suid y se puede ejecutar.

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-17 a las 23.59.37.png" alt=""><figcaption></figcaption></figure>

Finalmente ya podemos ejecutar bash -p y obtener el flag del usuario root.&#x20;

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-18 a las 0.01.20.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Captura de pantalla 2023-02-18 a las 0.01.35.png" alt=""><figcaption></figcaption></figure>
