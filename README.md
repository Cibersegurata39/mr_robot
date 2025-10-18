# Mr_robot

Máquina resuelta de *TryHackMe* en la que se trabaja la enumeración y *fingerprinting*, *reverse shell*, la escucha de puertos, decodificación de contraseñas y escalada de privilegios por medio del bit SUID.
<div>
  <img src="https://img.shields.io/badge/-Kali-5e8ca8?style=for-the-badge&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/-Nmap-6933FF?style=for-the-badge&logo=nmap&logoColor=white" />
  <img src="https://img.shields.io/badge/-Dirsearch-005571?style=for-the-badge&logo=dirsearch&logoColor=white" />
  <img src="https://img.shields.io/badge/-Wpscan-008080?style=for-the-badge&logo=wpscan&logoColor=white" />
  <img src="https://img.shields.io/badge/-Netcat-F5455C?style=for-the-badge&logo=netcat&logoColor=white" />
  <img src="https://img.shields.io/badge/-php-777BB4?style=for-the-badge&logo=php&logoColor=white" />
  <img src="https://img.shields.io/badge/-Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" />
  <img src="https://img.shields.io/badge/-python-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/-john-F4B942?style=for-the-badge&logo=john&logoColor=white" />
</div>

## Objetivo

Explicar la realización del siguiente _Capture the flag_ perteneciente a la plataforma *TryHackMe*. Esta máquina está basada en la serie de televisión *MR Robot*, en la cual se deberán intentar encontrar hasta tres *flags* diferentes. Para ello, se deberá penetrar en la máquina por medio de una vulnerabilidad de *WordPress* y realizar una escalada de privilegios.

## Que hemos aprendido?

- Realizar *fingerprinting* y enumeración de puertos y enumeración web.
- Utilizar la herramienta *Wpscan*.
- Realizar *reverse shell*.
- Poner puertos en escucha.
- Decodificar y desencriptar contraseñas.
- Buscar archivos por comando.
- Escalada de privilegios.

## Herramientas utilizadas

- *Kali Linux*.
- Enumeración: *Nmap*, *Dirsearch*.
- Penetración: *Wpscan*, *Python3*, *php*, *Netcat*, *John*, *Bash*. 

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

**Flag 1: 073403c8a58a1f80d943455fb30724b9**

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

Desde nuestra máquina se pone en escucha el puerto indicado en la *reverse shell* con la herramienta *netcat*. Sólo queda poner una url inexistetne para producir el error 404 y se produzca la conexión entre máquinas, por ejemplo 'https://10.10.116.104/wp-admi'.

<code>nc -lvnp 1234</code>

Se obtiene acceso como usuario 'daemon' pero nos interesa pivotar a un usuario del sistema. En la carpeta /home se muestra el directorio del usuario 'robot' y dentro de este encontramos el archivo 'key-2-of-3.txt' (el cual no tenemos permisos de lectura) y el archivo 'password.raw-md5'. Si leemos este segundo archivo tenemos el siguiente *hash* 'c3fcd3d76192e4007dfb496cca67e13b'.
Para desencriptarlo se utiliza la herramienta *Jhon*, indicando el formato del *hash* y la lista en la que basarse para llevar a cabo la acción. Como resultado se consigue la contraseña del usuario 'robot', *abcdefghijklmnopqrstuvwxyz*. Ya hemos conseguido pivotar de usuario.

<code>john -- format=raw-md5 -- wordlist=/usr/share/wordlists/rockyou. txt hash.txt</code>

<img width="778" height="510" alt="image" src="https://github.com/user-attachments/assets/8e5e83a2-12d8-46aa-9d3d-c3d3f681c8bd" />

Ahora ya somos capaces de leer el archivo que contenía la segunda bandera con el comando <code>cat</code>.

**Flag 2: 822c73956184f694993bede3eb39f959**

Por última queda realizar la escalada de privilegios, para ello se buscan los binarios con permisos de ejecución de *root* y aprovechar alguna vulnerabilidad que nos permita suplantar su identidad. La búsqueda se realiza con el comando <code>find</code>, desde el directorio raíz e indicando que debe tener el bit SUID activado (-perm -4000). Este bit indica el permiso de ejecución es del propietario y al ejecutarlo se adquieren sus privilegios. Los resultados que no cumplen con estos filtros no son mostrados y enviados a una 'papelera' (2>/dev/null).

<code>find / -perm -4000 2>/dev/null</code>

<img width="483" height="308" alt="Captura de pantalla 2025-10-15 183524" src="https://github.com/user-attachments/assets/f4bbc02d-8f14-45cc-9948-6bf1a11f94b7" />

Después de intentar con varios binarios, se consigue el propósito por medio de *nmap*. Sin embargo, no ha sido posible por medio del bit SUID y se ha aprovechado otra vulnerabilidad del comando <code>sudo</code>, con la ayuda de [*GTFOBins*](https://gtfobins.github.io/gtfobins/nmap/#suid). Para obtener la escalada se utiliza el modo *interactive* sólo disponible en las versiones 2.02-5.21. Un vz comprobada la versión del *nmap* instalado se puede ejecutar los comnados siguietnes para obteenr el rol de *root*.

<code>nmap -v</code>

<code>sudo nmap -- interactive</code>

<code>! sh</code>

<img width="527" height="204" alt="Captura de pantalla 2025-10-15 183502" src="https://github.com/user-attachments/assets/4c126d6c-8558-499c-83c0-e6127cfd29f5" />

Ahora ya se puede acceder al directorio '/root' y leer el archivo con la última bandera 'key-3-of-3.txt'.

**Flag 3: 04787ddef27c3dee1ee161b21670b4e4**
