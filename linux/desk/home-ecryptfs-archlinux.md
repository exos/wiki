<!-- TITLE: Cifrando tu home en Archlinux -->
<!-- SUBTITLE: Utilizando eCryptFS -->

> Originalmente postado en [log/y](https://log.exos.ninja/y)

---

Tengo una notebook nueva, así que le instalé un [ArchLinux](https://log.exos.ninja/?q=archlinux) y antes de cifrar toda la partición con DCrypt pensé en hacerlo *a la Ubuntu* que cifra la home del usuario sin cifrar particiones completas y montando automáticamente la home.

Obviamente me imaginé que Arch esto iba a ser un poco mas difícil, pero la realidad que me encontré es que gracias a estos scripts creados por Ubuntu se hace muy fácil, hasta casi da miedo.

Primer vamos a ver las ventajas de usar este método:

* Anda rápido, no agrega mucho overhead de lectura/escritura.
* No utiliza una partición *física*, por lo que no tenemos que reservar espacio y *perderlo*.
* Se automonta al loguear el usuario.

Esto se hace con [eCryptfs](http://ecryptfs.org/) y los scripts de Ubuntu. Para esto vamos a instalar el paquete ```ecryptfs-utils```.

Tambien vamos a necesitar otros paquetes, para preparar todo:

    # pacman -S ecryptfs-utils fuse lsof

Una vez instalado, vamos a tener que trabajar como root, con el usuario deslogueado, esto es muy importante, no tiene que estar el usuario logueado ni por consola, ni en el X, y si hay servicios de usuario, hay que bajarlos a todos.

Podemos comprobar si hay procesos ejecutados por ese usuario con:

    # ps aux | grep usuario

y si los hay, hay que killearlos.

Por otro lado, hay que tener espacio en ```/home```, por lo menos el peso de la home que vamos a cifrar, ya que el script de migración hará backup.

Ahora, antes de correr la migración, deberemos cargar el módulo, para eso:

    # modprobe ecryptfs

Una vez esto, simplemente ejecutamos el comando:

    # ecryptfs-migrate-home -u usuario

Esto va a empezar a migrar sola la home del usuario, puede tardar un tiempo, dependiendo de lo que pese esta.

Luego de terminar la migración nos va a dar 4 *consejos*, el primero y el segundo es probar que anduvo, aunque en realidad dice, antes de reiniciar, a no preocuparse porque la home queda backupeada en un directorio indicado también.

Para probar, desde el mismo usuario root de el estamos probando, podemos hacer:

    # su usuario
    $ ecryptfs-mount-private
    $ cd ~
    $ touch archivo

y hacer todos los chequeos que creas necesarios, copiar, leer, etc.

Los otros consejos son por ejemplo, generar un hash de recuperación (por si nos olvidamos la contraseña) y cifrar la memoria swap, pero por ahora vamos a omitir esto.

Ahora, cada vez que el usuario se loguee, puede ejecutar el comando ```ecryptfs-mount-private``` aunque se puede hacer automático al inicio de sesión.

Para eso editaremos el archivo ```/etc/pam.d/system-auth``` Agregando las siguientes lineas:

Después de ```auth required pam_unix.so``` poner:

    auth required	pam_ecryptfs.so unwrap    

Sobre (antes de) de ```password required pam_unix.so``` poner:

    password	optional	pam_ecryptfs.so

Y luego de ```session required pam_unix.so```:

    session	optional	pam_ecryptfs.so

De esta forma, al loguearnos automáticamente montará nuestra home :-)