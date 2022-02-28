# Sesión 2 - Encriptación y Control de Acceso

## Agenda

* Encriptación y Aseguramiento del SSH 

	- Encriptación de Particiones
	- Encriptación de Directorios
	- Encriptación de Volúmenes
	- Aseguramiento de SSH (sustitución de contraseñas por archivos de claves)

* Control de Acceso a Archivos y Directorios

	- Autoria de archivos y directorios con `chown`
	- Permisos de archivos y directorios con `chmod`

* Listas de Control de Acceso

	- Introducción a las ACL (Access Control Lists)
	- Listas de acceso a usuarios y grupos
	- Listas de acceso heredables en directorios
	- Permisos específicos en la máscara del ACL
	- Directorios compartidos

* Control de Acceso con SELinux

	- Introducción a SELinux
	- Configuración de los contextos de seguridad
	- Políticas de Seguridad de SELinux

## Introducción

Al trabajar con archivos y correos, en ambientes empresariales, será común que deseemos encriptar su información para protegerla. Esta no es una práctica común para un usuario tradicional del sistema. Pero hay escenarios dónde esto deber ser una norma. Por ejemplo, dentro de una empresa es común contar con material propietario de la empresa, correos, archivos, códigos fuentes, directorios y discos duros que necesitan mantenerse cifrados ante cualquier posible robo físico de la información, por ejemplo, que sustraigan algún disco duro durante el matenimiento y lo quieran clonar o acceder a su información.

Para entender mejor la encriptación podemos partir de un mecanismo común disponible tanto en *RHEL/CentOS* y *Debian/Ubuntu* llamado **GPG** (`GNU Privacy Guard`). El cual nos permitirá administrar llaves públicas y privadas, usando diferentes algoritmos bastantes fuertes dentro de la informática.

> Generar las claves principales con `gpg --gen-key`

    [linux]$ gpg --gen-key

    gpg (GnuPG) 2.2.19; Copyright (C) 2019 Free Software Foundation, Inc.
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.

    gpg: directory '/home/ubuntu/.gnupg' created
    gpg: keybox '/home/ubuntu/.gnupg/pubring.kbx' created
    Note: Use "gpg --full-generate-key" for a full featured key generation dialog.

    GnuPG needs to construct a user ID to identify your key.

    Real name: Alan Badillo
    Email address: dragonnomada123@gmail.com
    You selected this USER-ID:
        "Alan Badillo <dragonnomada123@gmail.com>"

    Change (N)ame, (E)mail, or (O)kay/(Q)uit? O
    We need to generate a lot of random bytes. It is a good idea to perform
    some other action (type on the keyboard, move the mouse, utilize the
    disks) during the prime generation; this gives the random number
    generator a better chance to gain enough entropy.
    ...

    # Nota: Usa `gpg --full-generate-key` para mayor especificación.

> Listar las claves generadas

    [linux]$ gpg --list-keys

    gpg: checking the trustdb
    gpg: marginals needed: 3  completes needed: 1  trust model: pgp
    gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
    gpg: next trustdb check due at 2024-02-25
    /home/ubuntu/.gnupg/pubring.kbx
    -------------------------------
    pub   rsa3072 2022-02-25 [SC] [expires: 2024-02-25]
        F2640CF58167F1FBFBF0BC21577B39E191DFFE58
    uid           [ultimate] Alan Badillo <dragonnomada123@gmail.com>
    sub   rsa3072 2022-02-25 [E] [expires: 2024-02-25]

> Mostrar los archivos de `.gnupg`

    $ ls -l ~/.gnupg

    total 20
    drwx------ 2 ubuntu ubuntu 4096 Feb 25 01:58 openpgp-revocs.d
    drwx------ 2 ubuntu ubuntu 4096 Feb 25 01:58 private-keys-v1.d
    -rw-rw-r-- 1 ubuntu ubuntu 1982 Feb 25 01:58 pubring.kbx
    -rw------- 1 ubuntu ubuntu   32 Feb 25 01:58 pubring.kbx~
    -rw------- 1 ubuntu ubuntu 1280 Feb 25 01:58 trustdb.gpg

> Encriptar un archivo con `gpg -c <path>/<file>`

    [linux]$ gpg -c test.txt

    # Nota: Se creará el archivo encriptado `test.txt.gpg`

> Eliminar repetidamente un archivo con `shred -u -z <file>`

    [linux]$ shred -u -z test.txt

    # Nota: `-u` elimina el archivo y `-z` llena de zeros el registro de reemplazo.

    # Advertencia: El borrado permanente del archivo no dejará huella en los discos duros, por lo que será potencialmente imposible recuperarlo.

> Desencriptar un archivo con `gpg -d <path>/<file>.gpg`

    [linux]$ gpg -d test.txt.gpg > test.txt

    # Nota: Se creará el archivo desencriptado `test.txt`

## Encriptación y Aseguramiento del SSH

### Encriptación de Particiones

Al trabajar en entornos empresariales es común tener distintas particiones y volúmenes, incluso de varios terabytes. Y es importante supervisar y controlar el acceso a los datos dentro de estos volúmenes.

Una forma de conseguir una buena seguridad dentro de volúmenes y particiones es encriptándolos. Para restringir su acceso incluso dentro de la misma empresa. Y así proteger potenciales robos de información.

Un mecanismo común es comenzar a particionar con encriptación. Lo primero es determinar cuáles son los volúmenes y particiones de los que disponemos.

> Listar los volúmenes y particiones con `lsblk`

    [linux]# lsblk

    NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sda           8:0    0    8G  0 disk
        ├─sda1        8:1    0    1G  0 part /boot
        └─sda2        8:2    0    7G  0 part
        ├─cs-root 253:0    0  6.2G  0 lvm  /
        └─cs-swap 253:1    0  820M  0 lvm  [SWAP]
    sr0          11:0    1 1024M  0 rom

En entornos virtuales como en *Virtual Box* podremos agregar más volúmenes en la configuración de la máquina virtual cómo se muestra en la imagen. Para ello debemos previamente apagar la máquina virtual (puedes usar `shutdown -h 0`).

![Crear un nuevo disco virtual](../assets/s2.1.png)

Una vez creado disco virtual y asociado a la máquina virtual listaremos nuevamente los volúmenes y particiones usando `lsblk`.

    [linux]# lsblk

    NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sda           8:0    0    8G  0 disk
        ├─sda1        8:1    0    1G  0 part /boot
        └─sda2        8:2    0    7G  0 part
        ├─cs-root 253:0    0  6.2G  0 lvm  /
        └─cs-swap 253:1    0  820M  0 lvm  [SWAP]
    sdb           8:16   0    4G  0 disk
    sr0          11:0    1 1024M  0 rom

Ahora veremos el nuevo volúmen de `4GB` sin particiones.

Procederemos a crear dos particiones dentro del volúmen de `2GB` cada una. Usando `fdisk` podemos de forma interactiva especificar nuevas particiones.

> Crear una nueva partición con `fdisk <volume>`

    [linux]# fdisk /dev/sdb

    Bienvenido a fdisk (util-linux 2.32.1).
    Los cambios solo permanecerán en la memoria, hasta que decida escribirlos.
    Tenga cuidado antes de utilizar la orden de escritura.

    El dispositivo no contiene una tabla de particiones reconocida.
    Se ha creado una nueva etiqueta de disco DOS con el identificador de disco 0x3b1e0e4d.

    Orden (m para obtener ayuda): m

    General
        d   borra una partición
        F   lista el espacio libre no particionado
        l   lista los tipos de particiones conocidos
        n   añade una nueva partición
        p   muestra la tabla de particiones
        t   cambia el tipo de una partición
        v   verifica la tabla de particiones
        i   imprime información sobre una partición

    ...

    Guardar y Salir
        w   escribe la tabla en el disco y sale
        q   sale sin guardar los cambios

    Crea una nueva etiqueta
        g   crea una nueva tabla de particiones GPT vacía
        G   crea una nueva tabla de particiones SGI (IRIX) vacía
        o   crea una nueva tabla de particiones DOS vacía
        s   crea una nueva tabla de particiones Sun vacía


