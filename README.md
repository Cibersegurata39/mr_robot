# mr_robot

Máquina resuelta de *TryHackMe* en la que se trabaja la enumeración y *fingerprinting*,
<div>
  <img src="https://img.shields.io/badge/-Kali-5e8ca8?style=for-the-badge&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/-Nmap-6933FF?style=for-the-badge&logo=nmap&logoColor=white" />
  <img src="https://img.shields.io/badge/-Dirsearch-005571?style=for-the-badge&logo=dirsearch&logoColor=white" />
  wpscan
  <img src="https://img.shields.io/badge/-php-777BB4?style=for-the-badge&logo=php&logoColor=white" />
  <img src="https://img.shields.io/badge/-Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" />
  <img src="https://img.shields.io/badge/-python-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/-john-F4B942?style=for-the-badge&logo=john&logoColor=white" />
</div>

## Objetivo

Explicar la realización del siguiente _Capture the flag_ perteneciente a la plataforma *TryHackMe*. Esta máquina está basada en la serie de televisión *MR Robot*, en la cual se deberán intentar encontrar hasta tres *flags* diferentes. Para ello, se deberá penetrar en la máquina por medio de una vulnerabilidad de *WordPress* y realizar una escalada de privilegios.

## Que hemos aprendido?

- Realizar *fingerprinting* y enumeración de puertos y enumeración web.
- Utilizr la herramienta *Wpscan*.
- .
- .
- Descifrar contraseñas.
- Buscar archivos por comando.
- Escalada de privilegios.

## Herramientas utilizadas

- *Kali Linux*.
- Enumeración: *Nmap*, *Dirsearch*.
- Penetración: *Wpscan*, *Bash*, *Python3*, *Unzip*, diferentes decodificadores web. 

## Steps

### Enumeración y fingerprinting

La máquina a vulnerar pertenece a la plataforma *TryHackMe*, la propia web te proporciona la IP de la máquina víctima. La conexión a esta se hace mediante una VPN que te proporciona THM y asigna una nueva IP para que tu máquina interactúe con la máquina vulnerable.

El primer paso es lanzar **Nmap** sin *ping* (-Pn) para agilizar el escaneo y ser más silencioso. Además, se quieren conocer las versiones que corren en cada puerto (-sV) y el sistema operativo (-O). No se indica, específicamente, que compruebe todos los puertos, pues en este tipo de máquina no será necesario. Se configura el escaneo para que este sea moderadamente rápido (-T4). Por último, se lanzan los *scripts* por *default* de *nmap* (-sC) para que encuentren posibles vulnerabilidades.

<code>nmap -Pn -sV -O 10.10.57.46 -T4 -sC</code>

<img width="831" height="508" alt="Captura de pantalla 2025-10-14 130344" src="https://github.com/user-attachments/assets/6bf067e0-67eb-4446-902a-c0b108e37fab" />

El comando devuelve 3 puertos TCPs abiertos:

- En el puerto 22 corre la versión Openssh 8.2p1, en un sistema Ubuntu, servicio SSH.
- En el puerto 80 corre un servidor Apache httpd, en un sistema Ubuntu, servicio HTTP.
- En el puerto 443 corre un servidor Apache httpd, en un sistema Ubuntu, servicio SSL/HTTP.

El sistema operativo detectado es un *Creston*, el cual se encarga de controlar interfaces gráficas, gestionar eventos, redes... Los *scripts*, por su parte, no han desvelado ningún tipo de información importante.

Así pues, primero nos dirigimos desde el navegador al puerto 80 de la IP dada y esta nos lleva a la página por defecto del servidor de *Apache*. Aparece una página con una panalla de comandos para interactuar, mostrando estos diferentes videos y fotografías relacionadas con la serie de televisión.

<img width="1847" height="464" alt="Captura de pantalla 2025-10-14 131254" src="https://github.com/user-attachments/assets/a2def164-5f2d-44d4-8010-85ea889fe60f" />

