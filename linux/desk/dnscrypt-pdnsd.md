<!-- TITLE: Configurando DNSCrypt con caché (con pdnsd) en Archlinux -->
<!-- SUBTITLE: Con yapa anti ads -->

> Posteado originalmente en [log/3s](https://log.exos.ninja/3s)

> **Nota**: El manual es para [ArchLinux](http://log.exodica.com.ar/?q=archlinux), pero no tendría que ser difícil de aplicar en otras distribuciones GNU/Linux, sobretodo si usan systemd.

Como muchos saben el protocolo DNS ha sufrido varios [problemas de seguridad](https://en.wikipedia.org/wiki/Domain_Name_System#Security_issues), aparte de que al ir en claro por la red, tanto el ISP como cualquiera que espíe la comunicación sabrá que dominios consultamos, las formas de que esto nos perjudique son muchísimas, desde extracción de información hasta [cache poisoning](https://en.wikipedia.org/wiki/DNS_spoofing) y demás. 

Una solución a esto es [DNSCrypt](https://dnscrypt.org/), el cuál consultará a través de HTTPS, estando toda esta comunicación cifrada de origen a destino, pero esto tiene un defecto, DNS fue hecho para ser rápido, por eso el uso de UDP, esto es por TCP, y aparte usa cifrado, así que podemos deducir que termina siendo lento.
Para resolver este problema, vamos a usar una caché de DNS local, en mi caso opté por [pdnsd](http://members.home.nl/p.a.rombouts/pdnsd/), ya que me gusta mucho su configuración.

# Instalación

    # pacman -S dnscrypt-proxy pdnsd

# Configurando dnscrypt-proxy

Una vez instalado estos dos, empezaremos configurando dnscrypt-proxy, para esto la wiki de Archlinux recomienda editar dnscrypt-proxy.socket, pero como queremos una mejor configuración, vamos a optar por crear el archivo ```/etc/conf.d/dnscrypt-config```,  acá pueden copiar esta configuración:

    DNSCRYPT_LOCALIP=127.0.0.2
    DNSCRYPT_LOCALPORT=53
    DNSCRYPT_USER=nobody
    DNSCRYPT_PROVIDER_NAME=2.dnscrypt-cert.fvz-rec-us-mia-01.dnsrec.meo.ws 
    DNSCRYPT_PROVIDER_KEY=B864:FA77:A58F:F757:6B53:1086:BDF0:6B2F:7D33:1D09:E561:236E:A9ED:557F:F6C3:B7F1
    DNSCRYPT_RESOLVERIP=173.44.61.182
    DNSCRYPT_RESOLVERPORT=443

En este caso estoy usando el server de DNSCrypt ```2.dnscrypt-cert.fvz-rec-us-mia-01.dnsrec.meo.ws``` que se encuentra en miami, pero pueden [elegir entre los muchos que hay](https://github.com/jedisct1/dnscrypt-proxy/blob/master/dnscrypt-resolvers.csv), o montar el suyo con [dnscrypt-wrapper](https://github.com/Cofyc/dnscrypt-wrapper).

En este caso estaremos escuchando en el puerto ```53``` de la ip local ```127.0.0.2```, esto es para no pisar la de ```localhost```, si cambian de servidor, no se olviden de pegar la llave.

Una vez que tengamos esta configuración creada, damos a iniciar el servicio:

    # systemctl stop # Por las dudas
    # systemctl start  dnscrypt-proxy
    # systemctl enable  dnscrypt-proxy # Para que inicie automáticamente en el proximo boot

Si todo va bien, ya tendríamos que tener esto andando. Para probarlo podemos ejecutar:

    $ nslooknslookup log.exodica.com.ar 127.0.0.2

Si esto funciona, ya tenemos nuestro proxy de DNSCrypt corriendo, pero todavía falta el cache....

# Configurando pdnsd

Editamos el archivo ```/etc/pdnsd.conf``` y copiamos lo siguiente (si hay algo, reemplazarlo):

    global {
        perm_cache = 1024;
        cache_dir = "/var/cache/pdnsd";
        run_as = "pdnsd";
        server_ip = 127.0.0.1;
        status_ctl = on;
        query_method = udp_tcp;
        min_ttl = 15m;       # Retain cached entries at least 15 minutes.
        max_ttl = 1w;        # One week.
        timeout = 30;        # Global timeout option (10 seconds).
        neg_domain_pol = on;
        udpbufsize = 1024;   # Upper limit on the size of UDP messages.
    }
    
    server {
        label = "dnscrypt-proxy";
        ip = 127.0.0.2;
        port = 53;
        timeout = 30;
        proxy_only = on;
        # If is OpenDNS, reject their shit
        reject = 208.69.32.0/24,
                 208.69.34.0/24,
                 208.67.219.0/24;
    }
    
    source {
        owner = localhost;
        file = "/etc/hosts";
    }
    
Ahora configuramos al pdnsd para que escuche en ```127.0.0.1:53``` y use de único server a nuestro dnscrypt-proxy, los rangos de ip rechazados son los que resuelve muchas veces [OpenDNS](https://www.opendns.com/) cuando el dominio no existe y demás.

Para probar que esto anda, lo iniciaremos y los testearemos:

    # systemctl stop pdnsd # Por si esta corriendo
    # systemctl start pdnsd
    # systemctl enable pdnsd

Una vez que lo tenemos corriendo, lo probaremos como hicimos con el dnscrypt-proxy:

    $ nslookup log.exodica.com.ar 127.0.0.1

Si esto funciona, es que está todo bien, acá pueden probar la utilidad del proxy, ya que es visible como la segunda consulta a un dominio es mucho mas rápida.

## Usando tmpfs para la caché de pdnsd

Este es un paso opcional, pero es altamente recomendable, sobretodo si tienen un SSD y pretenden evitar accesos de escritura constante.

Como habíamos definido el directorio de de cache en ```/var/cache/pdnsd```, haremos lo siguiente:

    # systemctl stop pdnsd # Por si está prendido
    # rm -rf /var/cache/pdnsd/*

Una vez que tenemos el demonio detenido y el directorio vacío, editaremos el archivo ```/etc/fstab``` y le agregaremos al final:

    tmpfs                   /var/cache/pdnsd                tmpfs   nodev,nosuid,size=1G            0       0

El parámetro ```size=1G``` se puede reemplazar por el tamaño que deseen. Una vez hecho esto, haremos:

    # mount -a

Si no tira error, ya podemos arrancar de nuevo el pdnsd

    # systemctl start pdnsd

# Configurando resolv

En Archlinux por defecto se usa netctl para manejar la red, si este es el caso, el archivo ```/etc/resolv.conf``` será escrito por este o por ```dhcpd```, si se usa una dirección estática, solo se debe cambiar el parametro de los DNS:

    DNS=('127.0.0.1')

Si se usa dhcp, se deberá editar el archivo ```/etc/dhcpcd.conf``` agregando el siguiente parámetro:

    nohook resolv.conf

Para otras configuraciones se puede [ver la wiki de archlinux](https://wiki.archlinux.org/index.php/Resolv.conf#Preserve_DNS_settings).

Una vez que recuperemos la edición manual de ```/etc/resolv.conf``` solo deberemos poner como única linea:

    nameserver 127.0.0.1

De esta forma ya empezaremos a usar nuestro pdnsd como server de DNS.

# Yapa, bloqueando ads

pdnsd tiene una opción que es para negar dominios específicos, esta opción sumada a una [lista de dominios de redes de ads](http://esfriki.com/wEP.md) es una interesante combinación, ya que nos permitirá bloquear publicidades y otras cosas directo por dominio. Para agregar dicha configuración a pdnsd, pueden copiar directamente [estas lineas](http://esfriki.com/wEQ.md) al final del archivo.

Espero les sea útil.