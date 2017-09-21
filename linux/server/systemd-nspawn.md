<!-- TITLE: Configurando un container Debian en Ubuntu server sin LXC ni Docker -->
<!-- SUBTITLE: Usando systemd-nspawn en Ubuntu/Debian -->

> Originalmente posteado en [log/3V](https://log.exos.ninja/3V)

Hace rato cuando veo a profesionales de IT usando sistemas operativos como M$ windoors o OS/X me agarra una especie de nostalgia mezclada con gran incertidumbre, ¿cómo alguien que supuestamente sabe puede usar un sistema operativo sin cgroups? ¿en que mundo picapiedruzco vive esta gente? ¿en serio levanta todos los servicios en el host?.

Como sea, [cgroups](https://en.wikipedia.org/wiki/Cgroups) es una característica fantástica, que agarro un concepto como la paravirtualización y la simplificó a niveles ridículos, haciendo que levantar un container en GNU/Linux sea algo tan natural como levantar un servicio cualquiera.

Entre LXC, LXD, Docker y toda la gama de tecnologías que aprovechan esta característica del SO del pingüino, existe una menos popular, que viene en [systemd](https://log.exos.ninja/?q=systemd); **systemd-nspawn**.

Si no lo conocés te invito a leer la [wiki de arch](https://wiki.archlinux.org/index.php/Systemd-nspawn) que, como siempre, tiene la mejor documentación.

Pero bueno ahora me toco levantar un container Debian en un Ubuntu Server y noté que a diferencia de Arch tiene unas vueltitas más, así que hago este log.

## Instalando lo necesario

Primero que nada aclaro que estoy en un [Ubuntu Xenial](http://releases.ubuntu.com/16.04/), que es el último LTS, para versiones mas nuevas puede que se diferencie un poco.

Es necesario instalar el paquete **systemd-container** para tener el target *machines*, de tiro también vamos a instalar **debootstrap**:

    $ sudo apt-get install -y systemd-container debootstrap

Una vez hecho esto procederemos a crear el container.

## Creando el container

Primero, debemos ser conscientes que tenemos que trabajar en ```/var/lib/machines```.

    # cd /var/lib/machines
    # debootstrap jessie micontainer

Obviamente ```micontainer``` es el nombre que deseen, como tercer parámetro se puede pasar un mirror específico.

Esto nos va a instalar un *chroot* con Debian Jessie, y esa fue toda la complejidad :P.

Ahora para levantar el container lo único que deberemos hacer es:

    # systemd-nspawn -D micontainer

Y listo! ya está, es así de simple.

Bueno tal vez te aparezca como a mí un error feo:

> Failed to create directory /var/lib/machines/micontainer/sys/fs/selinux: Read-only file system

Pero [aparentemente es ignorable](https://github.com/systemd/systemd/pull/3748).

**Importante**: una vez en esta instancia hay que definir una contraseña al usuario root para poder acceder cómodamente al container.

     # passwd

## Arrancando container on boot

Si, me acabo de dar cuenta que no se como traducir *on boot*, supongo que no quiero usar algo cutre como "al inicio del sistema"...

Primero habilitamos y arrancamos el **target machines**:

    # systemctl enable machines.target
    # systemctl start machines.target

Ahora solo tenemos que levantar el unit **systemd-nspawn@micontainer**, si el root está en el directorio que dijimos tendría que funcionar de una:

    # systemctl start systemd-nspawn@micontainer 

Y obviamente habilitarlo:

    # systemctl enable systemd-nspawn@micontainer 

Ahora se puede entrar poniendo:

    # machinectl login micontainer

Y listo!!... ¿no? bueno, si a esta altura pudiste entrar, seguramente fixearon el bug y te podés saltar la siguiente parte, pero si te da el mismo error que a mi:

> Failed to get login PTY: There is no system bus in container

Bienvenido al ruedo:

## Resolviendo el error con PTY

El por qué no podemos acceder directamente es un problema con varias partes:

**Primero**, por un lado la versión que trae hasta la fecha Ubuntu Server de systemd todavía no tiene [esto parchado](https://github.com/systemd/systemd/commit/f2273101c21bc59a390379e182e53cd4f07a7e71) así que la condición para poder entrar es que en el container estén corriendo **systemd** y **dbus** simultáneamente.

**Segundo**, por una restricción de Debian no podemos logearnos como root en terminales en ```/dev/pts/*``` y si bien muchos recomiendan habilitarlas, Debian dice que es inseguro, y como a mi me encanta la seguridad, no voy a hacer algo que la baje, así que propongo otra solución alternativa.

Primero que nada detenemos el container, y entramos a mano:

    # systemctl stop systemd-nspawn@micontainer
    # cd /var/lib/machines
    # systemd-nspawn --boot -D micontainer

El ```--boot``` es para que inicie systemd dentro del container, una vez que nos de el login nos logueamos como root, e instalamos dbus y sudo:

    # apt-get update && apt-get -y install dbus sudo

Luego creamos un usuario para administrar el container (recuerden que no vamos a poder usar el usuario root):

    # useradd -m -G sudo miusuario
    # passwd miusuario

Una vez hecho esto, podremos entrar como el usuario (en este caso) **miusuario**, primero *apagamos* el container apretando ```Ctrl``` + ```Alt gr``` + ```]``` un par de veces, luego volvemos a levantar desde systemctl:

    # systemctl start systemd-nspawn@micontainer

Y entramos:

    # machinectl login micontainer

Nos logueamos con el usuario que creamos y **no con root** y luego ya podemos hacer un ```sudo su``` o similar.

## Configurando la red

Bueno esto es otro tema ya que hay muchos casos, pero explico un forma muy básica. En Ubuntu por lo menos la red se configura por defecto con la opción ```--network-veth```, esto crea una interfaz llamada ```ve-micontainer``` (donde *micontainer* es el nombre del container). Y por defecto no tiene ninguna configuración de IP.

La manera mas fácil de darle salida a internet al container es instalando en ambos extremos (host y container) ```systemctl-networkd```:

    # apt-get install -y systemctl-networkd

También recomiendan instalar ```systemctl-resolved```, pero podemos, en cambio editar el archivo ```/etc/resolv.conf``` a mano y usar alguno de los tantos dns que hay por ahí.

Otra cosa que necesitamos también es tener habilitado el forwarding del lado del host, para lo que podemos crear el archivo ```/etc/sysctl.d/30-ipforward.conf``` con el siguiente contenido:

    net.ipv4.ip_forward=1
    net.ipv6.conf.default.forwarding=1
    net.ipv6.conf.all.forwarding=1

También tiramos el siguiente comando:

    # sysctl net.ipv4.ip_forward=1

Por último, hacemos nat con iptables:

    # iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
    # iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
    # iptables -A FORWARD -i ve-micontainer -o eth0 -j ACCEPT

Acá suponemos que la salida a internet está en la interfaz ```eth0```. 

Una vez hecho todo esto, reiniciamos ```systemctl-networkd``` en el host:

    # systemctl restart systemctl-networkd

Para configuraciones personalizadas por container se puede usar [el archivo .nspawn](https://www.freedesktop.org/software/systemd/man/systemd.nspawn.html#.nspawn%20File%20Discovery)

## Links utilizados/leídos:

* https://wiki.archlinux.org/index.php/Systemd-nspawn
* https://github.com/systemd/systemd/pull/3748
* https://github.com/systemd/systemd/issues/685
* https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=771675#20
* https://github.com/systemd/systemd/issues/2808
* https://lindenberg.io/blog/post/debian-containers-with-systemd-nspawn/
* https://wiki.archlinux.org/index.php/Systemd-networkd#Usage_with_containers
* https://www.freedesktop.org/software/systemd/man/systemd.nspawn.html
* https://wiki.archlinux.org/index.php/Internet_sharing#Enable_packet_forwarding
* https://www.freedesktop.org/software/systemd/man/systemd.nspawn.html#.nspawn%20File%20Discovery