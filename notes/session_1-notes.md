# Sesión 1 - Seguridad y Usuarios

## Agenda

* Linux y entornos virtuales

    - Configuración de un entorno virtual Linux
    - Introducción a RHEL (Red Hat Enterprise Linux)
    - Introducción a Ubuntu
    - Configuración de un Servidor Amazon Linux

* Seguridad en Cuentas de Usuarios y Grupos

    - Usuarios administrativos
    - El grupo predefinido admin
    - El archivo de políticas sudo
    - Usuarios sudo limitados
    - Prevención de ataques de fuerza bruta en contraseñas
    - Bloquear de cuentas de usuarios

* Seguridad del Servidor y el Firewall

    - Introducción a iptables
    - Zonas y servicios
    - Introducción a nftables

## Introducción

Los sistemas basados en Linux son muy utilizados en la industría, debido a su naturaleza Open Source, su comunidad activa y los miles de paquetes optimizados para robustecer un Sistema Operativo seguro y productivo.

Dentro de Linux existen diversas distribuciones como `Debian/Ubuntu`, `Red Hat/CentOS/Fedora/RHEL`, `Suse`, `Arch`, entre muchas otras. Cada distribución posee características únicas en el comportamiento interno de Linux, cómo el gestor de paquetes utilizado para extender las capacidades de software, la estructura de archivos y servicios principales disponibles, políticas de seguridad y reglas de administración en general.

Sin embargo, independiente de la arquitectura, hay un conjunto de herramientas y técnicas que nos permiten asegurar el buen funcionamiento de Linux, el control de accesos de usuarios, las reglas de configuración de peticiones TCP, el cifrado de los discos, carpetas y archivos y la auditoría en directorios, archivos y llamadas al sistema operativo.

En esta sesión aprenderemos a configurar un sistema basado en Linux usando las distribuciones de Ubuntu y Red Hat Enterprise Linux (RHEL) para en lo siguiente aprender a administrar de forma segura a ambos. También veremos como configurar usuarios y grupos, limitar a usuarios administradores tipo `sudo` y el manejo de seguridad en el *firewall* con `iptables` y `nftables`.

## Linux y entornos virtuales

    - Configuración de un entorno virtual Linux
    - Introducción a RHEL (Red Hat Enterprise Linux)
    - Introducción a Ubuntu
    - Configuración de un Servidor Amazon Linux

### Configuración de un entorno virtual Linux

Una forma rápida y segura de probar sistemas linux e interactuar con ellos, es creando *Entornos Virtuales de Linux*. Los cuáles podrán ser ejecutados dentro de otros ambientes como Windows, Linux y Mac. Esto nos permitirá aislar el entorno virtual y establecer restricciones en el uso del CPU, la memoria, la red y capacidad de almacenamiento. 

