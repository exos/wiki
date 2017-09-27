<!-- TITLE: Tunning GIT (v2.1) -->
<!-- SUBTITLE: Completa guía para tunear la herramienta de git de consola! -->

> Posteada originalmente en [log/U](https://log.exos.ninja/U.md)(v.2.1),  [esfriki.com/nm](http://esfriki.com/9pb)(2.0) y [esfriki.com/nm](http://esfriki.com/nm)(1.0)
---

Nada, me gusta la consola, soy una rata de consola y les quería dar algunos tips para tunear un poco GIT:

# Creando Aliases

Si te molesta teclear ```git checkout feature/332``` o ```git checkout master```, etc. Se pueden crear aliases, yo normalmente el primero que creo es:

    $ git config --global alias.s status -s

Este hace que ```git s``` sea igual que hacer ```git status -s``` (Vista de estado pero simplificada), para que muestre el status de forma resumida. 

Cada alias se configura como el anterior, teniendo:

* **--global**: Para que se aplique globalmente, si no se usa se creará un alias que solo funcione el proyecto actual.
* **alias.s**: Se pone alias.*el_nombre_del_alias*
* **status -s**: Es lo que va a ser reemplazado por el alias

Por ende cada vez que se corra ```git s``` será idéntico a poner ```git status -s```.

De esta forma se pueden hacer aliases para navegar fácilmente por las ramas:

    $ git config --global alias.master checkout master
    $ git config --global alias.develop checkout develop
    $ git config --global alias.co checkout

Deben recordar que esto se guarda en el archivo ```~/.gitconfig``` donde se pueden crear a mano o borrar, el formato es así:

    [alias]
        s = status -s
        c = checkout
        visual = !gitk
        develop = checkout develop
        master = checkout master

# Seteando editor

    $ git config --global core.editor gvim

# Usando gvimdiff para resolver problemas de mergeado!

    $ git config --global diff.tool vimdiff

> Esto también se puede hacer usando *vimdiff*, *kdiff3*, etc.

# Usando gvimdiff para ver los diffs entre versiones

    $ git config --global difftool.prompt false
    $ git config --global alias.d difftool

Se usa:

    $ git d [revisión, archivo, rama. tag, etc.]

Lo que logra que podamos comparar archivos así:

![vimdiff](http://esfriki.com/f/vim01.png)

# Mejorando los logs!

```git log``` tiene una serie de parámetros para mejorar la vista de logs, ver las ramificaciones de los branches (en ascii) y usar colores, una buena combinación es:

    $ log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit

De esta manera los logs se ven así:

![gitlogs](http://esfriki.com/f/console1.png)

Ahora como recordar toda esa cadena es un dolor de huevos, podemos simplemente crear un alias:

    $ git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

Ahora cada vez que hagamos un:

    $ git lg

Veremos el log de forma mucho mas simplificada y legible.

# Colores!

Git acepta colores (siempre y cuando la terminal lo soporte), para habilitarlos:

    $ git config --global color.ui true
    $ git config --global color.status auto
    $ git config --global color.log auto

Para cambiarlos!

    $ git config --global color.status.changed yellow
    $ git config --global color.status.deleted red

Entre todos los colores que se pueden usar están:

* normal:
* black
* green
* yellow
* blue
* magenta
* cyan
* white

También se pueden aplicar propiedades después del color:

* bold
* dim
* ul
* blink
* reverse

Por ejemplo:

    $ git config --global color.status.changed "green normal bold"
    $ git config --global color.status.added "green white blink"
    $ git config --global color.status.untracked "cyan"

Se verá: 

![colors](http://esfriki.com/f/console2.png)

Entre los colores configurables tenemos:

* branch.current
* branch.local
* branch.remote
* diff.meta
* diff.frag
* diff.old
* diff.new
* status.added
* status.changed
* status.untracked

# Usando rebase en los *pulls* (para solucionar conflictos en una rama separada)

El comando ```git pull``` en realidad hace un *fetch* de la rama donde estamos y luego aplica un merge, aveces lo que nos conviene es no intentar hacer el merge si sabemos que se va a romper todo, así que podríamos usar el parámetro *--rebase*:

    $ git pull --rebase

Para usar pull en modo rebase siempre, se puede dejar en la configuración:

    $ git config --global --bool pull.rebase true

Esto también se puede configurar solo para el proyecto actual, quitandole el *--global*.

# hooks

Una de las maravillas de git son sus **hooks**, que básicamente son scripts que se ejecutan en algún momento dado. Todo proyecto tiene sus directorios de hooks es ```.git/hooks``` donde ya hay bastantes de ejemplo, que con tan solo renombrarlos empezarán a funcionar.

Los hooks están buenos para automatizar procesos, por ejemplo, de chequeo, test unitarios, etc. Yo que suelo programar en JavaScript, suelo usar [grunt](http://gruntjs.com/) y [jshint](http://jshint.com/) para mis proyectos, una cosa que hago es, antes de hacer un commit (en el hook ```pre-commit```) chequeo que el código esté bien, de esa manera si cometí algún error, no lo voy a poder commitear.

Por ejemplo, en el archivo ```.git/hooks/pre-commit``` tengo: 

    #!/bin/sh
    # check with jshint
    exec grunt check

Eso ejecuta una tarea de grunt para revisar mi código, si eso da error, entonces el commit se detendrá:

![failcommit](http://esfriki.com/f/console.png)

# Usando ssh-keys diferentes por proyecto.

Aveces tenemos muchas cuentas en un mismo servidor de git porque tal vez una es de la empresa, otra personal, etc. También posiblemente tengamos alguna de deploy, y esa key de deploy es única por proyecto y en un servidor tenemos que usar varias.

Para resolver esto, ya que **no se le puede** indicar a GIT que ssh-key usar, se pueden hacer aliases de SSH.

En el archivo (puede que no exista) ```~/.ssh``` se [pueden configurar](https://help.ubuntu.com/community/SSH/OpenSSH/Configuring) varias cosas, entre ellas, crear aliases SSH configurables con varios parámetros. Un ejemplo:

    Host bitbucket-work:
        HostName bitbucket.org
        User git
        IdentityFile ~/.ssh/id_rsa_bitbucket_work

De esa forma, en vez de clonarlo con su URL original, lo haríamos con ese alias:

    $ git clone git@bitbucket-work:user/repo.git

Si ya está clonado y andando también se puede cambiar desde el archivo ```.git/config``` del proyecto que es donde se configuran los remotes.

# Automatizando el FLOW

Como git es super fácil de usar con branches, [hay workflows](http://nvie.com/posts/a-successful-git-branching-model/) para trabajar de cierta forma donde se aproveche tanto ese poder como se pueda, sobretodo en proyectos de varios desarrolladores donde el workflow se puede volver bastante caótico.

El modelo mas usado es básicamente el de la branch *develop* y de *features* y *releases*:

![flow](https://mockupstogo.mybalsamiq.com/mockups/4191.png)

Para seguirlo necesitamos crear todas las branchs necesarias en el proyecto y luego ir creando por ejemplo las de *features* desde *develop*, las de *hotfixes* desde *master*, después mergear a mano y etc.

Para hacer todo un poco mas automático, existe [git-flow](https://github.com/nvie/gitflow), con la cual después de clonar un repo o de hacer ```git init```, solamente tenemos que tipear:

    $ git flow init

Eso nos preguntará algunos nombres de ramas y otras configuraciones que si le damos todo por default sabemos que va a estar bien.

Luego de eso  tendremos una serie de comandos para hacernos la vida mas fácil. Podemos usar ```git flow help``` para ver todos los comandos que trae:

    ⏺ ~ ▶ git flow help
    usage: git flow <subcommand>
    
    Available subcommands are:
       init      Initialize a new git repo with support for the branching model.
       feature   Manage your feature branches.
       release   Manage your release branches.
       hotfix    Manage your hotfix branches.
       support   Manage your support branches.
       version   Shows version information.
    
    Try 'git flow <subcommand> help' for details.

Para saber como manejarse mejor con esta herramienta, recomiendo [esta excelente cheatsheet](https://danielkummer.github.io/git-flow-cheatsheet/index.es_ES.html).