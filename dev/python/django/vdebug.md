<!-- TITLE: Utilizando Vdebug con Django -->
<!-- SUBTITLE: Django + Vim + Vdebug -->

> Originalmente posteado en [log/3L](https://log.exos.ninja/3L)

Bueno como habrán visto en algún que otro screenshot en este sitio, soy un ferviente usuario de **vim**,  y una extensión que me encanta es [Vdebug](https://github.com/joonty/vdebug), que es un cliente para vim del protocolo *DBGP*.

Pero nunca había podido hacerlo funcionar con DJango, no sé si por un tema de threading o qué, pero nunca me había funcionado. Hasta que encontré este post: [Django debug with vim and vdebug](https://www.abidibo.net/blog/2016/04/06/django-debug-vim-and-vdebug/) (inglés) donde el autor, [Stefano Contini](https://plus.google.com/104134317604017984728) publica un patch para el paquete ```dbgp``` de python.

En su artículo está como usarlo, pero voy a tirar las instrucciones acá también, aclarando que esto es *robado* de su artículo.

Primero que nada entiendo que tenés vim configurado y vdebug instalado, siendo así procedemos, también, en mi escenario se usa VirtualEnv:

Primero que nada instalamos ```dbgp``` en una versión especifica que es para la que está hecha el parche:

    $ pip install dbgp==1.1
 
Luego bajamos el parche, y lo aplicamos:

    $ wget https://gist.githubusercontent.com/abidibo/e1fc75c4e574108cc675f1896f7543d4/raw/789779398b530cc1810dcd5564a13e22128a396c/client.patch
    $ patch env/lib/python2.7/site-packages/dbgp/client.py < client.patch 

Recuerden reemplazar la ruta del archivo ```client.py``` dentro del directorio con el environment virtual.

Una vez aplicado el parche ya está, es todo, ahora desde nuestro código (si esta solución se usa como la famosa ```pdb.set_trace()```):

    import dbgp.client
    dbgp.client.brk(host="localhost", port=9000)

Y podremos debuguear desde vim con Vdebug y todas sus facilidades que incluye.

En mi caso:

![vdebug](http://esfriki.com/f/vdebug.png)