Podemos crear entornos virtuales de linux mediante herramientas como [VirtualBox](https://www.virtualbox.org), aunque también existen más alternativas.

> Proceso para configurar un entorno virtual

    1. Instalar una herramienta de virtualización como Virtual Box
    2. Descargar una imagen de alguna distribución de Linux
    3. Configurar una nueva máquina virtual para el sistema operativo objetivo
    4. Configurar el CPU, Memoria, Red y Disco de la máquina virtual
    5. Iniciar la Máquina Virtual seleccionando la imagen de linux
    6. Instalar la distribución de linux elegida para nuestra máquina

En la siguiente imagen podemos observar una nueva imagen de linux en Virtual Box siendo instalada.

![Una nueva Máquina Virtual de Linux en Virtual Box](../assets/s1.1.png)

### Introducción a RHEL (Red Hat Enterprise Linux)

La distribución [Red Hat Enterprise Linux](https://www.redhat.com/es/enterprise-linux-8) de *RedHat*, es una de las más populares dentro de la industría, debido a sus capacidades sobre Cloud Computing y demás características Enterprise disponibles bajo las licencias establecidas. Al ser una distribución de Linux comercial, cuenta con gran soporte y una empresa de respaldo que invierte fuertes sumas en seguridad y características superiores a cualquier distribución tradicional de Linux. Esta es una gran alternativa para empresas y corporativos.

La gestión de paquetes en *RHEL* se puede lograr mediante el gestor de paquetes *YUM* o directamente con el adminstrador de paquetes *RPM* (*RedHat Package Manager*). El administrador de paquetes principal es *RPM* y *YUM* será un gestor que facilite el uso del administrador de paquetes *RPM*.

A continuación revisaremos algunos comandos esenciales para el control de paquetes en *RHEL*.

> Instalar paquetes a través de `yum`

    [rhel]$ sudo yum install python3

> Ver paquetes instalados con `rpm -qa`

    [rhel]$ rpm -qa

    # Filtrar los paquetes que contengan `python3`

    [rhel]$ rpm -qa | grep python3

    --- SALIDA ---
    
    python3-unbound-1.7.3-15.el8.x86_64
    python3-libsemanage-2.9-6.el8.x86_64
    python3-pyudev-0.21.0-7.el8.noarch
    ...

> Obtener la información de un paquete con `rpm -qi <package>`

    [rhel]$ rpm -qi python36-*

    --- SALIDA ---

    Name        : python36
    Version     : 3.6.8
    Release     : 38.module+el8.5.0+12207+5c5719bc
    Architecture: x86_64
    Install Date: Thu 24 Feb 2022 12:43:24 AM UTC
    Group       : Unspecified
    Size        : 13131
    License     : Python
    Signature   : RSA/SHA256, Mon 23 Aug 2021 02:43:51 PM UTC, Key ID 199e2f91fd431d51
    Source RPM  : python36-3.6.8-38.module+el8.5.0+12207+5c5719bc.src.rpm
    Build Date  : Wed 11 Aug 2021 10:56:38 AM UTC
    Build Host  : x86-vm-14.build.eng.bos.redhat.com
    Relocations : (not relocatable)
    Packager    : Red Hat, Inc. <http://bugzilla.redhat.com/bugzilla>
    Vendor      : Red Hat, Inc.
    URL         : https://www.python.org/
    Summary     : Interpreter of the Python programming language
    Description :
    Python is an accessible, high-level, dynamically typed, interpreted programming
    language, designed with an emphasis on code readibility.
    It includes an extensive standard library, and has a vast ecosystem of
    third-party libraries.

    ...

> Ver la lista de ficheros asociados a un paquete con `rpm -ql <package>`

    [rhel]$ rpm -ql python36-3.6.*

    --- SALIDA ---

    /usr/bin/easy_install-3
    /usr/bin/pip-3
    /usr/bin/pip3
    /usr/bin/pydoc-3
    /usr/bin/pydoc3
    /usr/bin/python3
    /usr/bin/python3.6
    /usr/bin/python3.6m
    /usr/bin/pyvenv-3
    /usr/bin/unversioned-python
    /usr/share/doc/python36
    /usr/share/doc/python36/README
    /usr/share/licenses/python36
    /usr/share/licenses/python36/LICENSE
    ...

> Ver la lista de ficheros de configuración de un paquete con `rpm -qc <package>`

    [rhel]$ rpm -qc bash

    --- SALIDA ---

    /etc/skel/.bash_logout
    /etc/skel/.bash_profile
    /etc/skel/.bashrc

> Listar los paquetes instalados con yum usando `yum list --installed`

    [rhel]$ yum list --installed | grep python3

    --- SALIDA ---

    python3-audit.x86_64        3.0-0.17...     @anaconda
    python3-babel.noarch        2.5.1-5.el8     @koji-override-1
    python3-cffi.x86_64         1.11.5-5.el8    @anaconda
    python3-chardet.noarch      3.0.4-7.el8     @anaconda
    python3-configobj.noarch    5.0.6-11.el8    @anaconda
    ...

> Desisntalar un paquete con `rpm -e <package>` y `yum remove <package>`

    [rhel]$ sudo yum remove python36.x86_64

    --- SALIDA ---

    Dependencies resolved.
    ==================================================================================================================================================
    Package                       Architecture      Version                                             Repository                              Size
    ==================================================================================================================================================
    Removing:
    python36                      x86_64            3.6.8-38.module+el8.5.0+12207+5c5719bc              @rhel-8-appstream-rhui-rpms             13 k
    Removing unused dependencies:
    python3-pip                   noarch            9.0.3-19.el8                                        @rhel-8-appstream-rhui-rpms            2.8 k
    python3-setuptools            noarch            39.2.0-6.el8                                        @rhel-8-baseos-rhui-rpms               450 k

    Transaction Summary
    ==================================================================================================================================================
    Remove  3 Packages

    Freed space: 466 k
    Is this ok [y/N]:

### Introducción a Ubuntu

La distribución de [Ubuntu](https://ubuntu.com) de *Canoical*, es una de las más populares a nivel usuario, por su facilidad de uso y gran soporte de paquetes, gracias a que está basada en [Debian](https://www.debian.org). Una de las principales características que le diferencias de otras distribuciones no comerciales, es el amplio soporte dado por *Canonical* y es una gran alternativa a infraestructuras no comerciales para empresas medianas y gobiernos.

La administración y gestión de paquetes en *Ubuntu* puede lograrse mediante *APT* (*Advanced Package Tool*) y *DPKG*.

A continuación revisaremos algunos comandos esenciales para el control de paquetes en *Ubuntu*.

> Actualizar el índice de paquetes con `apt update`

    [ubuntu]$ sudo apt update

    --- SALIDA ---

    Hit:1 http://mx.archive.ubuntu.com/ubuntu focal InRelease
    Hit:2 http://mx.archive.ubuntu.com/ubuntu focal-updates InRelease
    Hit:3 http://mx.archive.ubuntu.com/ubuntu focal-backports InRelease
    Hit:4 http://mx.archive.ubuntu.com/ubuntu focal-security InRelease
    Reading package lists... Done
    Building dependency tree
    Reading state information... Done
    44 packages can be upgraded. Run 'apt list --upgradable' to see them.

> Listar los paquetes actualizables

    [ubuntu]$ apt list --upgradable

    --- SALIDA ---

    Listing... Done
    alsa-ucm-conf/focal-updates 1.2.2-1ubuntu0.11 all [upgradable from: 1.2.2-1ubuntu0.9]
    base-files/focal-updates 11ubuntu5.5 amd64 [upgradable from: 11ubuntu5.4]
    cloud-init/focal-updates 21.4-0ubuntu1~20.04.1 all [upgradable from: 21.2-3-g899bfaa9-0ubuntu2~20.04.1]
    ...

> Actualizar los paquetes instalados con `apt upgrade`

    # Listar los paquetes actualizables

    [ubuntu]$ apt list --upgradable

    # Actualizar todos los paquetes

    [ubuntu]$ sudo apt upgrade
    
    # Actualizar un paquete en específico

    [ubuntu]$ sudo apt upgrade <package>

    # Instalar versión más reciente de un paquete

    [ubuntu]$ sudo apt install --only-upgrade <package>

> Instalar nuevos paquetes con `apt install <package>`

    [ubuntu]$ sudo apt install <package>

    # Instalar múltiples paquetes

    [ubuntu]$ sudo apt install <package 1> <package 2> ...

    # Instalar desde un archivo `.deb`

    sudo apt install <path>/<file>.deb

> Ver la información de un paquete con `apt show <package>`

    [ubuntu]$ apt show python3

    --- SALIDA ---

    Package: python3
    Version: 3.8.2-0ubuntu2
    Priority: important
    Section: python
    Source: python3-defaults
    Origin: Ubuntu
    Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
    Original-Maintainer: Matthias Klose <doko@debian.org>
    Bugs: https://bugs.launchpad.net/ubuntu/+filebug
    Installed-Size: 194 kB
    Provides: python3-profiler
    Pre-Depends: python3-minimal (= 3.8.2-0ubuntu2)
    Depends: python3.8 (>= 3.8.2-1~), libpython3-stdlib (= 3.8.2-0ubuntu2)
    Suggests: python3-doc (>= 3.8.2-0ubuntu2), python3-tk (>= 3.8.2-1~), python3-venv (>= 3.8.2-0ubuntu2)
    Replaces: python3-minimal (<< 3.1.2-2)
    Homepage: https://www.python.org/
    Task: minimal, ubuntu-core
    Download-Size: 47.6 kB
    APT-Manual-Installed: no
    APT-Sources: http://mx.archive.ubuntu.com/ubuntu focal/main amd64 Packages
    Description: interactive high-level object-oriented language (default python3 version)
    Python, the high-level, interactive object oriented language,
    includes an extensive class library with lots of goodies for
    network programming, system administration, sounds and graphics.
    ...

> Ver los archivos de un paquete con `dpkg -L <package>`

    [ubuntu]$ dpkg -L python3

    --- SALIDA ---
    /.
    /usr
    /usr/bin
    /usr/lib
    /usr/lib/valgrind
    /usr/lib/valgrind/python3.supp
    /usr/share
    /usr/share/doc
    /usr/share/doc/python3
    /usr/share/doc/python3/copyright
    /usr/share/doc/python3/python-policy.dbk.gz
    /usr/share/doc/python3/python-policy.html
    /usr/share/doc/python3/python-policy.html/build_dependencies.html
    /usr/share/doc/python3/python-policy.html/embed.html
    /usr/share/doc/python3/python-policy.html/index.html
    /usr/share/doc/python3/python-policy.html/module_packages.html
    /usr/share/doc/python3/python-policy.html/other.html
    /usr/share/doc/python3/python-policy.html/packaging_tools.html
    /usr/share/doc/python3/python-policy.html/programs.html
    /usr/share/doc/python3/python-policy.html/python.html
    /usr/share/doc/python3/python-policy.html/python3.html
    /usr/share/doc/python3/python-policy.html/upgrade.html
    /usr/share/doc/python3/python-policy.txt.gz
    /usr/share/doc/python3.8
    ...

> Ver los archivos de configuración de un paquete

    [ubuntu]$ ls /var/lib/dpkg/info/*.conffiles | grep bash

    --- SALIDA ---

    /var/lib/dpkg/info/bash-completion.conffiles
    /var/lib/dpkg/info/bash.conffiles

    # Ver el contenido de los archivos

    [ubuntu]$ cat $(ls /var/lib/dpkg/info/*.conffiles | grep bash)

    --- SALIDA ---

    /etc/bash_completion
    /etc/profile.d/bash_completion.sh
    /etc/bash.bashrc
    /etc/skel/.bash_logout
    /etc/skel/.bashrc
    /etc/skel/.profile

> Eliminar paquetes paquetes con `apt remove <package>`

    [ubuntu]$ sudo apt remove python3

    # Eliminar también los archivos residuales

    [ubuntu]$ sudo apt purge <package>

### Configuración de un Servidor Amazon Linux

[Amazon](https://aws.amazon.com/es/ec2/) provee un conjunto de arquitecturas e infraestructura basadas en la nube y el *Cómputo en la Nube* que permiten configurar rápidamente servidores basados en diversas distribuciones de Windows y Linux de forma sencilla.

Una de las principales características es que configura las mínimas reglas de seguridad para brindar servidores seguros.

Al configurar un servidor de Amazon podemos elegir el tipo de servidor, si será virtual o dedicado y también la ubicación, distribución y demás aspectos.

En la siguiente imagen se muestra la interfaz para el lanzamiento de nuevas instancias tipo Amazon EC2 listas para la producción.

![Lanzamiento de nuevas instancias en Amazon EC2](../assets/s1.2.png)

Durante la configuración de la instancia podremos crear un archivo de claves `.pem` para conectarnos a las nuevas instancias. Para hacerlo bastará crear una conexión remota *SSH* y apuntar a la IP asignada de la instancia, el usuario por defecto para la distribución y el archivo `.pem` creado.

> Conectar una instancia vía SSH de Amazon EC2

    [host]$ ssh -i "<path>/<file>.pem" <user>@<ip>

    # Dónde:
    #   <path> - Ruta hacía el archivo .pem
    #   <file> - Nombre del archivo .pem
    #   <user> - Nombre del usuario por defecto
    #   <ip>   - IP, Host o DNS de la instancia

En la siguiente imagen se observa la conexión vía SSH desde [MobaXterm](https://mobaxterm.mobatek.net).

![MobaXterm](../assets/s1.3.png)

## Seguridad en Cuentas de Usuarios y Grupos

    - Usuarios administrativos
    - El grupo predefinido admin
    - El archivo de políticas sudo
    - Usuarios sudo limitados
    - Prevención de ataques de fuerza bruta en contraseñas
    - Bloquear de cuentas de usuarios

### Usuarios administrativos



### El grupo predefinido admin



### El archivo de políticas sudo



### Usuarios sudo limitados



### Prevención de ataques de fuerza bruta en contraseñas



### Bloquear de cuentas de usuarios




## Seguridad del Servidor y el Firewall

    - Introducción a iptables
    - Zonas y servicios
    - Introducción a nftables

---

[![Alan Badillo Salas](https://avatars.githubusercontent.com/u/79223578?s=40&v=4 "Alan Badillo Salas")](https://github.com/dragonnomada) Por [Alan Badillo Salas](https://github.com/dragonnomada)

Estudié **Matemáticas Aplicadas** en la Universidad Autónoma Metropolitana, posteriormente realicé una Maestría en **Inteligencia Artificial** en el Instituto Politécnico Nacional.

He impartido cursos de Programación Avanzada en múltiples lenguajes de programación, incluyendo *C/C++, C#, Java, Python, Javascript* y plataformas como *Android, IOS, Xamarin, React, Vue, Angular, Node, Express*. Ciencia de Datos en *Minería de Datos, Visualización de Datos, Aprendizaje Automático y Aprendizaje Profundo*. También sobre *Sistemas de administración basados en Linux, Apache, Nignx* y *Bases de Datos SQL y NoSQL* como MySQL, SQL Server y Mongo. Desde hace 7 años en varios instituciones incluyendo el *IPN-CIC, KMMX, The Inventor's House, Auribox*. Para diversos clientes incluyendo al **INEGI, CFE, PGJ, SEMAR, Universities, Oracle, Intel y Telmex**.