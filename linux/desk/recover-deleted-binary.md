<!-- TITLE: Recuperar un binario ejecutable borrado, si todavía está ejecutándose. -->
<!-- SUBTITLE: Episodio IV "The new hope" -->

> Originalmente posteado en [log/E](https://log.exos.ninja/E)

---

Hace mucho, no voy a decir cuando para no dar pistas :P, borré accidentalmente el disco virtual de una instancia de KVM en producción. Antes de que el cliente lo sepa tire un masivo ping a un par de cercanos y amigos para ver que podía hacer, la mejor pregunta que me hicieron fue:

> ¿Sigue corriendo la VM?

Ahí caí que si seguía corriendo, tenían que estar los datos del archivo imagen de la VM, y dicho  y hecho estaba tal archivo linkeado desde `/proc`.

En fin, no me acuerdo bien de donde lo había sacado pero fue gracias a que el proceso de qemu-kdm *(esto fue hace un tiempo largo)* lo estaba usando y pude copiarlo desde el directorio ```/proc```.

Bueno ahora encontré este artículo que explica masomenos como viene esto y explica como recuperar un binario aún en ejecución.

Resumiendo todo *(aunque recomiendo leer el artículo para entenderlo mejor)*, los binarios ejecutables que están corriendo son linkeados desde ```/proc/{pid}/exe```, donde obviamente ```{pid}``` es el pid del proceso en cuestión.

Después voy a ver si me acuerdo como recuperar archivos borrados y lo posteo.

**Nota en cuestión**:
http://linuxito.com/gnu-linux/nivel-alto/440-como-recuperar-un-archivo-binario-ejecutable-borrado-del-filesystem-si-aun-esta-en-memoria