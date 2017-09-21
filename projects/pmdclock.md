<!-- TITLE: pmdclock -->
<!-- SUBTITLE: Manejando tiempos pomodoro desde la consola *like a boss*. -->

> Originalmente posteado en [log/pmdclock](https://log.exos.ninja/pmdclock)
# Presentando pmdclock

Hace tiempo vengo usando (o intentando usar) la técnica [pomodoro](https://es.wikipedia.org/wiki/T%C3%A9cnica_Pomodoro), para el que no la conozca es un método para manejar tiempos de concentración y relajo entre tareas.

A falta de un programa que me haya gustado para manejar estos tiempos, terminé usando [kteatime](https://www.kde.org/applications/games/kteatime) que no tiene nada que ver con la técnica, pero básicamente es un timer con tiempos preconfigurables.

Queriendo probar hace tiempo la creación de demonios tipo *agents* con [node.js](http://log.exodica.com.ar/?q=node.js) me puse a hacer una utilidad pero con las siguientes características:

* Que sea de linea de comando
* Que tenga un daemon para no tenerlo ocupando un tty
* Que notifique cuando se terminan los tiempos
* Que guarde un registro de las actividades y el tiempo de sobra o falta para perfeccionar las próximas estimaciones
* Que se pueda implementar con [i3wm](https://i3wm.org/) + [i3blocks](https://github.com/vivien/i3blocks) (y otras cosas)

De ahí que nació:

[pmdclock](https://github.com/exos/pmdclock), CLI clock agent/client for pomodoro 

# Instalación

La instalación por [npm](https://www.npmjs.com/package/pmdclock) es sencilla:

    $ sudo npm install -g pmdclock

Otra forma sería hacerlo desde las fuentes:

    $ git clone https://github.com/exos/pmdclock.git
    $ sudo npm install -g

# Configuración

El archivo de configuración será ```~/.pmdclock/config.yml```, cualquier cosa se configura ahí (se crea el archivo si no existe).

Los parámetros completos se encuentran en [el README](https://github.com/exos/pmdclock/blob/develop/README.md#configure). 

Por ejemplo, para hacer que cuando se termine el tiempo reproduzca un sonido podemos escribir:

    runCmd: "mplayer /usr/share/sounds/freedesktop/stereo/complete.oga"

# Uso

El uso de pmdclock es sencillo, con las opciones por defecto los tiempos serán los comunes de pomodoro, osea, 25 minutos de concentración, 5 de descanso, y 25 de descanso largo, que se activa automáticamente después de 4 pomodoros completados.

Igualmente hay que tener en cuenta que usa un agent, para eso tenemos que activarlo primero:

    $ pmdclock agent
    Starting agent
    $

Este comando nos devolverá al prompt y ya está listo, si se desea cerrar este agent, se puede hacer por medio del comando:

    $ pmdclock agent -k

Pero hay que tener en cuenta que si hay un timer activado este se perderá.

Como notaste también, la forma de uso esta basado en comandos tipo git, para verlos todos solo tenemos que hacer:

    $ pmdclock --help

Para ver los parametros de cada comando:

    $ pmdclock rest --help

Bueno, empezamos a trabajar, primero que nada debemos comenzar un pomodoro, al cual le podemos, opcionalmente, dar una descripción:

    $ pmdclock start -n "Pelando papas"
    Task started!
    $

Esto nos devolverá al prompt, así que la pregunta será, ¿como saber que esto esta corriendo?, bueno como dije esto funciona con un agent y funciona 100% en background, esto es así porque está pensando para usar en la misma terminal que desarrollemos. Por defecto igualmente seremos notificados a través de libnotify (notificaciones en el escritorio), por lo que no te preocupes de pasarte de tiempo.

Para ver como va, podemos ejecutar:

    $ pmdclock info
    Pelando papas finish in 17 minutes

Como verán los tiempos son todos *humanizados*, si queremos una salida mas detallada, podemos usar ```--format```:

    $ pmdclock info --format "%n lleva %e y se termina en %r"
    Pelando papas lleva 07:13 y se termina en 18:47

Donde ```%n``` es el nombre, ```%e``` es el tiempo que pasó y ```%r``` el tiempo restante (estos últimos es mm:ss). Para ver la lista completa de variables [pueden ver el README](https://github.com/exos/pmdclock/blob/develop/README.md#0-info-variables)

Si querés cambiar la salida por defecto para no tener que andar usando ```--format``` se puede cambiar desde la configuración:

    infoFormat: "%n lleva %e y se termina en %r"

Ahora solo nos queda concentrarnos en nuestra tarea por los próximos 25 minutos, una vez terminemos (no importa si es antes o después de ser notificados), podemos indicar que terminamos:

    $ pmdclock finish
    Pelando papas finish in 27 minutes

Siguiendo con la técnica pomodoro, lo que nos queda ahora es tomarnos un pequeño descanso de 5 minutos, para eso tenemos el comando *rest*:

    $ pmdclock rest
    Rest started!

Claramente, si usamos pomodoro correctamente, no tendríamos que tener una pausa entre terminar una tarea y empezar un descanso y viceversa, así que podemos cambiar de una a la otra si necesidad de terminarla con *finish*:

    $ pmdclock start
    Task started!
    $ pmdclock rest
    Task finished! in 27 minutes
    Rest started!

De esta forma podemos conseguir un flujo mas continuo.

Todas las actividades que hagamos, serán guardadas en un historial, para poder consultarlas y ver que tan bien llevamos los tiempos, para ver este historial, debemos hacer uso del comando *list*:

    $ pmdclock list
    ┌───┬──────────────────┬──────────────────┬──────────────────┬──────────┐
    │   │ Name             │ Started          │ Finished         │ On time  │
    │ * │ Pelando papas    │ Today at 9:00 AM │ Today at 9:27 AM │ 2 mins   │
    │ - │ Rest             │ Today at 9:27 AM │ ...              │ -3 mins  │
    └───┴──────────────────┴──────────────────┴──────────────────┴──────────┘

Este historial lo podemos borrar cuando queramos con:

    $ pmdclock clear
    History clear, 2 tasks deleted.

Como vimos antes, si llegamos a las 4 tareas realizadas, la próxima vez que indiquemos un descanso este automáticamente nos creara un *large rest*:

    $ pmdclock rest
    Large rest started!, be happy

Esto también es configurable y podemos también forzar un descanso corto:

    $ pmdclock rest --short

Bueno hasta ahí tenemos el funcionamiento básico, luego iré pasteando mas cosas.

Si les interesa el proyecto, no duden en [enviar bugs e ideas](https://github.com/exos/pmdclock/issues).

Saludos.