Como es costumbre se verifica el archivo '/robots.txt', el cual nos muestra la primera de las *flags* dentro del archivo de texto *key-1-of-3.txt*. Para obtenerla solo hay que dirigrse a la dirección 'http://10.10.57.46/key-1-of-3.txt'.

**Flag: 073403c8a58a1f80d943455fb30724b9**

Además, se encuentra un diccionario (fsocity.dic) que puede ser útil más adelante y se descarga con <code>wget</code>.

<code>wget http://10.10.57.46/fsocity.dic</code>

Lo siguiente es hacer una enumeración web con **Dirsearch** para encontrar posibles directorios y archivos. *Dirsearch* es lanzado con **Python3** indicando la IP a atacar (-u) y la lista a utilizar para la enumeración (-w).

<code>python3 /usr/lib/python3/dist-packages/dirsearch/dirsearch.py -u 10.10.57.46 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt</code>

De entre los directorios encontrados, además de '/robots.txt' ya visto anteriormente, aparece '/wp-login.php'. Esto nos indica que hay un *WordPRess* funcionando, el cual se podría aprovechar para hacer la penetración a la máquina. Otro directorio interesante encontrado es '/license', en este aparece una pregunta 'do you want a password or something?' y siguiendo con el *scroll* hacia abajo aparece un texto codificado en base64 *ZWxsaW90OkVSMjgtMDY1Mgo=*

<img width="814" height="855" alt="Captura de pantalla 2025-10-14 182300" src="https://github.com/user-attachments/assets/a2b69a5f-7b7e-4bde-a041-24128ec21b39" />

En la imagen se muestra una dirección *IP* distinta pues se tuvo que reiniciar la máquina.

### Vulnerabilidades explotadas

En la primera *flag* se ha aprovechado el decuido de poner información sensible en el arhcivo 'robots.txt' siendo este accesible para cualquiera. Por su parte, el directorio '/license' también es accesible para terceros y lo que parece ser una contraseña, está codificada en base64, lo cual es fácil de descodificar. Para tener éxito se utiliza el comando <code>echo</code> junto con un *pipeline* que indique la decodificación de base64.

<code>echo "ZWxsaW90OkVSMjgtMDY1Mgo=" | base64 -d</code>

Lo que nos devuelve es una combianción de usuario:contraseña: *elliot:ER28-0652*. Primero pruebo iniciar una conexión vía *SSH* pero estas credenciales no son válidas, por lo que tiene pinta que lo serán para el *WordPress* encontrado anteriormente. Antes utilizo la herramienta *Wpscan* para ver que información nos puede recuperar respecto al *Wordpress*. Esta me indica que la versión utilizada es la 4.3.1, la cual es insegura. También muestra el tema configurado 'twentyfiteen', que no hay plugins y otros parámetros.

<code>wpscan --url 10.10.234.12</code>

<img width="999" height="446" alt="Captura de pantalla 2025-10-15 173531" src="https://github.com/user-attachments/assets/fcb7fc22-e719-4ec6-bb55-2d6955ef10e6" />

Una vez visto esto, se prueba hacer *log in* con las credenciales obtenidas y resulta exitoso: hemos iniciado sesión como *Elliot*. Con el acceso total a la configuración del *Wordpress*, se pretende conseguir una *reverse shell* y para ello nos dirigimos a Appearance>Editor. Lo interesante aquí es sustituir el contenido de algún archivo *php* existente con el código de la *reverse shell* y un gran candidato es el '404 Template' (404.php). Este archivo solo se ejecuta cuando no encuentra algun directorio, lo cual podremos forzar para que se ejecute nuestro código. Este lo recuperamos del *Github* de nuestros amigos [*pentestmonkey*](https://github.com/pentestmonkey/php-reverse-shell). Sólo es necesario cambiar las líneas de código donde se indica la dirección *IP* de nuestra máquina atacante y nuestro puerto al que conectarnos.

<img width="1653" height="1233" alt="image" src="https://github.com/user-attachments/assets/660d71f1-bd72-4de8-87e6-79e652b731c6" />

