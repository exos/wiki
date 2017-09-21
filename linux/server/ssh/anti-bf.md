<!-- TITLE: Configuración definitiva para SSH + iptables -->
<!-- SUBTITLE: Seguro, cómodo, Anti-food, anti-bruteforce y sin servicios como f2b o sshguard. -->

> Originalmente posteado en [log/3U](https://log.exos.ninja/3U)

Esta se puede considerar una actualización a *[
La mejor forma de prevenir ataques de fuerza bruta a SSH con iptables](https://log.exos.ninja/3m)*, pero voy a hacerlo conciso y a mergearlo con *[Restringir a los usuarios SFTP acceso a un directorio principal de un sitio web](https://log.exos.ninja/3n)*.

# Plan

Bueno la idea de esta corta guía es configurar **sshd** para que sea seguro pero cómodo y podamos manejar accesos según grupos, para esto vamos a:

* Mejorar la configuración por defecto
* Asegurarnos de que las llaves sean seguras
* Manejar accesos por grupo
* Agregar configuración de iptables

# Configuración

Buenos los puntos de configuración a tocar en ```/etc/ssh/sshd_config```, no importa si ya vienen así, hay que darlos así (si vienen comentados recomiendo descomentarlos), no voy a entrar en detalle de que es cada cosa, en caso de dudas consulte a su buscador:

    Protocol 2
    UsePrivilegeSeparation yes
    LoginGraceTime 30
    PermitRootLogin no
    StrictModes yes
    MaxAuthTries 3
    RSAAuthentication yes
    PubkeyAuthentication yes
    IgnoreRhosts yes
    RhostsRSAAuthentication no
    HostbasedAuthentication no
    PermitEmptyPasswords no
    ChallengeResponseAuthentication no
    PasswordAuthentication yes
    MaxStartups 10:30:60 
    X11Forwarding no
    StrictModes yes

> Aviso: ```X11Forwarding no``` en caso de que sea un servidor si X.

# Regeneración de llaves

Para asegurarnos de que las llaves sean seguras, vamos a regenerarlas, hay que tener en cuenta que esto podrá afectar a los clientes que ya se conectaron y guardaron los fingerprints, por lo que habrá que borrarlas de ```know_hosts```.

    # ssh-keygen -q -f /etc/ssh/ssh_host_rsa_key -N '' -b4096 -trsa
    # ssh-keygen -q -f /etc/ssh/ssh_host_ecdsa_key -N '' -b521 -tecdsa

> Nota: La llave DSA esta fijada en 1024, así que no hace falta regenerarla.

Una vez hecho esto hay que reiniciar el servicio, y recomiendo verificar conexión antes de cerrar la sesión activa.

# Manejar accesos por grupo

Cualquier usuario del sistema se puede conectar por **SSH** por defecto, pero tal vez no necesitemos esto, para eso solo vamos a habilitar a los usuarios que necesitemos, pero lo vamos a hacer por grupos y no usuario por usuario, para esto:

Creamos dos grupos, *remote* y *sftpusers*:

    # groupadd remote
    # groupadd sftpusers

Ahora agregamos las siguientes lineas al ```/etc/ssh/sshd_config```:

    AllowGroups remote sftpusers

    Match Group sftpusers
        ChrootDirectory %h
        AllowTcpForwarding no
        ForceCommand internal-sftp

Ahora, para habilitar a un usuario a poder entrar por SSH se debe incluir en el grupo ```remote```:

    # usermod -aG remote usuario

Por otro lado si queremos que solo pueda acceder por SFTP, lo agregamos a sftpusers en vez de remote.

> Hay un artículo relacionado llamado [Restringir a los usuarios SFTP acceso a un directorio principal de un sitio web](https://log.exos.ninja/3n).

# Agregar configuración de iptables

Bueno viene la parte mas complicada; iptables. A diferencia de muchas guías, yo no recomiendo cambiar el puerto por defecto, ya que esto puede darle problemas a usuarios mas neofitos. A cambio, una buenas reglas de iptable van a hacer la diferencia.

Hay que tener en cuenta que estas reglas que voy a pasar son para *iptables-persistent* también llamado *netfilter-persistent* y se basan en una política por defecto de *DROP*, osea que TODOS los paquetes están bloqueados salvo en los casos especificados.

En fin, como es medio largo no lo voy a postear acá directamente, así que les dejo el link al gist del [archivo de configuración de iptables](https://gist.github.com/exos/ef828e75f3dfcc0f7284c9fc1e00e2c4)

Esto lo que hará será:

* Si hay mas de dos conexiones en un plazo de 5 segundos dropea, esto es mas anti-flod
* Si hay mas de 4 en 30 segundos, también dropea
* Si hay mas de 10 en los últimos 30 minutos bloquea la ip por ese tiempo, 30 minutos
* Ahora viene lo interesante, si hay mas de 100 conexiones en las pasadas 24 horas, bloquea y por cada nuevo intento de conexión reinicia el tiempo de espera.

Ahora, los primeros bloqueos duran poco, y después de un tiempo el servicio se habilita de nuevo para el cliente en cuestión, pero la última regla, aparte de bloquear actualizará el tiempo de espera, así que el cliente podrá acceder recién 24 horas después de *el último intento de conexión*, no de la última conexión exitosa.

Esto está pensado para los ataques automatizados, ya que si uno es *dropeado*, el atacante reintentará después de un tiempo, de esta forma, si estamos en una lista de ataques automáticos, el simple intento de conectarse va a hacer que el bloqueo no termine. Al fin y al cabo, seguramente seamos borrados de dicha lista.

Otros métodos como fail2ban, pueden hacernos vulnerables a ataques de DOS al llenarnos los logs por ejemplo, de esta forma no corremos ese riesgo y mantendremos a los atacantes bloqueados.

# Yapas y extras:

## Deshabilitar acceso por contraseña

Se puede deshabilitar el acceso por contraseña, forzando a los clientes a hacerlo por medio de una llave SSH, para eso podemos poner, tanto en la configuración general como dentro de un ```Match Group```:

    PasswordAuthentication no

## Política permisiva en lugar de restrictiva

Creamos un grupo para habilitar a los usuarios a conectarse al ssh, pero si en cambio queremos que todos se puedan conectar a menos que se lo prohibamos, podemos hacer lo siguiente:

* Crear un grupo al estilo ```denyssh```
* Comentar la linea de AllowGroups
* Agregar ```DenyGroups denyssh```

## iptables-persistent tira error en la linea *"COMMIT"*

Cuando pasa esto es que el comando de iptables falló por fuera de la sintaxis del archivo de configuración, si esto pasa, revisar ```dmesg```.

## Desbloquear clientes

Puede pasarnos que un cliente haya sido bloqueado por un falso positivo y necesitemos liberarlo, para eso podemos hacer lo siguiente:

    # echo -addr >/proc/net/xt_recent/blocked

Donde ```-addr``` es el ip del cliente, por ejemplo:

    # echo -181.44.65.301 >/proc/net/xt_recent/blocked
    # echo -181.44.65.301 >/proc/net/xt_recent/tmpblocked

> Ips ireales para ejemplo

Si querés limpiar todo:

    # echo / >/proc/net/xt_recent/blocked
    # echo / >/proc/net/xt_recent/tmpblocked

## Módulo *recent* no existe

Este módulo viene en casi todas las distros pero, puede llegar a pasar, si lo necesitás levantar a mano:

    # modprobe xt_recent

## xt_recent: hitcount (100) is larger than packets to be remembered (20)

Este error puede pasar en algunas distros, y es porque el módulo *recent* está limitado a recordar hasta 20 paquetes, para resolverlo tendremos que crear el archivo ```/etc/modprobe.d/ip_pkt_list_tot.conf``` con el siguiente contenido:

    options xt_recent ip_pkt_list_tot=100

Y luego habrá que recargar:

    # iptables -F
    # rmmod xt_recent
    # modprobe xt_recent

# Referencias

* https://serverfault.com/questions/610037/openssh-daemon-ignores-serverkeybits-directive
* https://www.rackaid.com/blog/how-to-harden-or-secure-ssh-for-improved-security/
* https://stackoverflow.com/questions/26936653/ratelimiting-with-iptables-recent-gives-error
* https://stackoverflow.com/questions/23286852/remove-those-entries-from-iptables-recent-list-which-are-not-there-in-an-ipset
* https://edwin-montenegro.blogspot.com.ar/2013/09/restringir-los-usuarios-sftp-la-carpeta.html
* https://compilefailure.blogspot.com.ar/2011/04/better-ssh-brute-force-prevention-with.html