Pulsaremos `m` para obtener el menú de de ayuda. Ahí veremos varias alternativas. Las más importantes son `n` para crear una partición, `p` para ver la tabla de particiones, `g` para crear una tabla de particiones `GPT` y `w` para escribir la tabla de particiones en el disco.

    : p

    Disco /dev/sdb: 4 GiB, 4294967296 bytes, 8388608 sectores
    Unidades: sectores de 1 * 512 = 512 bytes
    Tamaño de sector (lógico/físico): 512 bytes / 512 bytes
    Tamaño de E/S (mínimo/óptimo): 512 bytes / 512 bytes
    Tipo de etiqueta de disco: dos
    Identificador del disco: 0x3b1e0e4d

    : g

    Se ha creado una nueva etiqueta de disco GPT (GUID: 78F8B12A-534E-1042-8648-E8AFFE912204).

    : p

    ...
    Tipo de etiqueta de disco: gpt
    Identificador del disco: 78F8B12A-534E-1042-8648-E8AFFE912204

    : n
    
    Número de partición (1-128, valor predeterminado 1): <<1>>
    Primer sector (2048-8388574, valor predeterminado 2048): <<2048>>
    Último sector, +sectores o +tamaño{K,M,G,T,P} (2048-8388574, valor predeterminado 8388574): <<+2G>>

    Crea una nueva partición 1 de tipo 'Linux filesystem' y de tamaño 2 GiB.

    : p
    
    ...

    Disposit.  Comienzo   Final Sectores Tamaño Tipo
    /dev/sdb1      2048 4196351  4194304     2G Sistema de ficheros de Linux

    : n

    Número de partición (2-128, valor predeterminado 2): <<>>
    Primer sector (4196352-8388574, valor predeterminado 4196352): <<>>
    Último sector, +sectores o +tamaño{K,M,G,T,P} (4196352-8388574, valor predeterminado 8388574): <<>>

    Crea una nueva partición 2 de tipo 'Linux filesystem' y de tamaño 2 GiB.

    : p

    ...

    Disposit.  Comienzo   Final Sectores Tamaño Tipo
    /dev/sdb1      2048 4196351  4194304     2G Sistema de ficheros de Linux
    /dev/sdb2   4196352 8388574  4192223     2G Sistema de ficheros de Linux

Hasta ahora hemos creado una tabla de particiones GPT con dos particiones físicas (`/dev/sdb1` y `/dev/sdb2`) de `2GB` cada una. Podemos verificar que no hay errores con `:v` y el tamaño disponible del volúmen con `:F` (es `0 B`).

Para finalizar escribiremos la tabla de particiones con `:w`, se puede imprimir la información de las particiones con `:i`.

    : i

    Número de partición (1,2, valor predeterminado 2): <<1>>

            Device: /dev/sdb1
            Start: 2048
                End: 4196351
            Sectors: 4194304
            Size: 2G
            Type: Sistema de ficheros de Linux
        Type-UUID: 0FC63DAF-8483-4772-8E79-3D69D8477DE4
            UUID: 9B03BE43-F06A-314C-8574-1CF8029D1E66

    : i

    Número de partición (1,2, valor predeterminado 2): <<2>>

            Device: /dev/sdb2
            Start: 4196352
                End: 8388574
            Sectors: 4192223
            Size: 2G
            Type: Sistema de ficheros de Linux
        Type-UUID: 0FC63DAF-8483-4772-8E79-3D69D8477DE4
            UUID: 9FF770A1-42A4-2F4E-8736-6BCA5D186BEE

    : w

    Se ha modificado la tabla de particiones.
    Llamando a ioctl() para volver a leer la tabla de particiones.
    Se están sincronizando los discos.

Si ahora volvemos a `fdisk /dev/sdb`, podremos ver que `:p` muestra la tabla de particiones que escribimos.

---

Ahora podemos encriptar nuestras particiones nuevas (`/dev/sdb1` y `/dev/sdb2`).

El formato estandarizado LUKS (*Linux Unified Key Setup*) nos permitirá cifrar las particiones y mantener la compatibilidad con la mayoría de sistemas Linux e inclusive otros. Podemos usar `cryptsetup lucksFormat <patition>` para cifrar nuestras particiones.

> Cifrar una partición con `cryptsetup lucksFormat <partition>`

    [linux]# cryptsetup -v luksFormat /dev/sdb1

    WARNING!
    ========
    Esto sobreescribirá los datos en /dev/sdb1 de forma irrevocable.

    Are you sure? (Type 'yes' in capital letters): <<YES>>
    Introduzca la frase contraseña de /dev/sdb1: <<****>>
    Verifique la frase contraseña: <<****>>
    Ranura de claves 0 creada.
    Orden ejecutada correctamente.

Después de confirmar y establecer la contraseña la partición quedará cifrada. Repetiremos para `/dev/sdb2`. La opción `-v` muestra la información detallada y `-y` podrá confirmar automáticamente para no escribir `YES`.

* **Nota:** Si la contraseña no cumple los criterios suficientes, el cifrado no se realizará. Según el tamaño de la partición, esta podría tardar algún tiempo en completarse.

Para verificar la información de encriptación de la partición podremo usar `cryptsetup lucksDump <partition>`.

> Verificar el cifrado de una partición con `cryptsetup lucksDump <partition>`

    [linux]# cryptsetup luksDump /dev/sdb1

    LUKS header information
    Version:        2
    Epoch:          3
    Metadata area:  16384 [bytes]
    Keyslots area:  16744448 [bytes]
    UUID:           cb722da4-d639-4202-b899-494a383a114c
    Label:          (no label)
    Subsystem:      (no subsystem)
    Flags:          (no flags)

    ...

Ahora podemos abrir una partición bajo algún nombre asignado mediante `cryptsetup luksOpen <partition> <name>` (nos solicitará la contraseña de cifrado). Esta será ubicada como un enlace simbólico en `/dev/mapper`.

> Abrir una partición cifrada con 

    [linux]# cryptsetup luksOpen /dev/sdb1 software

    >>> Introduzca la frase contraseña de /dev/sdb1: <<****>>

    # Visualizamos los enlaces dinámicos generados en /dev/mapper

    [linux]# ll /dev/mapper/

    total 0
    crw-------. 1 root root 10, 236 feb 24 22:41 control
    lrwxrwxrwx. 1 root root       7 feb 24 22:41 cs-root -> ../dm-0
    lrwxrwxrwx. 1 root root       7 feb 24 22:41 cs-swap -> ../dm-1
    lrwxrwxrwx. 1 root root       7 feb 24 23:34 software -> ../dm-2

En este caso se asignó `/dev/dm-2` como objetivo aunque podremos referirnos a `/dev/mapper/software`. Podemos también ver la información de nuestra partición descifrada con `dmsetup info <name>`.

> Ver la información de una partición descifrada con `dmsetup info <name>`

    [linux]# dmsetup info software

    Name:              software
    State:             ACTIVE
    Read Ahead:        8192
    Tables present:    LIVE
    Open count:        0
    Event number:      0
    Major, minor:      253, 2
    Number of targets: 1
    UUID: CRYPT-LUKS2-cb722da4d6394202b899494a383a114c-software

Ahora podemos formatear la partición y crear un sistema de archivos. Existen diversos sistemas de archivos populares en linux como XFS y EXT4. Podemos usar `mkfs.xfs` o `mkfs.ext4` para formatear la partición usando alguno de esos sistemas de archivos. Al formatear no debemos olvidar usar `/dev/mapper/<name>` y no `/dev/<partition>` ya que esta última es la cifrada.

> Formatear la partición con `mkfs.ext4 <partition>`

    [linux]# mkfs.ext4 /dev/mapper/software

    mke2fs 1.45.6 (20-Mar-2020)
    Se está creando un sistema de ficheros con 520192 bloques de 4k y 130048 nodos-i
    UUID del sistema de ficheros: 72cc7edb-6bd8-4d83-a069-1983b28465b9
    Respaldos del superbloque guardados en los bloques:
            32768, 98304, 163840, 229376, 294912

    Reservando las tablas de grupo: hecho
    Escribiendo las tablas de nodos-i: hecho
    Creando el fichero de transacciones (8192 bloques): hecho
    Escribiendo superbloques y la información contable del sistema de ficheros: hecho

---

Finalmente, para poder usar las particiones debemos montarlas en alguna carpeta del sistema. Lo más recomendable es montarlas en carpetas específicas que no interfieran con otras carpetas del sistama, por ejemplo `/<name>` con referencia a la partición `<name>` que estemos usando.

> Montar la partición con `mount <partition> <destination>`

    [linux]# mount /dev/mapper/software /software

    # Nota: Crea previamente la carpeta `/software` con `mkdir /software`.

Ahora ya estamos listos para escribir archivos y utilizar nuestra partición. Para el sistema será transparente usar la carpeta `/<name>` cómo una más de las disponibles en el sistema, pero internamente sabremos que todo estará cifrado en esa partición y será segura utilizarla.

* **Nota:** Para desmontar la partición usaremos `umount <destination>` sólo indicando el destino dónde fue montada nuestra partición. Si listamos los archivos se encontrará una carpeta llamada `lost+found` (con esto reconoceremos que es una partición de otro volúmen). También podemos usar `mount | grep <name>` para ver la información de montado de nuestra partición.

---

Montar las particiones de forma manual no es una buena práctica, ya que podríamos olvidar hacerlo o forzarnos a hacerlo tras un reinicio. Por lo que podemos montar las particiones automáticamente mediante la configuración de `/etc/crypttab` y `/etc/fstab`.

El primer paso será ubicar el UUID de la partción mediante `cryptsetup luksUUID <partition>`

> Consultar el UUID de la partición con `cryptsetup luksUUID <partition>`

    [linux]# cryptsetup luksUUID /dev/sdb1

    >>> cb722da4-d639-4202-b899-494a383a114c

Ahora debemos editar a `/etc/crypttab`. Para cada partición deberemos agregar `luks-<UUID> UUID=<UUID> none`, donde `<UUID>` es el UUID de nuestra partición. También hay que observar que `none` nos solicitará la contraseña tras cada reinicio.

