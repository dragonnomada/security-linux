# Sesión 2 / Trabajo 2 - Carpetas cifradas

## Introducción

En este trabajo generaremos una carpeta `/marketing` compartida para los miembros del grupo `marketing` en el que puedan poner sus archivos y modificarlos. Sin embargo, montaremos la misma carpeta de `/marketing` de forma encriptada sobre `/marketing-admin`, donde esta última carpeta sea la de los administradores y ellos puedan colocar archivos que automáticamente queden cifrados para los que no sean administradores de Marketing (los que no pertenezcan al grupo `marketing-admin`).

Es decir, todo lo que se cree en la carpeta `/marketing` estará descifrado y todo lo que se cree en la carpeta `/marketing-admin` quedará cifrado. Y sólo el administrador principal sabrá la contraseña de la carpeta `/marketing-admin`.

> 1. Crear la carpeta `/marketing`

    [ubuntu]# mkdir /marketing

> 2. Establecer la propiedad/autoría de `/marketing` al usuario `nobody` y al grupo `marketing`

    [ubuntu]# groupadd marketing

    [ubuntu]# chown nobody:marketing /marketing

> 3. Establecer los permisos de `/marketing` completos al grupo de `marketing`

    [ubuntu]# chmod 070 /marketing

> 4. Crear un usuario en el grupo `marketing` que genere algunos archivos de prueba

    [ubuntu]# useradd -m -d /home/ana -s /bin/bash ana

    [ubuntu]# usermod -a -G marketing ana

    [ubuntu]# passwd ana

    [ubuntu]# su - ana

    [ana@ubuntu]$ cd /marketing

    [ana@ubuntu /marketing]$ echo "hola soy ana" > ana.txt

> 5. Crear la carpeta `/marketing-admin` para los administradores

    [ubuntu]# mkdir /marketing-admin

> 6. Establecer la propiedad/autoría de `/marketing-admin` al usuario `nobody` y al grupo `marketing-admin`

    [ubuntu]# groupadd marketing-admin

    [ubuntu]# chown nobody:marketing-admin /marketing-admin

> 7. Establecer los permisos de `/marketing-admin` completos al grupo de `marketing-admin`

    [ubuntu]# chmod 070 /marketing-admin

> 8. Montar la carpeta `/marketing` en la carpeta `/marketing-admin`

    # apt install ecryptfs

    [ubuntu]# mount -t ecryptfs /marketing /marketing-admin

> 9. Crear un usuario en el grupo `marketing-admin` que genere algunos archivos de prueba

    [ubuntu]# useradd -m -d /home/bety -s /bin/bash bety

    [ubuntu]# usermod -a -G marketing-admin bety

    [ubuntu]# passwd bety

    [ubuntu]# su - bety

    [bety@ubuntu]$ cd /marketing-admin

    [bety@ubuntu /marketing-admin]$ echo "hola soy bety" > bety.txt

> 10. Intentar leer el contenido de `/marketing/bety.txt` con el usuario `ana`

    [ana@ubuntu /marketing]$ cat bety.txt

    >>> [CÓDIGO CIFRADO / BYTES CIFRADOS]

* **NOTA:** Ambos usuarios (los del grupo `marketing` y los del grupo `marketing-admin`) están trabajando sobre la misma carpeta (`/marketing`), pero los administradores acceden a `/marketing-admin` que hace que todos los archivos que escriban ellos desde ahí queden crifrados para los usuarios con acceso normal a `/marketing`.