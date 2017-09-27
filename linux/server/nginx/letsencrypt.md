<!-- TITLE: Configurando un SSL gratis con Let's Encrypt en Nginx -->
<!-- SUBTITLE: Con configuración fácil para proxies reversos -->

> Originalmente posteado en [log/3A](https://log.exos.ninja/3A.md)

---

Hace un tiempo se pueden obtener certificados SSL gratuitos gracias [Let's Encrypt](https://letsencrypt.org/). En un momento había hecho un articulo de como obtener un certificado SSL gratuito por un año en mi viejo blog (que algún día tengo que resucitar) que [quedó en taringa](http://www.taringa.net/post/ebooks-tutoriales/16169512/Como-obtener-un-certificado-SSL-gratis-por-un-ano.html#comment-1551443), lo malo es que muchas distribuciones de GNU/Linux y otros sistemas operativos ya no lo soportan. 

# Instalando Let's Encrypt

No voy a entrar en detalle, pueden seguir las instrucciones en su sitio, en Ubuntu existe el paquete "letsencrypt" y en Arch está en el aur como "letsencrypt-cli".

# Agregando un *document root*

Primero que nada, expongo una situación: Como dice en el subtitulo, queremos que sea una configuración fácil para proxies reversos, ya que en este escenario, no contamos con un *document root* (necesario, ya veremos por qué). El método que usa Let's Encrypt para saber que realmente somos los dueños del sitio al que le queremos sacar un certificado SSL, es generando archivos que pueda leer desde sus servidores y estos archivos los genera la herramienta que nos trae, así que si tenemos esta configuración de proxy reverso, nginx mandaría el tráfico al otro servidor, el cual obviamente no va a tener dicho directorio.

Lo bueno es que sabemos cuál es el directorio, así que con solo agregar estas lineas, nos facilitaremos la cosa:

    server {
        listen 80;
        listen [::]:80;
        server_name eldominio.com;

        location /.well-known {
            allow all;
            root /srv/certs/letscrypt/eldominio.com;
        }

        location / {
            proxy_pass http://10.0.3.40;
            proxy_set_header  X-Real-IP  $remote_addr;
            proxy_set_header Host "eldominio.com";
        }
    }

En este caso hay que tener en cuenta dos cosa:

* eldominio.com es un ejemplo y va a tener que ser el dominio real que ustedes estén configurando.
* ```/srv/certs/letscrypt/eldominio.com``` es un directorio cualquiera al cual el Nginx pueda tener acceso, en mi caso use este pero pueden usar el que quieran.

# Crear el certificado

Una vez que tenemos un *document root*, por lo menos para nuestra verificación, crearemos el certificado:

     # letsencrypt certonly --webroot -w /srv/certs/letscrypt/eldominio.com  -d eldominio.com

Si esto funciona debería haber generado un par de archivos en ```/etc/letsencrypt/live/eldominio.com``` (en realidad son enlaces simbólicos, pero son los que necesitamos):

    cert.pem -> ../../archive/eldominio.com/cert1.pem
    chain.pem -> ../../archive/eldominio.com/chain1.pem
    fullchain.pem -> ../../archive/eldominio.com/fullchain1.pem
    privkey.pem -> ../../archive/eldominio.com/privkey1.pem

Algo a tener en cuenta, según la herramienta, o la versión que utilicemos, las extensiones pueden cambiar, pero los nombres serán los mismo (o muy parecidos).

# Generamos la llave Diffie-Hellman:

Este paso se lo saltean muchas guías pero es importante para tener una configuración segura.

    openssl dhparam 2048 > /etc/letsencrypt/live/eldominio.com/dhparam.pem

# Configuramos Nginx

Para esto vamos a usar el [Generador de configuraciones SSL seguras](https://log.exos.ninja/38) que publiqué hace un tiempo ya que nos ahorra trabajo y nos da una configuración bastante avanzada.

Primero que nada, no nos importa ofrecer nuestra web sin cifrado, así que vamos a forzar la utilización de la versión segura, para esto, cualquier petición que tengamos la redirecionaremos permanentemente a la versión segura:

    server {
        listen 80;
        listen [::]:80;
        server_name eldominio.com;

        # Redirect all HTTP requests to HTTPS with a 301 Moved Permanently response.
        return 301 https://$host$request_uri;
    }

Esto es compatible con el SEO de los sitios, así que no afectará la indexación en buscadores, de hecho podría aumentar ya que estos prefieren los sitios seguros. También compatibiliza todos los links ya existentes.

Para la configuración del sitio:

    server {
        listen 443 ssl;
        listen [::]:443 ssl;
        server_name eldominio.com;

        # certs sent to the client in SERVER HELLO are concatenated in ssl_certificate
        ssl_certificate /etc/letsencrypt/live/eldominio.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/eldominio.com/privkey.pem;
        ssl_session_timeout 1d;
        ssl_session_cache shared:SSL:50m;
    #    ssl_session_tickets off;

        # Diffie-Hellman parameter for DHE ciphersuites, recommended 2048 bits
        ssl_dhparam /etc/letsencrypt/live/eldominio.com/dhparam.pem;

        # modern configuration. tweak to your needs.
        ssl_protocols TLSv1.2;
        ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256';
        ssl_prefer_server_ciphers on;

        # HSTS (ngx_http_headers_module is required) (15768000 seconds = 6 months)
        add_header Strict-Transport-Security max-age=15768000;

        # OCSP Stapling ---
        # fetch OCSP records from URL in ssl_certificate and cache them
        ssl_stapling on;
        ssl_stapling_verify on;

        ## verify chain of trust of OCSP response using Root CA and Intermediate certs
        ssl_trusted_certificate /etc/letsencrypt/live/eldominio.com/chain.pem;

        resolver 8.8.8.8 8.8.4.4 valid=86400;

        location /.well-known {
                allow all;
                root /srv/certs/letscrypt/redmine;
        }

        location / {
                proxy_pass      http://10.0.3.40;
                proxy_set_header  X-Real-IP  $remote_addr;
                proxy_set_header Host "eldominio.com";
        }
    }

Los campos que tenemos que setear correctamente (partiendo de la configuración que nos dio la herramienta de Mozilla, son:

* **ssl_certificate**: Con la ruta completa del archivo ```fullchain.pem```
* **ssl_certificate_key**:  Con la ruta completa del archivo ```privkey.pem```
* **ssl_session_tickets off**:  Esta linea personalmente la comento por un tema de optimización, ya que no permite la re utilización de *tickets* entre peticiones, también puede causar problemas
* **ssl_dhparam**: Con la ruta completa del archivo ```dhparam.pem``` (Este lo generamos nosotros)
* **ssl_trusted_certificate** Con la ruta completa del archivo ```chain.pem```
* **resolver**: En este caso utilizamos los de google ```8.8.8.8 8.8.4.4 valid=86400```

Una cosa que podrán observar es que estas rutas apuntan a enlaces simbólicos, estos se encuentran dentro del directorio ```live```, esto es una forma de que la tool pueda cambiar los certificados (por ejemplo, cuando renovamos) haciéndolo transparente para nosotros.

Otra cosa que pueden observar, es que dejé las lineas de *document root*, y esto es porque nos va a servir para futuras renovaciones del certificado (recuerden que solo dura 3 meses).

Obviamente reiniciamos nuestro nginx para aplicar todos los cambios:

    # systemctl restart nginx

Esto dependerá de la distro en cuestión.

# Renovando el certificado automáticamente

Algo bueno que tiene Letsencrypt es que si bien los certificados duran poco (3 meses), se pueden renovar automáticamente.

Cuando creamos el certificado, se guarda la configuración para poder renovarlo con solo un comando, así que podremos ejecutar:

    # letsencrypt renew

En este caso vamos a poner un cron:

    # crontab -e 

Y agregamos la linea:

    0       0       *       *       *       /usr/bin/letsencrypt renew 2>&1  >> /var/log/letsencrypt-renew.log

Esto correrá la tool de renovación todos los días guardando la salida en ```/var/log/letsencrypt-renew.log```, como dijimos en el caso que el certificado se renueve, el enlace simbólico apuntará al nuevo, por lo que es totalmente transparente para nosotros u el nginx.

# Probando nuestra configuración

Hay varios analizadores online que nos darán varios detalles para saber si nuestra configuración esta bien, el que mas me gustó fue:

https://www.ssllabs.com/ssltest/analyze.html?d=redmine.gentisoft.com

Que también prueba algunas vulnerabilidades comunes, si seguiste esta guía al pie de la letra y tus versiones son relativamente modernas deberías obtener una **A+** de calificación.

Otro que también prueba vulnerabilidades aunque no nos da un resultado tan detallado es Symantec:

https://cryptoreport.websecurity.symantec.com/checker/

# Problema conocido: Nginx no puede acceder al directorio ```.well-known```

Esto en algunas distribuciones sucede, y es causado por SELinux, en vez de decir *"des habiliten SELinux"* como hacen muchas guías (si lo leen alguna vez, busquen la solución sin des habilitarlo, ya que es una tontería), en nuestro caso ejecutamos:

    chcon -Rt httpd_sys_content_t /srv/certs/letscrypt/eldominio.com;