> Editar a `/etc/crypttab`

    [linux]# nano /etc/crypttab

    luks-cb722da4-d639-4202-b899-494a383a114c UUID=cb722da4-d639-4202-b899-494a383a114c none

Ahora procedemos a editar a `/etc/fstab` y agregar `/dev/mapper/luks-<UUID> <destination> <filesystem> defaults 0 0`, donde `<UUID>` es el UUID de nuestra partición. En este caso debemos recordar que `<filesystem>` se refiere al sistema de archivos de la partición, para estas notas `ext4`.

> Editar a `/etc/fstab`

    [linux]# nano /etc/crypttab

    ...
    /dev/mapper/cs-swap     none                    swap    defaults        0 0
    /dev/mapper/luks-cb722da4-d639-4202-b899-494a383a114c /software ext4 defaults 0 0

* **Nota:** Podemos reiniciar mediante `shutdown -r 0`

Finalmente, tras reiniciar veremos que se nos solicita la contraseña de la partición. Esto puede evitarse usando un archivo de claves. Pero esa configuración escapa de estas notas.

![Requisito de contraseña para la partición](../assets/s2.2.png)

Una vez adentro podremos ver que nuestra partición fue montada correctamente en `<destination>` (`/software`).

![Requisito de contraseña para la partición](../assets/s2.3.png)


### Encriptación de Directorios

Para la encriptación de directorios podemos usar `eCryptfs`, la cual está disponible para *Debian/Ubuntu*, pero fue removida de *RHEL/CentOS* a partir de la versión `7`.

El cifrado de directorios se dará a través del montaje de un directorio sobre su misma ruta, usando el sistema de archivos `ecryptfs`. Usaremos `mount -t ecryptfs <path> <path>` para conseguirlo. 

Todos los archivos generados mientras la carpeta esté montada de forma cifrada podrán leerse y escribirse sin mayor problema. Sin embargo, si la carpeta se encuentra demostrada, los archivos estarán cifrados.

Esto es muy útil cuándo compartimos volúmenes/particiones o directorios en los que necesitamos mantener todos los archivos de un directorio de forma cifrada. Por ejemplo, dentro de una empresa podría ser una carpeta con los códigos fuente o los reportes financieros.

En *Debian/Ubuntu* instalaremos el paquete `ecryptfs-utils` para utilizarlo.

[UBUNTU]

    sudo apt install ecryptfs-utils

Como en *RHEL/CentOS* no disponemos de `eCryptfs`, podemos intentar instalar manualmente los paquetes usando `dnf install <package>`, pero previamente descargaremos los RPM necesarios para que los reconozca `dnf` (`Dandified YUM`).

* **Nota:** No hay garantías que funcione, pero puede intentarse.

[CENTOS]

    [rhel]# wget https://download-ib01.fedoraproject.org/pub/epel/8/Everything/x86_64/Packages/e/epel-release-8-14.el8.noarch.rpm

    [rhel]# rpm -Uvh epel-release*rpm

    [rhel]# dnf install pkcs11-helper

    [rhel]# wget http://mirror.rackspace.com/elrepo/elrepo/el8/x86_64/RPMS/elrepo-release-8.2-1.el8.elrepo.noarch.rpm

    [rhel]# rpm -Uvh elrepo-release*rpm

    [rhel]# dnf install ecryptfs-utils

Una vez asegurado `eCryptfs` en el sistema podemos montar directorios, pero antes deberemos configurar la frase de montado.

> Generar una contraseña de montado de `eCrypt`

    [ubuntu]$ ecryptfs-setup-private

    Enter your login passphrase [ubuntu]: <<~~~~>>
    Enter your mount passphrase [leave blank to generate one]: <<****>>
    Enter your mount passphrase (again): <<****>>

    ************************************************************************
    YOU SHOULD RECORD YOUR MOUNT PASSPHRASE AND STORE IT IN A SAFE LOCATION.
    ecryptfs-unwrap-passphrase ~/.ecryptfs/wrapped-passphrase
    THIS WILL BE REQUIRED IF YOU NEED TO RECOVER YOUR DATA AT A LATER TIME.
    ************************************************************************


    Done configuring.

    Testing mount/write/umount/read...
    Inserted auth tok with sig [26270991e350d295] into the user session keyring
    Inserted auth tok with sig [f9257081132b6f12] into the user session keyring
    Inserted auth tok with sig [26270991e350d295] into the user session keyring
    Inserted auth tok with sig [f9257081132b6f12] into the user session keyring
    Testing succeeded.

    Logout, and log back in to begin using your encrypted directory.

* **Nota:** Podemos usar `ecryptfs-unwrap-passphrase ~/.ecryptfs/wrapped-passphrase` para recuperar nuestra frase montado.

Ya podemos montar la carpeta mediante `mount -t ecryptfs <path> <path>`

> Montar una carpeta en forma cifrada con `eCrypt`

    [ubuntu]$ sudo mount -t ecryptfs /secret /secret

    Passphrase: <<****>>
    Select cipher:
        1) aes: blocksize = 16; min keysize = 16; max keysize = 32
        2) blowfish: blocksize = 8; min keysize = 16; max keysize = 56
        3) des3_ede: blocksize = 8; min keysize = 24; max keysize = 24
        4) twofish: blocksize = 16; min keysize = 16; max keysize = 32
        5) cast6: blocksize = 16; min keysize = 16; max keysize = 32
        6) cast5: blocksize = 8; min keysize = 5; max keysize = 16
    Selection [aes]: <<>>
    Select key bytes:
        1) 16
        2) 32
        3) 24
    Selection [16]: <<>>
    Enable plaintext passthrough (y/n) [n]: <<>>
    Enable filename encryption (y/n) [n]: <<y>>
    Filename Encryption Key (FNEK) Signature [a48d5eecd5bd6b96]:
    Attempting to mount with the following options:
        ecryptfs_unlink_sigs
        ecryptfs_fnek_sig=a48d5eecd5bd6b96
        ecryptfs_key_bytes=16
        ecryptfs_cipher=aes
        ecryptfs_sig=a48d5eecd5bd6b96
    WARNING: Based on the contents of [/root/.ecryptfs/sig-cache.txt],
    it looks like you have never mounted with this key
    before. This could mean that you have typed your
    passphrase wrong.

    Would you like to proceed with the mount (yes/no)? : <<yes>>
    Would you like to append sig [a48d5eecd5bd6b96] to
    [/root/.ecryptfs/sig-cache.txt]
    in order to avoid this warning in the future (yes/no)? : <<yes>>
    Successfully appended new sig to user sig cache file
    Mounted eCryptfs

    # Inspeccionar la ruta montada

    [ubuntu]$ mount | grep secret

    /secret on /secret type ecryptfs (rw,relatime,ecryptfs_fnek_sig=a48d5eecd5bd6b96,ecryptfs_sig=a48d5eecd5bd6b96,ecryptfs_cipher=aes,ecryptfs_key_bytes=16,ecryptfs_unlink_sigs)

* **ADVERTENCIA:** No debes montar la carpeta mientras estás dentro de ella.

Ahora todos los archivos que se creen dentro de la carpeta mientras este montada quedarán cifrados.

> Desmontar la carpeta

    [ubuntu]$ sudo umount /secret

Una vez desmontada los archivos generados quedarán encriptados y su lectura será imposible.

### Encriptación de Volúmenes

`VeraCrypt` es una gran alternativa a las particiones `LUKS` y al cifrado de directorios `eCrypt`. Esta nos permitirá crear volúmenes cifrados compatibles con otros sistemas operativos. Su funcionamiento es bastante sencillo y cómodo y hay alternativas GUI para su uso.

La consola de `veracrypt` nos permitirá crear los volúmenes e interacturar con ellos. Podemos instalarlo desde [https://www.veracrypt.fr/en/Downloads.html](https://www.veracrypt.fr/en/Downloads.html).

> Instalar VeraCrypt

    # Descargar el paquete de VeraCrypt

    [linux]$ wget https://launchpad.net/veracrypt/trunk/1.25.9/+download/veracrypt-1.25.9-setup.tar.bz2

    # Descomprimir el paquete

    [linux]$ tar xvf veracrypt-1.25.9-setup.tar.bz2

    # Ejecutar el instalador de consola x64

    [linux]$ ./veracrypt-1.25.9-setup-console-x64

Ahora ya podemos crear volúmenes y usarlos.

> Crear un nuevo volumen con `veracrypt -c`

    [linux]$ veracrypt -c

    Volume type:
        1) Normal
        2) Hidden
    Select [1]:

    Enter volume path: <</volumes/data.hc>>

    Enter volume size (sizeK/size[M]/sizeG.sizeT/max): <<1G>>

    Encryption Algorithm:
        1) AES
        2) Serpent
        3) Twofish
        4) Camellia
        5) Kuznyechik
        6) AES(Twofish)
        7) AES(Twofish(Serpent))
        8) Camellia(Kuznyechik)
        9) Camellia(Serpent)
        10) Kuznyechik(AES)
        11) Kuznyechik(Serpent(Camellia))
        12) Kuznyechik(Twofish)
        13) Serpent(AES)
        14) Serpent(Twofish(AES))
        15) Twofish(Serpent)
    Select [1]:

    Hash algorithm:
        1) SHA-512
        2) Whirlpool
        3) SHA-256
        4) Streebog
    Select [1]:

    Filesystem:
        1) None
        2) FAT
        3) Linux Ext2
        4) Linux Ext3
        5) Linux Ext4
        6) NTFS
        7) exFAT
        8) Btrfs
    Select [2]: <<5>>

    Enter password: <<****>>
    Re-enter password: <<****>>

    Enter PIM: <<8891>>

    Enter keyfile path [none]:

    Please type at least 320 randomly chosen characters and then press Enter:
    <<...>>

    Done: 100.000%  Speed:  18 MiB/s  Left: 0 s

    The VeraCrypt volume has been successfully created.

