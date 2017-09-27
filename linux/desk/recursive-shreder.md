<!-- TITLE: shred recursivo -->
<!-- SUBTITLE: shred'eando todos los archivos de un directorio -->

> Posteado originalmente en [log/3j](https://log.exos.ninja/3j)

---

Lamentablemente el comando [shred](https://en.wikipedia.org/wiki/Shred_%28Unix%29) no tiene una opción para hacerlo recursivo, pero eso en Linux *(o en cualquier unix en realidad)* no es problema, ya que contamos con muchas herramientas para esto, esta es una forma de hacerlo:

    $ find logs/ -type f -exec shred -zvun 3 {} \;
		
Donde:

* **-type f**: Es que solo buque archivos
* **-exec ...**: Ejecuta ese comando sobre los archivos encontrados
*  **-z**: Rellena el nombre de zeros para que no quede registro en el FS
*  **-u**: Borra el archivo después de *shredearlo*
*  **-n 3**: Reescribe 3 veces los datos

Después de eso ya podemos borrar el directorio tranquilamente.

    $ rm -rf logs