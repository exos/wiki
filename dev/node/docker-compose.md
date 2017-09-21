<!-- TITLE: Usando docker-composecon node.js -->
<!-- SUBTITLE: Con supervisor y toda la bola -->


Hace rato vengo usando [Docker](https://log.exos.ninja/?q=docker) tanto para dev como en producción para algunos servicios, y una herramienta genial para armar entornos de desarrollo en realmente pocos comandos es [docker-compose](https://docs.docker.com/compose/).

> Antes que nada aclaro unas cosas:

> * **NO** usar la siguiente receta para producción
> * Yo personalmente no recomiendo usar *docker-compose* en producción
> * No voy a entrar en detalles de como instalar/configurar Docker

# Instalación

docker-compose está disponible en varias distribuciones como paquete, y también se puede instalar por medio de [pip](https://pypi.python.org/pypi/pip), en Debian/Ubuntu:

    $ sudo apt-get install docker-compose

En Archlinux existe el paquete [community/docker-compose](https://www.archlinux.org/packages/community/x86_64/docker-compose/):

    # pacman -S docker-compose

Y si se desea instalar a través de pip:

    $ pip install docker-compose

# Creación del Dockerfile

Esta parte es normal a Docker, solo que nuestro *container* va a tener algunas diferencias de si lo estaríamos creando para producción, por ejemplo, vamos a montar el código fuente, levantar nuestra app con [supervisor](https://www.npmjs.com/package/supervisor) *(si nunca lo usaste no es problema)*.

En mi caso me baso en la imágen [mhart/alpine-node](https://hub.docker.com/r/mhart/alpine-node/) simplemente porque al estar basada en alpine es liviana.

Una cosa a tener en cuenta es que el orden de las instrucciones importa mucho, ya que los cambios mas intrusivos van primero para evitar la re-generación de imágenes intermediarias:

    FROM mhart/alpine-node:6
    RUN mkdir -p /src
    ENV DEBUG "*"
    EXPOSE 3000
    WORKDIR /src
    ADD package.json /src/
    RUN npm install --no-optional
    ADD . /src/

La linea `ENV DEBUG "*"` es para mostrar la salida de debug, en caso de utilizar [debug](https://www.npmjs.com/package/debug), en mi caso yo prefiero no setearla en `*` ya que muchas libs lo usan y se vuelve demaciado verborragico, yo las llamadas a `debug` normalmente les pongo un prefijo, al estilo `miapp.{modulo}`, por lo que si es el caso podemos setearla; `ENV DEBUG "miapp.*"`.

Como abrás notado primero agrego el archivo package.json y hago un install de las dependencias, eso hará que la imagen intermediaría de las dependencias quede creada y no necesite ser actualizada cada vez que actualizamos el código fuente de nuestra aplicación. Para esto hay que tener en cuenta que `node_modules` vamos a tener que ignorarlo en docker, agregando "node_modules" en el archivo `.dockerignore`,  a este punto tenemos que tener en cuenta que no necesitamos tener el código fuente para que se instalen estas. También ser cocientes de que se instalarán todas las dependencias, incluyendo las de desarrollo.

Lo siguiente que hacemos es meter todo el arbol de directorios (menos lo ignorado por `.dockerignore`) en `/src`.

Ahora en este caso no le vamos a dar un entry-point ya que eso solo agregaría mas intermediarios y esto lo vamos a trabajar directamente con docker-compose.

Luego, una vez creado nuestro Dockerfile, en vez de hacer el build, vamos a generar nuestro `docker-compose.yml`, que será nuestro archivo de configuración de docker-compose:

    version: '2'
    services:
    db:
        image: postgres:9.6-alpine
        environment:
            POSTGRES_USER: "someuser"
            POSTGRES_PASSWORD: "somepassword"
            POSTGRES_DB: "SomeDBName"
        ports:
            - "7432:5432"
    MyProject:
        build: .
        command: ./node_modules/.bin/supervisor -w lib /src/app.js
        volumes:
            - .:/src
        ports:
            - "3000:3000"
        depends_on:
            - db

De esta forma definimos un container llamado `db` *(usando la imagen `postgres`)* y le seteamos la configuración por defecto que debe tomar a través de variables de entorno y aparte exponemos el puerto `7462` para poder conectarnos localmente a la base de datos y poder manipular los datos a mano. El segundo container es `MyProject` donde en vez de definirle una imagen a usar, le decimos que haga `build` desde `.` *(usando nuestro Dockerfile)*, y que el directorio actual, osea el de nuestro proyecto, va montado como `/src`. Finalmente en command, que sería nuestro `entry-point`, le decimos que corra supervisor con el parametro `-w` (watch), para que si hacemos algún cambio en nuestro código, el servidor se reinicie, sin necesidad de resetear los containers ni nada.

# Levantando el entorno de desarrollo

Para levantar todo de una vez, solo deberemos usar el comando:

    $ docker-compose up

Eso automáticamente creará la imágen de nuestro proyecto, las dependencias y levantará todo.

Si necesitamos ejecutar un comando dentro de nuestro entorno de desarrollo, podemos usar el comando `run` de `docker-compose`, por ejemplo:

    $ docker-compose run MyProject npm run syncdb

Como el `workdir` es `/src`, si corremos npm podremos correr scripts definidos en nuestro `package.json` libremente.

# Rebuild

En el caso de que tengamos un cambio severo en nuestro container principal, por ejemplo, instalamos librerías y tenemos que re-armar la imágen con las dependencias de npm *(tener en cuenta que `node_modules` dentro del container es independiente)* o instalamos alguna librería a nivel sistema, deberemos parar el `docker-compose up`, y ejecutar:

    $ docker-compose build MyProject

Eso forzará el re-build de la imagen.