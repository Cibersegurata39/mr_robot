# mr_robot

Máquina resuelta de *TryHackMe* en la que se trabaja la enumeración y *fingerprinting*,
<div>
  <img src="https://img.shields.io/badge/-Kali-5e8ca8?style=for-the-badge&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/-Nmap-6933FF?style=for-the-badge&logo=nmap&logoColor=white" />
  <img src="https://img.shields.io/badge/-Dirsearch-005571?style=for-the-badge&logo=dirsearch&logoColor=white" />
  <img src="https://img.shields.io/badge/-Gobuster-3CBDB1?style=for-the-badge&logo=gobuster&logoColor=white" />
 <img src="https://img.shields.io/badge/-ssh-231F20?style=for-the-badge&logo=ssh&logoColor=white" />
  <img src="https://img.shields.io/badge/-php-777BB4?style=for-the-badge&logo=php&logoColor=white" />
  <img src="https://img.shields.io/badge/-Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" />
  <img src="https://img.shields.io/badge/-python-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/-unzip-000000?style=for-the-badge&logo=unzip&logoColor=white" />
</div>

## Objetivo

Explicar la realización del siguiente _Capture the flag_ perteneciente a la plataforma *TryHackMe*. Esta máquina está basada en la serie de televisión *MR Robot*, en la cual se deberán intentar encontrar hasta tres *flags* diferentes. Para ello, se deberá penetrar en la máquina por medio de una *reverse shell* y realizar una escalada de privilegios.

## Que hemos aprendido?

- Realizar *fingerprinting* y enumeración de puertos y enumeración web.
- Realizar una conxexión *SSH*.
- Crear un servidor con *python3*.
- Descomprimir archivos.
- Descifrar contraseñas.
- Buscar archivos por comando.
- Escalada de privilegios.

## Herramientas utilizadas

- *Kali Linux*.
- Enumeración: *Nmap*, *Dirsearch*, *Gobuster*.
- Penetración: *SSH*, *Bash*, *Python3*, *Unzip*, diferentes decodificadores web. 

## Steps

### Enumeración y fingerprinting

La máquina a vulnerar pertenece a la plataforma *TryHackMe*, la propia web te proporciona la IP de la máquina víctima. La conexión a esta se hace mediante una VPN que te proporciona THM y asigna una nueva IP para que tu máquina interactúe con la máquina vulnerable.

El primer paso es lanzar **Nmap** sin *ping* (-Pn) y sin resolución DNS inversa (-n) para agilizar el escaneo y ser más silencioso. Además, se quieren conocer las versiones que corren en cada puerto (-sV) y el sistema operativo (-O). No se indica, específicamente, que compruebe todos los puertos, pues en este tipo de máquina no será necesario. Por último, se lanzan los *scripts* por *default* de *nmap* (-sC) para que encuentren posibles vulnerabilidades.

<code>nmap -n -Pn -sV -O 10.10.226.58 -sC</code>

![Captura de pantalla 2025-05-15 171616](https://github.com/user-attachments/assets/e2ad0251-ddf8-4c77-a94c-cc004aea621b)

El comando devuelve 2 puertos TCPs abiertos:

- En el puerto 22 corre la versión Openssh 7.2p2, en un sistema Ubuntu, servicio SSH.
- En el puerto 80 corre el servidor Apache 2.4.18, en un sistema Ubuntu, servicio HTTP.

No ha sido capaz de detectar el sistema operativo y los *scripts* no han devuelto ninguna vulnerabilidad.

Así pues, primero nos dirigimos desde el navegador al puerto 80 de la IP dada y esta nos lleva a la página por defecto del servidor de *Apache*. Al analizar el código fuente de la página, se encuentra un comentario donde se incita a interactuar con la consola para obtener información. En otra parte del código se encuentra una lista de *pokemons*, la cual aparece en la consola a través de la variable 'original'.

![Captura de pantalla 2025-05-15 172413](https://github.com/user-attachments/assets/cfe2840a-818a-4b94-b058-eb2a56c9b651)

Lo siguiente es hacer una enumeración web con **Dirsearch** para encontrar posibles directorios y archivos.  Por la parte de directorios, únicamente aparece '/server-status' el cual no es alcanzable. *Dirsearch* es lanzado con **Python3** indicando la IP a atacar (-u) y la lista a utilizar para la enumeración (-w); en este caso la 'raft-small-directories.txt'.

![Captura de pantalla 2025-05-15 175230](https://github.com/user-attachments/assets/02e63ef5-95ec-4349-9d69-63bd7fbb1097)


### Vulnerabilidades explotadas



**Flag: PoKeMoN{Bulbasaur}**

