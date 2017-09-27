<!-- TITLE: Configurando Dropbox con Docker en Archlinux (+ systemd) -->
<!-- SUBTITLE: Corriendo software privativo encapsulado con cgroups :) -->

> Posteado originalmente en [log/3v](https://log.exos.ninja/3v)

---

Para un trabajo necesito tener un directorio sincronizado con este servicio privativo, aunque soy usuario de [Syncthing](https://en.wikipedia.org/wiki/Syncthing) tengo que tenerlo corriendo por un cliente específico. No me gusta tener código privativo compilado por una empresa en mi sistema, por un tema de ~~seguridad~~ *(paranoia)*. Así que decidí correrlo encapsulado en un container.

> Nota: Este tutorial es para [Archlinux](http://log.exos.ninja/?q=archlinux) pero puede ser fácilmente adaptado a otras distribuciones, sobretodo si usan systemd.

Primero necesitamos tener docker instalado, no me voy a dar detalles de como instalarlo y configurarlo, así que si es tu caso te recomiendo leer [la wiki de arch](https://wiki.archlinux.org/index.php/Docker) donde está mas que claro.

En fin, estoy usando la imagen hecha por [janeczku](https://hub.docker.com/r/janeczku/dropbox/) que anda bien y viene muy bien explicada, pueden seguir su guía, pero igual voy a poner los pasos acá.

# Configurando inotify.max_user_watches

Este paso es opcional, pero es una recomendación de Dropbox para poder monitorear todos los subdirectorios dentro del directorio sincronizado.  Esto lo deberemos hacer como root, editamos el archivo ```/etc/sysctl.d/99-sysctl.conf``` (puede no existir) y agregamos (o modificamos):

    fs.inotify.max_user_watches=100000

Y luego refrescamos sysctl (como root, o con sudo):

    # sysctl -p /etc/sysctl.d/99-sysctl.conf 

# Creando el container

Primero que nada para que el container se identifique con nuestro usuario, le puse de nombre ```dropbox-exos```, así es identificable, en este caso necesitamos dos directorios, uno para el directorio sincronizado, y el otro para los datos de la cuenta:

    $ mkdir -p ~/.config/dropbox-docker
    $ mkdir ~/dropbox

Estos directorios pueden ser los que gusten, solo recuerden que tienen que existir ya que los pasaremos como volúmenes a nuestro container.

Ahora, con la magia de docker, con simplemente correr un comando tendremos todo, pero primero tenemos que tener en cuenta, nuestro ```UID``` y ```GID``` para no tener problemas con los permisos de los archivos (recuerden que los ids manejados dentro del container no son los de nuestro sistema host), para sacar nuestros ids:

    $ id

Ese comando nos dirá cuales son los valores, en mi caso ambos son ```1000``` *(que es el primer usuario creado)*. Una vez obtenido estos, corremos el comando *magico*:

    $ docker run -d --restart=always --name=dropbox-exos -e DBOX_UID=1000 -e DBOX_GID=1000 -v /home/exos/.config/dropbox-docker:/dbox/.dropbox -v /home/exos/dropbox:/dbox/Dropbox janeczku/dropbox

En este caso ```--name=dropbox-exos``` indica el nombre del container, que puede ser el que gusten, el resto de párametros creo que son claros para que son.

# Configurando nuestra cuenta

Este paso es bastante sencillo, ya que solo deberemos entrar a una url, y este paso solo lo haremos una sola vez (mientras el directorio de configuración sea el mismo siempre). Para ver la salida del container corremos (usando el nombre que le hayan puesto, en mi caso es ```docker-exos```)*

    $ docker logs dropbox-exos    

El resultado debería ser parecido a este:

    This computer isn't linked to any Dropbox account...
    Please visit https://www.dropbox.com/cli_link_nonce?nonce=..... to link this device.

Si aparece varias veces no se hagan problema, es solo porque tira el mensajes cada x segundos.

Una vez que entran en la url, siguen los pasos y listo. Para comprobar si tuvo efecto vuelven a ejecutar el comando anterior, si todo está bien, debería mostrar:

    This computer is now linked to Dropbox. Welcome Fulano

Esto ya va a empezar a sincronizar el directorio donde lo definimos.

# Creamos el servicio systemd

En este caso vamos a hacerlo a nivel usuario, no sistema, sobretodo porque si por ejemplo usamos una [home cifrada](http://log.exodica.com.ar/y) los directorios no existirán antes de loguearnos, aparte de esta forma lo podemos manejar sin privilegios (igualmente necesitaremos estar en el grupo ```docker```).

Primero creamos el directorio para services personalizados (por si no existe)

    $ mkdir -p ~/.local/share/systemd/user

Luego creamos el archivo ```~/.local/share/systemd/user/dropbox-docker.service``` con el siguiente contenido:

    [Unit]
    Description=Dropbox exos container

    [Service]
    TimeoutStartSec=0
    Restart=always
    ExecStartPre=-/usr/bin/docker stop dropbox-exos  
    ExecStartPre=-/usr/bin/docker rm dropbox-exos
    ExecStartPre=-/usr/bin/docker pull janeczku/dropbox
    ExecStart=/usr/bin/docker run --rm --restart=always --name=dropbox-exos -e DBOX_UID=1000 -e DBOX_GID=1000 -v /home/exos/.config/dropbox-docker:/dbox/.dropbox -v /home/exos/dropbox:/dbox/Dropbox janeczku/dropbox

    [Install]
    WantedBy=multi-user.target

Una vez creado el archivo, hacemos un *daemon-reload*:

    $ systemctl --user daemon-reload 

Estos destruirá el container cada vez que se startee, y hará un *pull* para obtener la última versión de la imagen antes de arrancarla de nuevo. Que se destruya no quiere decir que se pierda la configuración, ya que esta está montada desde los directorios que le pasamos, así que cuando arranque dropbox debería funcionar con la cuenta ya configurada.

Para probar si esto funciona:

    $ systemctl --user start dropbox-docker
    $ systemctl --user status dropbox-docker

Si todo va bien, podemos dejarlo habilitado para que arranque cada vez que nos logueamos:

    $ systemctl --user enable dropbox-docker

De esta forma nos quedará corriendo siempre, usando nuestra cuenta, pero corriendo en un container separado, si mas acceso a los directorios de configuración y datos.

También es una buena solución para correr múltiples instancias de dropbox con un mismo usuario, o en el mismo sistema (como root). Las posibilidades son varias.

# LAN Sync

Dropbox tiene una característica para sincronizar por red local (LAN), útil si hay varias maquinas en le red con el mismo directorio sincronizando el/los mismo/s directorios, pero como nuestro Dropbox corre en un container, este tiene una ip interna de una red virtual separada de la nuestra, así que esta característica no funcionará, para que funcione, deberemos usar la red del host, para eso, a la orden de ```execStart``` le podemos agregar el parámetro ```--net="host"```.

Espero que les sirva. Cualquier cosa pueden crearse un usuario y comentar.

Saludos.