<!-- TITLE: Teniendo una partición encriptada "portable" -->
<!-- SUBTITLE: Utilizando dm-crypt -->

> Originalmente posteado en [log/u](https://log.exos.ninja/u)

---

> Esta forma tiene algunas desventajas, te recomiendo ver [Cifrando directorios arbitrarios con EncFS](https://wiki.exos.ninja/linux/desk/enc-fs).

---

Hoy por hoy en linux es fácil tener una partición cifrada, y sistemas como Ubuntu permiten encriptar la *home* del usuario. Pero aveces solo necesitamos usar ciertos archivos solo por un momento y después queremos que queden protegidos.

Bueno, lo que yo hice y es un archivo, montable, con un fs encriptado, en solo unos simples pasos.

En este caso vamos a usar **dm-crypt** que viene en la mayoria de distribuciones GNU/Linux, en caso de no tener las herramientas, puedes buscar como instalarlas.

# Generando el archivo

Primer que nada, para complicar mas al que quiera *romper* nuestro archivo de loop, vamos a crear el archivo desde */dev/urandom* para que los datos no tengan sentido. En este caso usar */dev/random* no es tan importante y sería mucho mas lento.

Primer decidimos que tamaño ponerle, en este ejemplo vamos hacerla de 1GB, pero esto se puede ajustar a tus necesidades, creamos el loop:

    $ dd if=/dev/urandom of=~/.secreto bs=1M count=1024

# Convirtiendo nuestro archivo en una *unidad*

Bueno ahora lo trataremos como si fuera un dispositivo, así que lo primero será crearle una partición, lamentablemente esto lo tenemos que hacer como root:

    $ sudo fdisk  ~/.secreto

No voy a explicar acá como usar **fdisk**, pero la cosa sería hacer una partición común y corriente.

# Convertir nuestra *unidad* en una unidad cifrada

Bueno ahora viene la parte divertida, que será cifrar esto, en este caso lo vamos a hacer con el formato lucks y usando las herramientas dm-crypt, de nuevo como root:

    $ sudo cryptsetup -v --cipher aes-xts-plain64 --key-size 256 --hash sha1 --iter-time 1000 --use-random --verify-passphrase luksFormat ~/.secreto

En este caso, estamos usando */dev/random* para generar la llave, el parametro *--iter-time*  le indica que nuestra clave (hasheada con un salt en sha1) se rotará 1000 veces, todos estos parámetros son configurables y harán las cosas mas o menos lentas, dependiendo de los recursos con que se cuenten, aunque esto es solo para la generación de la llave AES, una vez creada la partición correra bastante rápida, sobretodo en sistemas de 64 bits donde las instrucciones AES están en el microprocesador.

En fin, ya tenemos nuestro archivo de loop casi listo. Lo que nos queda ahora es darle un formato, pero antes tendremos que tenerlo como un dispositivo usable.

# Mapeando el dispositivo

    $ sudo cryptsetup open --type luks secreto

En este caso nos creará una unidad llamada **secreto** la cual podemos encontrar en */dev/mapper/secreto*. Obviamente le podemos poner el nombre que querramos.

# Formateando

Ahora solo queda darle formato:

    $ sudo mkfs.ext4 /dev/mapper/secreto

# Montando

Una vez terminado el formato, ya podemos tratar a */dev/mapper/secreto* como si fuera una partición normal, por lo que podemos montarla en un directorio, en este caso, voy a crear *~/secreto* (sin punto al inicio).

    $ mkdir ~/secreto
    $ sudo mount /dev/mapper/secreto ~/secreto
    $ sudo chown -R $USER:$GROUPS ~/secreto

Y listo!, ya tenemos nuestro directorio *secreto*, que será transparente para cualquier uso.

# Desmontando y cerrando el mapper

Obviamente nuestro objetivo es tenerlo montado solo cuando lo necesitemos. Para demontarlo será normal:

    $ umount ~/secreto

Pero el mapper seguirá existiendo, así que la sacamos con:

    $ sudo cryptsetup close secreto

# Para montarlo

Bueno para volver a tenerlo disponible, solo deberemos repetir los pasos de crear el mapper y montarlo, sin formatear obviamente!

    $ sudo cryptsetup open --type luks secreto
    $ sudo mount /dev/mapper/secreto ~/secreto

Obviamente cuando creamos el mapper nos pedirá la contraseña.

# Creando alias para el *.bashrc*

Bueno esta es una yapa, si queremos poder montarlo y demontarlo con solo 2 comandos, podemos agregar la siguiente configuración al *~/.bashrc*.

    alias montar_secreto="sudo cryptsetup open --type luks ~/.secreto secreto && sudo mount /dev/mapper/secreto ~/secreto"
    alias desmontar_secreto="sudo fuser -k ~/secreto; sleep 3; sudo umount  ~/secreto && sudo cryptsetup close secreto"

Esto hará que podamos usar los comandos *montar_secreto* y *desmontar_secreto* desde la consola.