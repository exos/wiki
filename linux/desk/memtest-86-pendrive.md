<!-- TITLE: Crear pendrive booteable con Memtest86 -->
<!-- SUBTITLE: Utilizando syslinux -->

> Originalmente posteado en [log/o](https://log.exos.ninja/o)

---

# Preparamos el pendrive

## Hacemos todo como root

    $ sudo -s

## Insertamos el pendrive

Insertamos el pendrive en la PC y revisamos como se llama en nuestro sistema. Si solo tienes un disco duro, seguramente lo encontraras como ```/dev/sdb```, sino ejecuta:

    # dmesg | tail

## Creamos una partición del tipo FAT

    # fdisk /dev/sdb

Una vez dentro ->

    n
    p
    1
    [enter, enter]
    a (activar la partición como arrancable - boot flag)
    1
    t
    b (esto es para FAT32)
    w

## Formateamos la partición

    # mkfs.vfat -F 32 /dev/sdb1

Sacamos y volvemos a insertar el pendrive en la PC

# Instalar Syslinux y memtest

## Ubicamos el archivo MBR

Seguramente lo encontraras en ```/usr/lib/syslinux/```,  sino lo puedes buscar:

    # locate mbr.bin

## Copiamos el MBR en el pendrive

    # cat /usr/lib/syslinux/mbr.bin > /dev/sdb 

## Configuramos Syslinux

    # syslinux /dev/sdb1

En caso de no tenerlo instalado en el sistema linux:

    # aptitude install syslinux

## Copiamos el archivo BIN del software MEMTEST en el pendrive

    # cp memtest86+-4.00.bin /media/usb/memtest

## Creamos un archivo syslinux.cfg en el pendrive

    # cd /media/usb/
    # touch syslinux.cfg
    # nano syslinux.cfg

## Ponemos lo siguiente en el archivo syslinux.cfg:

    default memtest
    label memtest
    kernel memtest
    
Para guardar y salir de nano: ```control + x``` ,  ```y``` y ```[enter]```

## Deberíamos de tener 3 archivos dentro del pendrive

* ldlinux.sys
* memtest
* syslinux.cfg

## Desmontamos el pendrive

Reiniciamos e iniciamos la PC desde el pendrive, deberia aparecer el PROMPT del SYSLINUX

    SYSLINUX 3.63 Debian-2008-07-15 EBIOS Copyright (C) 1994-2008 H.Peter Anvin
    BOOT:_

## Escribimos memtest y presionamos [enter]

    BOOT: memtest  [enter]

Con esto ya debería funcionar.

Adaptado de acá: http://foro.noticias3d.com/vbulletin/archive/index.php?t-325390.html