Después de crear el volumen seremos capaces de utilizarlo fácilmente mediante `veracrypt <volume> <destination>`, donde `<volume>` es la ruta a nuestro volumen y `<destination>` será el lugar donde se monte.

> Montar un volumen con `veracrypt <volume> <destination>`

    [linux]$ veracrypt /volumes/data.hc /volumes/data

Ahora ya podemos usar el volumen a través de `/volumes/data`. En modo administrador podremos cambiar los permisos para poder utilizarlo libremente.

> Desmontar un volumen con `veracrypt -d <volume>`

    [linux]$ -d veracrypt /volumes/data.hc

### Aseguramiento de SSH (sustitución de contraseñas por archivos de claves)

En los sistemas empresariales es muy común sustituir el uso de contraseñas por llaves públicas, con el fin de asegurar sus sistemas en un medio más físico. La desventaja de usar contraseñas, es que son más fáciles de compartir entre usuarios, dando como resultado un acceso más inmediato y vulnerable. Piense en una empresa con miles de programadores, y la posibilidad de que alguno comparta su contraseña con algún colega o extraño. Por otro lado, poseer claves públicas como medio de acceso al sistema, es una buena práctica laboral, ya que estas pueden ser configuradas en equipos dedicados a los empleados y no ser compartidas fácilmente. Si un empleado quisiera compartir la clave, tendría que enviarla por algún medio, como correo o algún dispositivo de almacenamiento, dejando rastro físico de dicho evento.

Más allá de eso, una gran ventaja de utilizar llaves públicas como método de acceso, es poder automatizar el acceso a los sistemas. Imagina escenarios donde periodicamente se renuevan las credenciales de acceso al sistema y los empleados descargan las nuevas claves o estas se descargan automáticamente en sus equipos. Estos mecanismos generan un flujo de trabajo llamado `Passwordless` (libre de contraseñas). Dónde los usuarios no tienen que memorizar contraseñas y poner en riesgo al sistema por la construcción de contraseñas débiles que pueden ser inferidas por atacantes.

Las llaves públicas son generadas en pareja de forma asimétrica por algoritmos de encriptación. Su contra parte es una llave privada que nunca es compartida y la reserva el sistema, para validar a la llave pública. Es decir, en los algoritmos de encriptación, podemos encontrar algoritmos simétricos (de una sola clave) o algoritmos asimétricos (de parejas de claves). Los algoritmos de encriptación asimétricos nos permitirán entonces administrar parejas de llaves, de las cuáles una podrá ser compartidad de forma segura y la otra quedará protegida en el sistema. Así la llave compartida o la *llave pública* (o clave pública), puede ser entregada como medio de acceso a los usuarios y tendrán una equivalencia a una contraseña. Sólo que en este caso, será una cadena larga de caracteres (o bytes), los cuales harán difícil que los usuarios siquiera se tomen el tiempo de memorizarlas o ver su contenido, pero mejor aún, se basan en algoritmos de cifrado robustos que tomarían bastante tiempo en ser decifrado, incluso milenios (con la computación moderna y actual).

---

Vamos a comenzar por generar una pareja de claves mediante `ssh-keygen -t <alg>`, donde `<alg>` será el algoritmo de encriptación deseado (generalmente `rsa`). Este generador nos preguntará por el nombre de nuestras claves (por defecto `~/.ssh/id_rsa`). La clave privada será protegida por una contraseña que será preguntada.

> Generar una pareja de claves con `ssh-keygen -t <alg>`

    [linux]$ ssh-keygen -t rsa

    Generating public/private rsa key pair.
    Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa): <<>>
    Enter passphrase (empty for no passphrase): <<****>>
    Enter same passphrase again: <<****>>
    Your identification has been saved in /home/ubuntu/.ssh/id_rsa
    Your public key has been saved in /home/ubuntu/.ssh/id_rsa.pub
    The key fingerprint is:
    SHA256:peRWcP310j2uLDGGbEGYNNcACBatigbcX0/CybK97yA ubuntu@ubuntu-server
    The key's randomart image is:
    +---[RSA 3072]----+
    |   ++ o+=o+.     |
    |  .  o oo+ ..   .|
    |. . . o + o  . oo|
    |.. o . O *    o.+|
    |o . . = S o   ...|
    |.o   o o = +   . |
    |.    E .o . + .  |
    |      ...  . o   |
    |        oo  .    |
    +----[SHA256]-----+

Esto nos genera dos claves, la clave privada llamada `id_rsa` (o el nombre especificado) y la clave pública `id_rsa.pub` (o el nombre especificado más la extensión `.pub`). Si inspeccionamos las claves, estas deberían contener un conjunto de caracteres y algunas cabeceras indicativas.

> Inspeccionar las claves

    [linux]$ cat ~/.ssh/id_rsa

    -----BEGIN OPENSSH PRIVATE KEY-----
    b3BlbnNzaC1rZXktdjEAAAAACmFlczI1Ni1jdHIAAAAGYmNyeXB0AAAAGAAAABBpDZo+26
    ... (bastantes más caracteres)
    qmeK89M6y0NR0y/S5pLtOT0YD3yUlkPXWaQELXZm17kYMxuksjWN8W0gqNOUPZPWvgyhm8
    XNiOXxDWy4QmGW+g3+jzMfRE5ec=
    -----END OPENSSH PRIVATE KEY-----

    [linux]$ cat ~/.ssh/id_rsa.pub

    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQ...= ubuntu@ubuntu-server

---

Ahora ya podemos crear una comunicación segura entre máquinas. Veamos una tabla que resumirá lo necesario para trabajar.

Máquina | Llave | Comentarios
--- | --- | ---
Cliente | Privada | El cliente genera una pareja de claves y posee la llave privada a su resguardo como contraseña de autenticación
Servidor | Pública | El servidor agrega la clave pública del cliente en su registro de claves autorizada `.ssh/authorized_keys`

Suponiendo que nuestro servidor *Debian/Ubuntu* o *RHEL/CentOS* es la máquina a la que queramos ingresar mediante las llaves públicas para no usar contraseñas desde nuestros equipos remotos (nuestra máquina local o de oficina). Entonces tendremos que considerar una serie de pasos.

1. El usuario con el que deseamos ingresar debe estar registrado en el servidor.
2. El usuario debería tener una carpeta HOME de usuario con su propia carpeta `.ssh` y el archivo `.ssh/authorized_keys` que contenga todas las llaves públicas permitidas para iniciar sesión con este usuario.
3. El cliente remoto que se conectará con nuestro usuario del servidor deberá poseer el archivo de la llave privada que generó la llave pública agregada en nuestro servidor.
4. Debemos habilitar la posibilidad de aceptar inicios de sesión con llaves públicas desde `/etc/ssh/sshd_config`.
5. Opcionalmente podemos desactivar los inicios de sesión con contraseña.

Si seguimos estos pasos, podremos configurar usuarios del servidor y proveerle a nuestros clientes la posibilidad de enviarnos sus llaves públicas para darles acceso con los usuarios registrados en el servidor.

> 1. Crear un usuario nuevo

    [server]$ sudo useradd -m -d /home/roboto -s /bin/bash roboto

    [server]$ id roboto
    
    >>> uid=1003(roboto) gid=1003(roboto) groups=1003(roboto)

> 2. Generar la carpeta `.ssh` y el archivo `authorized_keys` del usuario

    [server]$ sudo mkdir -p ~roboto/.ssh

    [server]$ sudo touch ~roboto/.ssh/authorized_keys

> 3. Generar pares de llaves para cada cliente

    [local]> ssh-keygen -t rsa -C "client-1 as roboto"

    Generating public/private rsa key pair.
    Enter file in which to save the key (C:\Users\lanz/.ssh/id_rsa): <<client-1>>
    Enter passphrase (empty for no passphrase): <<****>>
    Enter same passphrase again: <<****>>
    Your identification has been saved in client-1.
    Your public key has been saved in client-1.pub.
    The key fingerprint is:
    SHA256:UF8q8VvDV+0mYnauMkgm0WNu7c/7SntkZysB9ZkceGY client-1 as roboto
    The key's randomart image is:
    +---[RSA 3072]----+
    |        o   . . o|
    |       . + + o E.|
    |      ... + = B.+|
    |      ..+. ++oo=o|
    |       +So.o.+ o |
    |      . = .  .+ o|
    |       = o  .+.o.|
    |        . +o.o.. |
    |           +*=o  |
    +----[SHA256]-----+

    [local]> more client-1.pub

    >>> ssh-rsa AAAAB3.../U2Jefk= client-1 as roboto

    # NOTA: Estas se pueden generar en una máquina local, no necesariamente sobre el servidor.

    # Agregamos la clave a `~roboto/.ssh/authorized_keys`

    [server]$ sudo nano ~roboto/.ssh/authorized_keys

    --- 
    ssh-rsa AAAAB3.../U2Jefk= client-1 as roboto
    ---

    # Alternativamente podemos copiar la identidad desde el cliente

    [local]> ssh-copy-id -i roboto.pub roboto@<server>

