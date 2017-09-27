<!-- TITLE: Descargando de forma parelela con wget -->
<!-- SUBTITLE: O cualquier otro comando -->

> Originalmente posteado en [log/36](https://log.exos.ninja/36)

---

Si necesitamos descargar un par de urls pero de forma paralela, solo necesitamos tener el listado en un archivo y ejecutar:

    $ cat urls.txt | xargs -n 1 -P 10 wget

Los parametros son para:

* **-n 1** Envía solo un parámetro a la vez
* **-P 10** Ejecuta en paralelo hasta 10 procesos

> Sacado de acá: [Parallel downloads with wget](http://jmuras.com/blog/index.html%3Fp=90.html)