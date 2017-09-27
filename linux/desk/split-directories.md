<!-- TITLE: Dividiendo un directorio completo en varios archivos -->
<!-- SUBTITLE: Para subidas u otras yerbas -->

> Originalmente posteado en [log/3I](https://log.exos.ninja/3I)

Si, no está mal el título, aunque es imposible dividir un directorio, en realidad primero lo metemos en un archivo tar y despúes lo dividimos, pero con este método nos ahorramos el necesitar mucho mas espacio en disco.

# El comando split

Si ya lo conoces pasá al siguiente subtitulo, esto es para los que no conocen este comando, se trata simplemente de una utilidad para dividir un archivo en trozos, pueden echarle una leída a su [página de man](https://linux.die.net/man/1/split) para aprender a usarlo con todas sus funciones. 

Lo que resolvemos en esta entrada es poder dividir un directorio, ya que esta utilidad solo divide archivos.

# Usando pipes

Un problema que podemos tener al hacer un *tar* del directorio es que necesitamos espacio suficiente, igual a lo que pesa el directorio, esto sumado a que cuando dividamos el tar también necesitaremos mas espacio, pero hay una solución, usar **pipes**.

El comando en cuestión es:

    $ tar -cf - directorio_en_cuestion | pv |  split -a 4 -b 99MB /dev/stdin /ruta/a/los/volumenes/nombre_

> Nota: el comando ```pv``` no es necesario, pero es cómodo ver a que velocidad estamos copiando los datos, se le puede pasar por parámetros el peso del directorio para que nos tire un porcentaje aproximado.

Esto lo que hace es:

* Crea un *tar* del directorio en cuestión, respetando el árbol de subdirectorios, pero en vez de guardarlo en un archivo, lo pasamos mediante una *pipe* o *tubería* al comando split, por eso nuestro archivo final va a ser ```-``` que es la forma de indicarle que mande la salida a ```stdout```.
* El comando ```split``` no entiende el parámetro ```-``` así que le pasamos la ruta del *device* que representa la entrada estándar. 

Los parámetros de ```split``` son configurables, en este caso le dije que el tamaño de cada volumen es de 99MB, y ```-a 4``` es que use 4 caracteres para nombrar los volúmenes creados.

Después de eso vamos a tener varios archivos de 99MB cada uno.

# Desempaquetando el directorio

Ahora podríamos unirlos todos en un *tar* y usarlo como tal, pero para no ocupar mas tamaño, vamos a hacerlo directamente, en este caso el proceso será el reverso.

    $ cat /los/volumenes/nombre_* | pv | tar -xvf - 

Eso nos generará el directorio directamente de los volúmenes, si necesidad de generar el *tar*.

Es preferible chequear bien el checksum de los archivos para saber que está todo bien, en la mayoría de configuraciones un ```cat *``` va a leer los archivos en orden, pero si ese no fuera el caso, podemos hacerlo a mano:

    $ find /los/volumenes -type f -name "nombre_*" | sort | xargs -n1 cat | tar -xvf -

# Comprimiendo

En estos ejemplos no se hace compresión, lo que puede ser extremadamente útil, si necesitamos los volúmenes para subirlos a internet o demás.

Como venimos trabajando con el poder de las *pipes* o *tuberías*, las posibilidades son infinitas, cualquier cosa que se quiera hacer es simplemente agregar un comando en el stream, por ejemplo para comprimir:

    $ tar -cf - directorio_en_cuestion | gzip | pv |  split -a 4 -b 99MB /dev/stdin /ruta/a/los/volumenes/nombre_

Desempaquetar:

    $ find /los/volumenes -type f -name "nombre_*" | sort | xargs -n1 cat | gzip -d | tar -xvf -

Y con esto se puede hacer lo que se les ocurra, por ejemplo, cifrar con *gpg*:

    $ tar -cf - directorio_en_cuestion | gpg -er ID_LLAVE_O_MAIL | pv |  split -a 4 -b 99MB /dev/stdin /ruta/a/los/volumenes/nombre_

Desempaquetar:

    $ find /los/volumenes -type f -name "nombre_*" | sort | xargs -n1 cat | gpg -d | tar -xvf -

Como dije, las posibilidades son infinitas.

Espero que les sirva. Saludos.