> 4. Activamos el inicio de sesión con llaves públicas

    [server]$ sudo nano /etc/ssh/sshd_config

    ---
    #PubkeyAuthentication no
    PubkeyAuthentication yes
    ...
    #AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2
    AuthorizedKeysFile      .ssh/authorized_keys
    ---

> 5. [Opcional] Deshabilitamos la autenticación por contraseña

    [server]$ sudo nano /etc/ssh/sshd_config

    ---
    ...
    PasswordAuthentication no
    ---

Finalmente, los clientes podrán conectarse usando alguna interfaz o directamente desde sus terminales.

> Conectar desde el cliente al servidor mediante la terminal

    [local]> ssh -i client-1 roboto@<server>

    Enter passphrase for key 'client-1': <<****>>

    ...

    [roboto@<server>]$ whoami

    >>> roboto

En la siguiente imagen podemos ver el resultado desde una terminal en Windows 10.

![Client 1 conectado al server](../assets/s2.4.png)

* **COMENTARIO:** Como último comentario, el cliente puede usar `ssh-copy-id -i <keyfile> <user>@<server>` para agregar automáticamente la llave pública en el servidor, sin la necesidad de registrarla manualmente, sin embargo, el cliente debería saber la contraseña del usuario (`<user>`), por lo cuál podría perder sentido práctico.

[REFERENCIAS]

