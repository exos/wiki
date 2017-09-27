<!-- TITLE: Borrar la meta-data de todas las imágenes de un directorio -->
<!-- SUBTITLE: Desde linea de consola y en una sola linea :) -->

> Originalmente posteado en [log/3g](https://log.exos.ninja/3g)

---

Para esto voy a usa [ExifTool](http://www.sno.phy.queensu.ca/~phil/exiftool/), un script en Perl para ver/escribir la meta-data en imágenes. 

Para instalar en debian/Ubuntu:

    $ sudo apt-get install libimage-exiftool-perl

Después podemos hacer uso de un par de comandos combinados, primeramente no voy a buscar por extensión, sino que lo voy a hacer directamente por el mime indicado con el comando file, así que voy a usar el ```find``` para listar todos los archivos, al que le ejecutaré el comando ```file```, luego con ```grep``` voy a listar solo los que indiquen ser imágenes. Con ```sed``` voy a borrar la parte que indica el tipo en listado y luego con ```xargs``` voy a ejecutar finalmente el ```exiftool``` para que vacíe toda la meta-data.

    $  find . -type f -exec file {} \; | grep -o -P '^.+: \w+ image' | sed "s/:[^:]\+\$//g" | xargs -n1 -P 4 exiftool -r -overwrite_original -P -all= -gps:all=  -copyright=""

También podemos usar el parámetro de *xargs* ```-P``` el cual levanta varios procesos, como vimos en [como hacer descargas paralelas con wget y xargs](https://log.exos.ninja/36), de esta forma se puede poner la cantidad de cores que se tenga para optimizar la tarea.

También se pueden setear propiedades como ```-copyright=""``` por ejemplo, si tenés un directorio de fotos tuyas y querés agregarles tu propio copyright.

En fin, ya con eso podremos retirar toda la data de geolocalización y demás.