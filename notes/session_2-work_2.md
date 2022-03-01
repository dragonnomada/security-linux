# Sesión 2 / Trabajo 2 - Carpetas cifradas y compartidas

## Introducción

En este trabajo generaremos una carpeta `/marketing` compartida para los miembros del grupo `marketing` en el que puedan poner sus archivos y modificarlos. Sin embargo, montaremos la misma carpeta de `/marketing` de forma encriptada sobre `/marketing-admin`, donde esta última carpeta sea la de los administradores y ellos puedan colocar archivos que automáticamente queden cifrados para los que no sean administradores de Marketing (los que no pertenezcan al grupo `marketing-admin`).

Es decir, todo lo que se cree en la carpeta `/marketing` estará descifrado y todo lo que se cree en la carpeta `/marketing-admin` quedará cifrado. Y sólo el administrador principal sabrá la contraseña de la carpeta `/marketing-admin`.

> 1. Crear la carpeta `/marketing-shared`

    [ubuntu]# mkdir /marketing-shared

> 2. Establecer la propiedad/autoría de `/marketing-shared` al usuario `nobody` y al grupo `marketing`

    [ubuntu]# groupadd marketing

    [ubuntu]# chown nobody:marketing /marketing-shared

> 3. Establecer los permisos de `/marketing` completos al grupo de `marketing`

    [ubuntu]# chmod 070 /marketing-shared

> 4. Crear un usuario en el grupo `marketing` que genere algunos archivos de prueba

    [ubuntu]# useradd -m -d /home/ana -s /bin/bash ana

    [ubuntu]# usermod -a -G marketing ana

    [ubuntu]# passwd ana

    [ubuntu]# su - ana

    [ana@ubuntu]$ cd /marketing-shared

    [ana@ubuntu /marketing-shared]$ echo "hola soy ana" > ana.txt

> 5. Crear la carpeta `/marketing-admin` para los administradores y la subcarpeta `shared` dónde se montará `/marketing`

    [ubuntu]# mkdir /marketing-admin

    [ubuntu]# mkdir /marketing-admin/shared

    # NOTA: mkdir -p /marketing-admin/shared

> 6. Establecer la propiedad/autoría de `/marketing-admin` al usuario `nobody` y al grupo `marketing-admin`

    [ubuntu]# groupadd marketing-admin

    [ubuntu]# chown nobody:marketing-admin /marketing-admin

> 7. Establecer los permisos de `/marketing-admin` completos al grupo de `marketing-admin`

    [ubuntu]# chmod 070 /marketing-admin

> 8. Montar la carpeta `/marketing-shared` en la carpeta `/marketing-admin/shared`

    # apt install ecryptfs

    [ubuntu]# mount -t ecryptfs /marketing-shared /marketing-admin/shared

> 9. Crear un usuario en el grupo `marketing-admin` que genere algunos archivos de prueba

    [ubuntu]# useradd -m -d /home/bety -s /bin/bash bety

    [ubuntu]# usermod -a -G marketing-admin bety

    [ubuntu]# passwd bety

    [ubuntu]# su - bety

    [bety@ubuntu]$ cd /marketing-admin/shared

    [bety@ubuntu /marketing-admin/shared]$ echo "hola soy bety" > bety.txt

> 10. Intentar leer el contenido de `/marketing/bety.txt` con el usuario `ana`

    [ana@ubuntu /marketing-shared]$ cat bety.txt

    >>> [CÓDIGO CIFRADO / BYTES CIFRADOS]

> 11. Agregar los permisos especiales a `/marketing-shared` (Sticky y SGID)

    [ubuntu]# chmod 3070 /marketing-shared

    # NOTA: Impide borrar no propios y el grupo propietario sería `marketing` para nuevos archivos.

> 12. Establecer los permisos ACL por defecto de `/marketing-shared`

    [ubuntu]# setfacl -m d:u::rw /marketing-shared
    [ubuntu]# setfacl -m d:g::- /marketing-shared
    [ubuntu]# setfacl -m d:o::- /marketing-shared

> 13. Verificar que entre usuarios no puedan borrar, leer, escribir ni ejecutar archivos que no sean suyos.

* **NOTA:** Ambos usuarios (los del grupo `marketing` y los del grupo `marketing-admin`) están trabajando sobre la misma carpeta (`/marketing-shared`), pero los administradores acceden a `/marketing-admin` que hace que todos los archivos que escriban ellos desde ahí queden crifrados para los usuarios con acceso normal a `/marketing-shared`.