* [https://man7.org/linux/man-pages/man5/crypttab.5.html](https://man7.org/linux/man-pages/man5/crypttab.5.html)
* [https://devconnected.com/how-to-create-disk-partitions-on-linux/](https://devconnected.com/how-to-create-disk-partitions-on-linux/)
* [https://www.veracrypt.fr/en/Command%20Line%20Usage.html](https://www.veracrypt.fr/en/Command%20Line%20Usage.html)
* [https://kb.iu.edu/d/aews](https://kb.iu.edu/d/aews)
* [https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server-es](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server-es)

## Control de Acceso a Archivos y Directorios

### Autoria de archivos y directorios con `chown`

El comando `chown` nos permite definir la autoría de carpetas y archivos. Podemos establecer a quién le pertenecerán los recursos y que privilegios tendrán los demás usuarios.

Existen tres niveles de acceso a los recursos (carpetas y archivos) que se listan en la siguiente tabla.

Nivel | Descripción
--- | ---
*Usuario* |  Determina el acceso a nivel de un usuario específico.
*Grupo* | Determina el acceso a nivel de usuarios del mismo grupo.
*Otros* | Determina el acceso a nivel de otros usuarios que no están en el grupo.

* **IMPORTANTE:** A estos tres niveles de acceso se les llama **`UGO`** (`ugo`).

Al generar o reasignar la propiedad de pertenencia de los recursos (autoría / nivel de acceso), debemos considerar hacerlo en esos tres niveles como buena práctica (al menos que sólo se desee dar acceso a nivel grupo y otros).

> Asignar la propiedad de un recurso a un usuario

    SINTAXIS: chown <user> <resource>

> Asignar la propiedad a un grupo

    SINTAXIS: chown :<group> <resource>

> Asignar la propiedad a un usuario y un grupo

    SINTAXIS: chown <user>:<group> <resource>

Es importante aclarar que el grupo (`<group>`) no necesariamente es el mismo que el del usuario. Por ejemplo, el usuario puede ser `ana` cuyo grupo es `rh`. Pero ella crea un reporte que debe estar disponible para el equipo de soporte, cuyo grupo es `soporte`. Entonces, podríamos asignar la autoría del recurso como `chown ana:soporte reporte.xlsx`, cuyo propietario es el usuario `ana` y el grupo `soporte`.

### Permisos de archivos y directorios con `chmod`

Determinar la autoría de los recursos es esencial para administrar correctamente un sistema, sin problemas comunes como que algunos usuarios alteren o accedan a otros recursos protegidos por accidente o intensionadamente.

Existen varias formas de establecer los permisos sobre los recursos, pero entendamos primero los tiempos de permisos que se pueden otorgar, resumidos en la siguiente tabla.

Permiso | Símbolo | Modo | Descripción
--- | --- | --- | ---
*read* | `r` | `4` | Permite leer el recurso
*write* | `w` | `2` | Permite escribir el recurso
*execute* | `x` | `1` | Permite ejecutar/búscar el recurso

Los permisos pueden ser combinados sobre un recurso, por ejemplo, `rw` significa el permiso de lectura y escritura, cuya suma de *modos* es `6` (`4 + 2`). En la siguiente tabla se muestran las combinaciones posibles.

Modo | Permiso | Descripción
--- | --- | ---
`0` | `-` | Ningún permiso
`1` | `x` | Ejecución
`2` | `w` | Escritura
`3` | `wx` | Escritura + Ejecución
`4` | `r` | Lectura
`5` | `rx` | Lectura + Ejecución
`6` | `rw` | Lectura + Escritura
`7` | `rwx` | Lecutra + Escritura + Ejecución (Todos)

Es común ver permisos en símbolos o números, por ejemplo, `rx` o `5`. Generalmente cuándo listamos los detalles de una carpeta usando `ls -l` podremos ver en la primer columna los permisos con el formato `--- --- ---` que representa a tres ternas de permisos considerados los permisos **`ugo`**.

---

Al asignar permisos a un recurso tomaremos en cuenta los tres niveles de acceso de la propiedad del recurso (`ugo`) establecidos por `chown`.

Para cada nivel determinaremos los permisos del recurso para cada nivel. En la siguiente tabla se muestran los tres niveles de acceso, con algunos modos de ejemplo.

Nivel | Permiso | Descripción
--- | --- | ---
**u** | `rwx` | El usuario propietario del recurso tiene permiso de lectura escritura y ejecución
**g** | `r` | El grupo propietario del recurso sólo tiene permiso de lectura
**o** | `-` | Otros usuarios no tienen permisos de acceso
**u** | `r` | El usuario propietario sólo es capaz de leer el recurso
**g** | `w` | El grupo propietario sólo es capaz de escribir el recurso
**o** | `x` | Otros usuarios son capaces de ejecutar el recurso
**u** | `5` | El usuario propietario es capaz de leer y ejecutar el recurso
**g** | `3` | El grupo propietario es capaz de escribir y ejecutar el recurso
**o** | `0` | Otros usuarios no tienen ningún permiso sobre el recurso

---

Existen dos modos principales de establecer los permisos a un recurso mediante `chmod`, el primero se considera simbólico y el segundo numérico.

> Establecer permisos en modo simbólico

    # Modifica los permisos sólo para el usuario propietario

    SINTAXIS: chmod u=<mode> <resource>

    # Modifica los permisos para el usuario propietario y el grupo propietario

    SINTAXIS: chmod u=<mode>,g=<mode> <resource>

    # Modifica los permisos para los tres niveles de acceso `ugo`

    SINTAXIS: chmod u=<mode>,g=<mode>,o=<mode> <resource>

    # Establece los permisos para el usuario propietario y deja sin permisos al grupo y los otros

    SINTAXIS: chmod u=<mode>,g=,o= <resource>
    
    # Agregar un permiso singular (no altera los otros)

    SINTAXIS: chmod <symbol>+<mode> <resource>

    # Quitar un permiso singular (no altera los otros)

    SINTAXIS: chmod <symbol>-<mode> <resource>

    # Ejemplo: Agrega lectura al recurso `<resource>` para el grupo propietarios
    #   chmod g+w <resorce>

Veamos un ejemplo en el que el recurso sea un reporte generado por el usuario Ana (`ana`) de Recursos Humanos (grupo `rh`). Y queremos establecer que Ana tenga acceso de lectura y escritura, el equipo de Soporte (grupo `soporte`) tenga los permisos de lectura y ejecución y los demás usuarios no tengan permisos, entonces podemos establecer los siguientes permisos sobre el reporte.

> EJEMPLO: Permisos del reporte

    [linux]# chmod u=rw,g=rx,o= reporte.xlsx

    # Tabla de permisos
    #   r w x
    # u • • -
    # g • - •
    # o - - -

Otra forma de asignar permisos es mediante el modo numérico, este se basará en la suma de los modos (`r=4, w=2, x=1`). A diferencia del modo simbólico, aquí nos veremos forzados a establecer el permiso para cada nivel de acceso (`ugo`).

> Establecer permisos en modo numérico

    # Modifica los permisos sólo para el usuario propietario

    SINTAXIS: chmod <mode> <resource>

    # Donde <mode> es la suma de los permisos
    # r - 4
    # w - 2
    # x - 1

    # Ejemplo: chmod 650 /data
    #   * Indica que el usuario propietario de /data puede leer y escribir (4 + 2 = 6)
    #   * Indica que el grupo propietario de /data puede leer y ejecutar (4 + 1 = 5)
    #   * Indica que otros usuarios no tienen permisos (0)

Veamos el mismo ejemplo del reporte como quedaría.

> EJEMPLO: Permisos del reporte

    [linux]# chmod u=rw,g=rx,o= reporte.xlsx

    # Tabla de permisos
    #   u g o 
    # r 4 4 0
    # w 2 0 0
    # x 0 1 0
    #   - - -
    # = 6 5 0

## Listas de Control de Acceso

### Introducción a las ACL (Access Control Lists)

Las Listas de Control de Acceso (Access Control Lists / `ACL`) nos permiten establecer permisos más específicos que `chmod` para nuestros recursos, para otorgar accesos a usuarios o grupos específicos.

Esto nos da la granularidad necesaria para trabajar proteger nuestros recursos de una forma bastante sencilla y sin complicarnos mucho, evitando unir a los usuarios a grupos específicos.

Lo primero será instalar el paquete `acl` que provee a `getfacl` y `setfacl`.

> Instalar `acl`

    [linux]# apt install acl

Con `getfacl` (*get file access control lists*) podemos obtener información más detallada del recurso, como el usuario propietario, el grupo propietario, los permisos `ugo` (*user, group, others*) y la lista ACL definida para el archivo si posee una.

> Usar `getfacl` para consultar el ACL de un recurso

    SINTAXIS: getfacl [options] <resource>

    EJEMPLO:

    # Creamos un archivo nuevo llamado `demo.txt`

    [linux]$ touch demo.txt

    # Consultamos sus permisos mediante `ls -l`

    [linux]$ ls -l demo.txt
    
    >>> -rw-rw-r-- 1 ubuntu ubuntu 0 Feb 26 19:12 demo.txt

    # Observamos: 
    #   * El usuario propietario puede leer y escribir
    #   * El grupo propietario puede leer y escribir
    #   * Otros pueden leer

    # Consultamos sus permisos mediante `getfacl`

    [linux]$ getfacl demo.txt

    # file: demo.txt
    # owner: ubuntu
    # group: ubuntu
    user::rw-
    group::rw-
    other::r--

    # Observamos: 
    #   * El usuario propietario es `ubuntu`
    #   * El grupo propietario es `ubuntu`
    #   * El usuario propietario puede leer y escribir
    #   * El grupo propietario puede leer y escribir
    #   * Otros pueden leer

Cuando un usuario crea recursos, el se vuelve el propietario, junto a su grupo y se otorgan los permisos por defecto para leer y escribir, tanto para el usuario, como para el grupo.

Antes de usar las listas de control de acceso (*ACL*), vamos a quitar los permisos para el grupo y para otros, de esta manera iremos granularmente otorgando los permisos necesarios, según lo planeemos. Entonces, dejaremos sólo los permisos de lectura y escritura para el usuario propietario y los demás ninguno (modo `600` / `u=rw,g=,o=`).

> Dejar unicamente los permisos para el propietario

    [linux]$ chmod 600 demo.txt

    [linux]$ ls -l demo.txt

    >>> -rw------- 1 ubuntu ubuntu 0 Feb 26 19:12 demo.txt

    [linux]$ getfacl demo.txt

    # file: demo.txt
    # owner: ubuntu
    # group: ubuntu
    user::rw-
    group::---
    other::---

### Listas de acceso a usuarios y grupos

Las Listas de Control de Acceso (*ACL*) nos permiten otorgar permisos específicos mediante `setfacl`. Por ejemplo, en un escenario sobre nuestro archivo `demo.txt` queremos darle permisos de lectura al usuario `roboto` y permisos de lectura y ejecución al grupo `soporte`. Esto no se lograría usando `chmod`, ya que el usuario propietario es `ubuntu` y no queremos que esto cambie, tampoco queremos cambiar al grupo propietario (igulamente `ubuntu`).

> Otorgar permisos granulares a usuarios y grupos con `setfacl`

    SINTAXIS: setfacl [options] [ -m | -x <spec> ] <resource>

    # Donde:
    #   * -m significa modificar el permiso (también `-M`)
    #   * -x significa borrar el permiso (también `-X`)
    #   * <spec> es la especificación del permiso
    #       - u:<user>:<mode> 
    #       - g:<group>:<mode> 
    #       - o:<mode> 
    #       - m:<mode> 
    #       -- u | user
    #       -- g | group
    #       -- o | other
    #       -- m | mask
    #       -- <mode> puede ser simbólico (`rwx`), numérico o sin permisos (`-`)

    EJEMPLO:

    [linux]$ setfacl -m u:roboto:r demo.txt
    
    [linux]$ getfacl demo.txt
    
    # file: demo.txt
    # owner: ubuntu
    # group: ubuntu
    user::rw-
    user:roboto:r--
    group::---
    mask::r--
    other::---

    # Observa que sólo el propietario puede leer y escribir, pero el usuario `roboto` puede leer a `demo.txt`.

    [linux]$ setfacl -m g:soporte:rx demo.txt
    
    [linux]$ getfacl demo.txt

    # file: demo.txt
    # owner: ubuntu
    # group: ubuntu
    user::rw-
    user:roboto:r--
    group::---
    group:soporte:r-x
    mask::r-x
    other::---

    # Observa que ahora le otorgamos permisos de lectura y escritura al grupo de `soporte`.

### Listas de acceso heredables en directorios

Sobre las carpetas podemos ajustar permisos ACL por defecto que se apliquen a todos los archivos que se creen en el directorio. Por ejemplo, un escenario es cuándo queremos que todos los archivos que descargue el usuario `ubuntu`, estén disponibles para su lectura con el usuario `roboto` y para su ejecución para el grupo `pruebas`. Bastará con poner una `d:` antes del permiso en `setfacl` (por ejemplo, `setfacl -m d:u:roboto:r <directory>`).

> Crear permisos heredables en directorios

    SINTAXIS: setfacl -m d:<spec> <folder>

    EJEMPLO:

    [linux]$ mkdir downloads

    [linux]$ setfacl -m d:u:roboto:r downloads/

    [linux]$ setfacl -m d:g:pruebas:x downloads/

    [linux]$ getfacl downloads/

    # file: downloads/
    # owner: ubuntu
    # group: ubuntu
    user::rwx
    group::rwx
    other::r-x
    default:user::rwx
    default:user:roboto:r--
    default:group::rwx
    default:group:pruebas:--x
    default:mask::rwx
    default:other::r-x

    # Si queremos proteger los archivos de `downloads` de otros usuarios, incluso dentro del mismo grupo podemos cambiar los permisos por defecto heredados.

    [linux]$ setfacl -m d:u::rw downloads/

    [linux]$ setfacl -m d:g::- downloads/

    [linux]$ setfacl -m d:o::- downloads/

    [linux]$ getfacl downloads/

    # file: downloads/
    # owner: ubuntu
    # group: ubuntu
    user::rwx
    group::rwx
    other::r-x
    default:user::rw-
    default:user:roboto:r--
    default:group::---
    default:group:pruebas:--x
    default:mask::r-x
    default:other::---

    # Observa que ahora los archivos que se creen sólo podrán ser leídos y escritos por el usuario propietario, leídos por el usuario `roboto` y ejecutados por los usuarios del grupo `pruebas`

    [linux]$ cd downloads/

    [linux]$ wget linux.org

    [linux]$ ls -l

    >>> -rw-r-----+ 1 ubuntu ubuntu 93592 Feb 26 20:32 index.html

    [linux]$ getfacl index.html

    # file: index.html
    # owner: ubuntu
    # group: ubuntu
    user::rw-
    user:roboto:r--
    group::---
    group:pruebas:--x               #effective:---
    mask::r--
    other::---

    # Ahora podemos el contenido con el usuario `roboto`

    [roboto@linux]$ head -7 index.html | tail -1

    >>> <title>Linux.org</title>

    # Pero cualquier otro usuario no podrá

    [demo@linux]$ head -7 index.html | tail -1

    >>> head: cannot open 'index.html' for reading: Permission denied

Ahora ya podemos crear directorios con listas de permisos heredables proteger los archivos que sean dispuestos ahí. Si algún archivo fuera copiado, podríamos actualizar sus permisos con `getfacl -d . | setfacl --set-file=- <file>` (que copiará los permisos por defecto de la carpeta actual `.` hacia el archivo).

* **Nota:** Si queremos resetear los permisos por defecto de una carpeta a todos sus archivos podemos hacer `getfacl -d <folder> | setfacl -R --set-file=- <folder>`.

> Resetear los permisos por defecto de una carpeta a todos sus archivos

    SINTAXIS: getfacl -d <folder> | setfacl -R --set-file=- <folder>

### Permisos específicos en la máscara del ACL

La máscara de permisos es un filtro final, que es calculada para el recurso y podemos observar con `getfacl`.

Por ejemplo, para un archivo tenemos los permisos de la siguiente tabla, se generará la máscara descrita.

Permiso | Descripción
--- | ---
`u:roboto:rw` | Usuario `roboto` puede leer y escribir
`u:demo:w` | Usuario `demo` puede escribir
`u:ubuntu:wx` | Usuario `ubuntu` puede escribir y ejecutar
`g:sporte:x` | Grupo `soporte` puede ejecutar

La máscara resultante será `rwx`.

Ahora pensemos en un escenario donde queramos desactivar la escritura del archivo, sin tener que remover las entradas para los usuarios a los que se les otorgó permisos de escritura. Entonces, en este panorama podemos modificar la máscara, para deshabilitar la escritura, sin tener que eliminar las especificaciones.

> Modificar la máscara para desactivar la escritura

    SINTAXIS: setfacl -m m::rx <resource>

    ALTERNATIVA: setfacl -m mask::rx <resource>

De esta manera ahora los usuarios no podrán escribir aunque se les haya dado el permiso anteriormente.

### Directorios compartidos

En las carpetas compartidas podemos asignar un usuario y grupo diseñados para darle propiedad a la carpeta con el usuario `nobody` y el grupo que deseemo (`chown nobody:<group> <folder>`). Esto provocará que ningún usuario como tal sea el propietario, siendo útil para especificar que el propietario de la carpeta es el grupo.

El uso del bit *SGID* (de los permisos especiales) nos permitirá hacer que todos los archivos creados o dispuestos en la carpeta tengan como grupo el mismo que la carpeta. Es decir, los archivos contenidos tendrán el grupo de la carpeta y no el grupo de los usuarios. Esto será útil en escenarios diseñados para dar permisos por grupos sobre la carpeta compartida. Para activar el *SGID bit* usaremos el modo `2000 + <mode>`, por ejemplo, `chmod 2770 <folder>` permitirá `rwx` para propietarios y el grupo.

El uso del bit *Sticky* (de los permisos especiales) nos impedirá borrar los archivos de 

> Activación de permisos especiales para una carpeta compartida

    SINTAXIS: chmod <special><mode> <resource>

    # Donde <special> puede ser:
    # 1 - Sticky bit (impide borrar)
    # 2 - SGID bit (mismo grupo)
    # 3 - Sticky + SGID
    # 4 - SUID bit (mismo usuario, no recomendado)
    # 5 - Sticky + SUID
    # 6 - SGID + SUID
    # 7 - Sticky + SGID + SUID

    # Donde <mode> es el modo numérico

    # Podemos activar o desactivar manualmente los bits especiales

    Sticky: chmod +t / -t <resource>
    SGID:   chmod g+s / g-s <resource>
    SUID:   chmod u+s / u-s <resource>

* **Nota:** Si algún archivo perdiera la autoría del grupo, podemos recursivamente volver a establecer el grupo para todos los archivos con `chgrp -R marketing /marketing`.

Veamos un ejemplo de una carpeta compartida para miembros del grupo de Marketing.

> Compartir una carpeta para los usuarios del grupo `marketing`

    [linux]# mkdir /marketing
    
    [linux]# chown nobody:marketing /marketing
    
    # Impide borrar archivos de otros

    [linux]# chmod +t /marketing

    # Asigna el grupo `marketing` a nuevos archivos

    [linux]# chmod g+s /marketing

    # Permite `rwx` a propietarios y grupo `marketing`

    [linux]# chmod 770 /marketing

    # Alternativo: chmod 3770 /marketing

Con esto nuestra carpeta `/marketing` será compartida de forma segura entre todos los miembros del grupo `marketing`. Cada usuario podrá crear y borrar sus propios archivos, pero no los archivos de otros. A otros usuarios fuera del grupo `marketing` no se les dará acceso a la carpeta.

Finalmente, podemos también usar los permisos *ACL* para proteger archivos dentro de la carpeta compartida. Por ejemplo, vamos a suponer que el usuario `pedro` que es Líder de Marketing tiene la nómina de los empleados (`nomina.txt`) en la carpeta compartida y sólo la quiere compartir con los usuarios `lisa` y `clara` que llevan la contabilidad y recursos humanos respectivamente.

> Restringir archivos con ACL en carpetas compartidas

    # Pedro (Líder de Marketing)

    [pedro@linux]$ chmod 700 nomina.txt

    [pedro@linux]$ setfacl -m u:lisa:rwx nomina.txt

    [pedro@linux]$ setfacl -m u:clara:rx nomina.txt

    # Lisa (Contabilidad)

    [lisa@linux]$ nano nomina.txt

    >>> (Lectura, Escritura y Ejecución)

    # Clara (Recursos humanos)

    [clara@linux]$ nano nomina.txt

    >>> (Sólo Lectura y Ejecución)

    # Junior (Becario de Marketing)

    [junior@linux]$ nano nomina.txt

    >>> Permission denied

[REFERENCIAS]

* [https://linux.die.net/man/1/chown](https://linux.die.net/man/1/chown)
* [https://linux.die.net/man/1/chmod](https://linux.die.net/man/1/chmod)
* [https://www.redhat.com/sysadmin/suid-sgid-sticky-bit](https://www.redhat.com/sysadmin/suid-sgid-sticky-bit)
* [https://www.ochobitshacenunbyte.com/2019/06/17/permisos-especiales-en-linux-sticky-bit-suid-y-sgid/](https://www.ochobitshacenunbyte.com/2019/06/17/permisos-especiales-en-linux-sticky-bit-suid-y-sgid/)

## Control de Acceso con SELinux

### Introducción a SELinux

**SELinux** es un proyecto open source gratuito desarrollado por la NSA (*National Security Agency / Agencia Nacional de Seguridad*) de los Estados Unidos. Esta viene activada en *RHEL/CentOS*, pero se puede instalar en cualquier otra distribución.

Hay tres usos comunes que se le puede dar a *SELinux*.

* Se puede usar para ayudar a prevenir que intrusos exploten/dañen el sistema.
* Se puede utilizar para garantizar que solo los usuarios con la autorización de seguridad adecuada puedan
acceder a archivos que están etiquetados con una clasificación de seguridad.
* Además de MAC (*Mandatory Access Control*), provee un tipo basado en roles de acceso (`RBAC` / *Role Based Access Controls*).

En estas notas sólo usaremos a *SELinux* para ayudar a prevenir que intrusos exploten el sistema, que es la forma más común de usarlo. Los otros puntos requerirían un estudio completo.

> Instalación de SELinux

    [ubuntu]# apt install selinux-utils

> Verficar si SELinux está deshabilitado

    [linux]$ selinuxenabled; echo $?

    # 1 - Deshabilitado
    # 0 - Habilitado

> Instalación de `policycoreutils`

    [ubuntu]# apt install policycoreutils

> Verificar si SELinux está activado con `sestatus`

    [linux]$ sestatus

    >>> SELinux status:                 disabled

* **ADVERTENCIA:** *Debian/Ubuntu* cuenta con *AppArmor* como alternativa a *SELinux*. Además *SELinux* se encuentra en estado experimental. Antes de activarlo es se tiene desactivar *AppArmor*.

En *RHEL/CentOS* ya viene activado por defecto.

    [rhel]# sestatus

    SELinux status:                 enabled
    SELinuxfs mount:                /sys/fs/selinux
    SELinux root directory:         /etc/selinux
    Loaded policy name:             targeted
    Current mode:                   enforcing
    Mode from config file:          enforcing
    Policy MLS status:              enabled
    Policy deny_unknown status:     allowed
    Memory protection checking:     actual (secure)
    Max kernel policy version:      33

[REFERENCIAS]

* [https://www.linux.com/news/securing-linux-mandatory-access-controls/](https://www.linux.com/news/securing-linux-mandatory-access-controls/)

### Configuración de los contextos de seguridad

*SELinux* provee un sistema etiquetas para archivos y carpetas conocidas como *Contextos de Seguridad* (*Security Contexts*). Las cuales extienden los atributos de los recursos. También agrega el mismo tipo de etiquetas a los procesos de sistema y son conocidas como *Dominos* (*Domains*).

Podemos ver los dominios y conextos usando el atributo `-Z` en los comandos `ls` (listado de archivos) y `ps` (listado de procesos).

> Ver las etiquetas de los archivos con `ls -Z`

    SINTAXIS: ls -Z

    EJEMPLO:

    [rhel]# ls -Z

    system_u:object_r:admin_home_t:s0 anaconda-ks.cfg
    unconfined_u:object_r:admin_home_t:s0 ecryptfs-utils-111-19.1.el8.elrepo.x86_64.rpm
    unconfined_u:object_r:admin_home_t:s0 elrepo-release-8.2-1.el8.elrepo.noarch.rpm
    unconfined_u:object_r:admin_home_t:s0 epel-release-8-14.el8.noarch.rpm

> Ver las etiquetas de los procesos con `ps -Z`

    SINTAXIS: ps -Z

    EJEMPLO:

    [rhel]# ps -Z

    LABEL                               PID TTY          TIME CMD
    unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 1634 pts/0 00:00:00 bash
    unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 1827 pts/0 00:00:00 ps

Inspeccionemos las etiquetas:

* **SELinux user** - generalmente `unconfined_u`
* **SELinux role** - generalmente `object_r` (en `ls`) y `unconfined_r` (en `ps`)
* **type** - generalmente `user_home_t` o `admin_home_t` (en `ls`) y `unconfined_t` (en `ps`)
* **sensitivity** - generalmente `s0` (en `ls`) y `s0-s0` (en `ps`)
* **category** - generalmente `c0.c1023` (sólo en `ps`)

Adicionalmente a **SELinux** debemos instalar las herramientas de administración por separado.

> Instalar las herramientas de `SELinux`

    [rhel]# yum install setools policycoreutils policycoreutils-python-utils

> Instalar el comprobador de errores

    [rhel]# yum install setroubleshoot

    # Reiniciar el servicio `auditd`

    [rhel]# service auditd restart

Para poner un ejemplo práctico instalaremos Apache para proveer servicios web con *SELinux* activado.

> Instalar y activar Apache

    [rhel]# yum install httpd

    # Activa e inicia el servicio `httpd`

    [rhel]# systemctl enable --now httpd

> Agregar al `firewall` el puerto de `http`

    [rhel]# firewall-cmd --permanent --add-service=http

    # Aplicar los cambios

    [rhel]# firewall-cmd --reload

> Verificar la información del proceso Apache

    [rhel] ps ax -Z | grep httpd

    system_u:system_r:httpd_t:s0       3870 ?        Ss     0:00 /usr/sbin/httpd -DFOREGROUND
    system_u:system_r:httpd_t:s0       3871 ?        S      0:00 /usr/sbin/httpd -DFOREGROUND
    system_u:system_r:httpd_t:s0       3872 ?        Sl     0:00 /usr/sbin/httpd -DFOREGROUND
    system_u:system_r:httpd_t:s0       3873 ?        Sl     0:00 /usr/sbin/httpd -DFOREGROUND
    system_u:system_r:httpd_t:s0       3874 ?        Sl     0:00 /usr/sbin/httpd -DFOREGROUND

> Tabla de Etiquetas SELinux (Dominio)

Etiqueta | Valor
--- | ---
**user** | `system_u`
**role** | `system_r`
**type** | `httpd_t`
**sensitivity** | `s0`

**TIPO:** `httpd_t`

Apache expone la carpeta de contenido web en `/var/www/html`. Podemos inspeccionarla.

    [rhel]# ls -ld -Z /var/www/html/

    >>> drwxr-xr-x. 2 root root system_u:object_r:httpd_sys_content_t:s0 6 nov 11 23:58 /var/www/html/

> Tabla de Etiquetas SELinux (Contexto)

Etiqueta | Valor
--- | ---
**user** | `system_u`
**role** | `object_r`
**type** | `httpd_sys_content_t`
**sensitivity** | `s0`

**TIPO:** `httpd_sys_content_t`

Ahora modifiquemos el contenido web del `index.html`.

    [rhel]# nano /var/www/html/index.html
    
    ---
    <h1>Apache Funciona! Prueba de SELinux</h1>
    ---

    [rhel]# ls -Z /var/www/html/index.html

    >>> unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/index.html

    [rhel]# curl localhost

    >>> <h1>Apache Funciona! Prueba de SELinux</h1>

Si ahora creamos ese mismo index en la carpeta HOME y luego copiamos el archivo a la carpeta `/var/www/html`, veremos que el archivo no funcionará ya que el tipo será `admin_home_t` en lugar de `httpd_sys_content_t`.

    [rhel]# echo '<h1>Apache No Funciona!!!</h1>' > index.html
    
    [rhel]# ls -Z index.html

    >>> unconfined_u:object_r:admin_home_t:s0 index.html

    [rhel]# mv index.html /var/www/html/
    
    >>> mv: ¿sobreescribir '/var/www/html/index.html'? (s/n) <<s>>

    [rhel]# curl -I localhost

    HTTP/1.1 403 Forbidden
    Date: Sun, 27 Feb 2022 01:58:01 GMT
    Server: Apache/2.4.37 (centos)
    Last-Modified: Sun, 27 Jun 2021 23:47:13 GMT
    ETag: "30c0b-5c5c7fdeec240"
    Accept-Ranges: bytes
    Content-Length: 199691
    Content-Type: text/html; charset=UTF-8

Observamos que ahora la página no funciona, por violar el contexto de Apache. Podemos corregir el problema mediante `chcon`.

    [rhel]# chcon -t httpd_sys_content_t /var/www/html/index.html

    [rhel]# ls -Z /var/www/html/index.html

    >>> unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/index.html

    [rhel]# curl localhost

    >>> <h1>Apache No Funciona!!!</h1>
    
    [rhel]# curl -I localhost
    
    HTTP/1.1 200 OK
    Date: Sun, 27 Feb 2022 02:01:00 GMT
    Server: Apache/2.4.37 (centos)
    Last-Modified: Sun, 27 Feb 2022 01:54:08 GMT
    ETag: "1f-5d8f635232de7"
    Accept-Ranges: bytes
    Content-Length: 31
    Content-Type: text/html; charset=UTF-8

Una vez corregido, el sistema volverá a funcionar.

### Políticas de Seguridad de SELinux

Las políticas de seguridad en SELinux parten de activaciones booleanas que permiten o bloquean la capacidad de realizar alguna cosa dentro del sistema.

Los *booleans* pueden ser consultadas por el comando `getsebool -a` o `getsebool <policy>`.

    [linux]$ getsebool -a

    abrt_anon_write --> off
    abrt_handle_event --> off
    ...
    zoneminder_anon_write --> off
    zoneminder_run_sudo --> off

La mayoría de los procesos tienen políticas establecidas para hacer bloqueos automáticos. Por ejemplo, en Apache y Samba no se podrá acceder a los archivos contenidos en las carpetas HOME de los usuarios.

Si por algún motivo decidieramos cambiar estas políticas, bastaría establecer su valor `on` en lugar de `off` mediante el comando `setsebool <policy> on`.

> Ejemplo: Activar los directorios de usuario en Samba

    [linux]# setsebool samba_enable_home_dirs on

    # Ver el efecto de la activación

    [linux]$ getsebool samba_enable_home_dirs

    >>> samba_enable_home_dirs --> on

    # NOTA: La opción -P hará permanente el cambio, ya que este sólo se aplicará antes del reinicio del sistema (será temporal).

---

En algunos casos será necesario revisar la lista de puertos permitidos por SELinux y agregar o bloquear más puertos. Por ejemplo, para ver los puertos disponibles para Apache tenemos el tipo `http_port_t`. Mediante `semanage port -l` podremos consultar los puertos y decidir agregar o eliminar puertos.

    [linux]# semanage port -l

    Tipo de Puerto SELinux         Proto    Número de Puerto

    afs3_callback_port_t           tcp      7001
    afs3_callback_port_t           udp      7001
    ...
    zookeeper_leader_port_t        tcp      2888
    zope_port_t                    tcp      8021

> Agregar un puerto TCP para `http_port_t`

    [linux]# semanage port -a 82 -t http_port_t -p tcp

    [linux]# semanage port -l | grep 'http_port_t'

    >>> http_port_t                    tcp      82, 80, 81, 443, 488, 8008, 8009, 8443, 9000

    # NOTA: Usa `systemctl restart httpd` para aplicar el cambio

> Eliminar un puerto TCP para `http_port_t`

    [linux]# semanage port -d 82 -t http_port_t -p tcp

    [linux]# semanage port -l | grep 'http_port_t'

    >>> http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000

    # NOTA: Usa `systemctl restart httpd` para aplicar el cambio

---

Para crear nuevos módulos de políticas usaremos `udit2allow`.

[REFERENCIAS]

* [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/using_selinux/writing-a-custom-selinux-policy_using-selinux](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/using_selinux/writing-a-custom-selinux-policy_using-selinux)

---

[![Alan Badillo Salas](https://avatars.githubusercontent.com/u/79223578?s=40&v=4 "Alan Badillo Salas")](https://github.com/dragonnomada) Por [Alan Badillo Salas](https://github.com/dragonnomada)

Estudié **Matemáticas Aplicadas** en la Universidad Autónoma Metropolitana, posteriormente realicé una Maestría en **Inteligencia Artificial** en el Instituto Politécnico Nacional.

He impartido cursos de Programación Avanzada en múltiples lenguajes de programación, incluyendo *C/C++, C#, Java, Python, Javascript* y plataformas como *Android, IOS, Xamarin, React, Vue, Angular, Node, Express*. Ciencia de Datos en *Minería de Datos, Visualización de Datos, Aprendizaje Automático y Aprendizaje Profundo*. También sobre *Sistemas de administración basados en Linux, Apache, Nignx* y *Bases de Datos SQL y NoSQL* como MySQL, SQL Server y Mongo. Desde hace 7 años en varios instituciones incluyendo el *IPN-CIC, KMMX, The Inventor's House, Auribox*. Para diversos clientes incluyendo al **INEGI, CFE, PGJ, SEMAR, Universities, Oracle, Intel y Telmex**.