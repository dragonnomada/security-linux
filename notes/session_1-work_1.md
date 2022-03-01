# Trabajo 1 - Usuarios y Permisos

> 1. Crear los usuarios y grupos

Usuario | Grupos
--- | ---
`ana` | `marketing`
`karen` | `marketing`, `sales`
`bety` | `marketing`, `dev`
`juan` | `rh`
`pepe` | `rh`
`paco` | `sales`
`luis` | `sales`
`pedro` | `dev`, `security`

Grupos | Usuarios
--- | ---
`marketing` | `ana`, `karen`, `bety`
`rh` | `juan`, `pepe`
`sales` | `paco`, `luis`, `karen`
`dev` | `bety`, `pedro`
`security` | `pedro`

> 2. Establecer una contraseña para un usuario de `marketing` y uno que esté en `marketing`

> 3. Crear una carpeta llamada `marketing` con propietatorio `nobody:marketing` y permisos 070

    [linux]# mkdir /marketing

    [linux]# chown nobody:marketing /marketing

    [linux]# chmod 070 /marketing

> 4. Iniciar sesión con el usuario de `marketing`

    [linux]$ su - ana

> 5. Ir a la carpeta de `/marketing` y colocar un arhivo llamado `prueba.txt`

    [ana@linux]$ cd /marketing

    [ana@linux]$ touch prueba.txt

    [ana@linux]$ ls -l

    >>> -rwxr-xr-x ana ana prueba.txt ...

> 6. Iniciar sesión con el usuario que no pertenece a `marketing`

    [linux]$ su - paco

> 7. Ir a la carpeta de `/marketing` y colocar un arhivo llamado `prueba.txt`

    [paco@linux]$ cd /marketing

    >>> Permission Denied

    [paco@linux]$ cat /marketing/prueba.txt

    >>> Permission Denied

    [paco@linux]$ ls -ld /marketing

    >>> d---rwx--- nobody:marketing /marketing ....
