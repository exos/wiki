<!-- TITLE: AN2Linux -->
<!-- SUBTITLE: Recibír notificaciones de Android en ArchLinux -->

> Originalmente posteado en [log/3W](https://log.exos.ninja/3W)

[AN2Linux](https://github.com/rootkiwi/an2linuxclient/)  es una utilidad bastante copada para recibir notificaciones desde Android. Obviamente no solo funciona en ArchLinux, pero en este post nos vamos a centrar en instalarlo para esta distribución.

Primero que nada vamos a necesitar instalar una app en el dispositivo con Android (o basados en android), aunque la app [está en el PlayStore](https://play.google.com/store/apps/details?id=kiwi.root.an2linuxclient&hl=es_419), obviamente voy a recomendar bajarla desde [F-Droid](https://f-droid.org/packages/kiwi.root.an2linuxclient/) *(Si no tienen instalado [F-Droid](https://f-droid.org) se los recomiendo, Aunque desde el link que les pasé se puede descargar directamente el APK)*.

Del lado del PC necesitamos correr el [servidor](https://github.com/rootkiwi/an2linuxserver), que corre con python, en ArchLinux no está disponible en los repositorio oficiales pero lo podemos descargar desde el AUR como [an2linuxserver-git](https://aur.archlinux.org/packages/an2linuxserver-git/), yo uso *yaourt* así que en mi caso:

    $ yaourt -S an2linuxserver-git

Una vez instalado, deberemos correr el ejecutable que trae por primera vez:

    $ an2linuxserver.py

Esto creará un certificado RSA, e iniciará el servidor, por defecto en el puerto `46352` *(si usan firewall no se olviden de habilitarlo por TCP)*.

Ahora vamos al dispositivo, y luego de seguir los pasos para habilitar a la aplicación para tener acceso a las notificaciones y le damos a `Server configuration` en donde podemos agregar tantos server como querramos. Una gran ventaja que tiene es que se puede poner una *whitelist* de SSIDs de redes Wifi's, personalmente recomiendo limitar las redes habilitadas.

El cliente hará la petición de paridad al servidor y en la consola podremos aceptar el *fingerprint* o *huella criptográfica* del cliente y listo. Ahora pode cerrar la aplicación ejecutada desde consola con `Ctr^C` y habilitar el servicio para que ya quede corriendo siempre:

    $ systemctl --user enable an2linuxserver.service
    $ systemctl --user start an2linuxserver.service

No se olviden de habilitar las aplicaciones deseadas en la lista de aplicaciones en `Enabled aplications` ya que por defecto no se habilita ninguna, para probar que todo ande deberemos incluir la aplicación `an2linux` y luego desde el menu de configuración de la app poner `Display test notification`.