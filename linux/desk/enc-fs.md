<!-- TITLE: Cifrando directorios arbitrarios con EncFS -->
<!-- SUBTITLE: Sin archivos loop ni superusuario -->

> Posteado originalmente en [log/3G](https://log.exos.ninja/3G.md)

Suelo tener algunos archivos a los cuales monto como unidades loop, y cifro con [dm-crypt](https://gitlab.com/cryptsetup/cryptsetup/wikis/DMCrypt). El problema es que el tamaño del archivo es fijo, y como no está completamente ocupado, esto hace que se pierda espacio, aparte de que me pasa de quedarme corto y tener que pasar todo a un nuevo loop mas grande. Aparte de esto, necesito hacer todo como root.

Hace un tiempo publiqué [como cifrar nuestra home en Archlinux](http://log.exos.ninja/y) con [eCryptFS](http://ecryptfs.org/), pero cuando se trata de cifrar un directorio arbitrario (o de ubicación aleatorea), no es para nada fácil.

Así que decidí utilizar otra herramienta; [EncFS](https://vgough.github.io/encfs/), que básicamente es:

> EncFS provides an encrypted filesystem in user-space. 

O en español:

> EncFS provee un **sistema de archivos cifrado** en **espacio de usuario**.

Tengo que decir que me sorprendió la facilidad con la que podemos tener directorios arbitrarios cifrados de manera super fácil.

Para crear nuestro directorio secreto, en este caso lo vamos a hacer en una partición montanda dentro de ```/mnt```, obviamente los paths pueden ir en cualquier lado:

# Instalamos

En ArchLinux

    # pacman -S encfs

En Debian/Ubuntu, etc

    # apt-get install encfs

# Creamos un directorio para los datos, y otro para el contenido

    $ mkdir /mnt/storage/.misecreto
    $ mkdir /mnt/storage/misecreto

Obviamente tienen que tener permisos de escritura sobre el directorio en el que vamos a trabajar, pero por todo lo demás, haremos todo con nuestro usuario.

# Montamos por primera vez:

    $ encfs /mnt/storage/.misecreto /mnt/storage/misecreto

Lo único a tener en cuenta es que las rutas tienen que ser absolutas, pero esto no nos afectara si después queremos mover el directorio de datos.

Al ser  la primera vez, lo primero que nos preguntará es si queremos usar la configuración estándar, la *paranoica* o la experta. En este caso la paranoica es una buena opción.

    Creando nuevo volumen cifrado.
    Por favor, seleccione una de las siguientes opciones:
    introduzca "x" para modo de configuración para expertos,
    introduzca "p" para el modo preconfigurado paranoico,
    cualquier otra cosa (o una línea vacía) seleccionará el modo estándar.
    ?> p

Después de eso solo pedirá una contraseña, aunque lo mejor es usar una *passphrase* (o frase-contrseña) así que traten de hacerla lo mas larga y recordable posible, recuerden [que lo que importa es el largo](http://log.exodica.com.ar/D).

Una vez hecho esto el directorio esta montado y listo, podemos probar de copiar archivos dentro o escribir alguno:

    $ echo "Test, estoy grabando un archivo" > archivo.txt

# Desmontamos

Como esto funciona en espacio de usuario, usa FUSE así que podemos ejecutar simplemente:

    $ fusermount -u /mnt/storage/misecreto

Ahora si hacemos un ```ls``` en el directorio ```/mnt/storage/misecreto``` veremos que está vacío, y ```/mnt/storage/.misecreto``` tendrá archivos con nombres extraños.

En fin, eso es todo, nada mas, las próximas veces que montemos nuestro directorio (igual que la primera vez):

    $ encfs /mnt/storage/.misecreto /mnt/storage/misecreto

Este solo preguntará la password y montará sin problemas, si es la segunda vez que lo montan pueden verificar que esté todo bien revisando el archivo de prueba que gravamos antes.

# Permisos 

Algo que no me gustó, es que el archivo de configuración (que tiene nuestra key cifrada) es de lectura para todos, haciendo ```ls -lah``` sobre el directorio de datos veremos:

    rw-r--r--  1 exos exos 1,3K oct  3 03:46 .encfs6.xml

Aunque el nombre podría variar, lo mejor es hacerlo solo lectura para nuestro usuario, así que podemos sacar permiso de lectura y ejecución a *otros* del directorio de datos:

    $ chmod 700 /mnt/storage/.misecreto 

Y proteger aún mas nuestro archivo de configuración:

    $ chmod 400 /mnt/storage/.misecreto/.encfs6.xml

Todo sea por la paranoia.

# Desmontando después de inactividad

Si si ya termino :P, pero esto me pareció copado de poner, EncFS tiene una opción para desmontar después de x minutos de inactividad:

    $ encfs -i 15 /mnt/storage/.misecreto /mnt/storage/misecreto

Eso hará que tras 15 minutos de inactividad, la partición se desmonte automáticamente.