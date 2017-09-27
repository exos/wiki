<!-- TITLE: Usando Xdebug con PHP-CLI -->
<!-- SUBTITLE: Incluyendo el server embebido en php-cli -->

> Originalmente posteado en [log/S](https://log.exos.ninja/S)

[Xdebug](http://log.exos.ninja/?q=xdebug) es la herramientas mas utilizada para debuggear (hay que ver que pasa con [php>dbg](http://log.exos.ninja/z#phpdbg)) PHP, teniendo soporte para vim, NetBeans, Eclipse, etc.

Su configuración es relativamente fácil y los entornos integrados (IDE's) lo hacen mas fácil todavía.

Una forma muy fácil de indicarle a PHP desde consola que empiece a debuggear, ya sea en un script CLI o corriendo el [servidor embebido](http://php.net/manual/es/features.commandline.webserver.php), y es simplemente seteando la variable ```XDEBUG_CONFIG```.

    $ XDEBUG_CONFIG="idekey=mytest" php myscript.php

Si lo que queremos es correr un site desde consola y testearlo, podemos hacer exactamente lo mismo, antes de ejecutar el server:

    $ XDEBUG_CONFIG="idekey=mytest" php -S 127.0.0.1:3000 index.php

> Robado de acá: https://stackoverflow.com/questions/1947395/how-can-i-debug-a-php-cli-script-with-xdebug