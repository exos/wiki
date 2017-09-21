<!-- TITLE: Transparent Onions -->
<!-- SUBTITLE: Acceder a *hidden services* de TOR de forma transparente -->

> Originalmente posteado en [log/3R](https://log.exos.ninja/3R)

Sin una interfaz bridgeada ni proxies extraños

> Nota: El manual es para [ArchLinux](http://log.exodica.com.ar/?q=archlinux), pero no tendría que ser difícil de aplicar en otras distribuciones GNU/Linux, sobretodo si usan systemd. Igualmente cabe aclarar que ESTE manual es bastante mas complejo que los que suelo subir, y se necesitan conocimientos de manejo de la consola y un poquito de redes.

---

> **IMPORTANTE**: Por otro lado dejo en claro que esto es solo para poder acceder a hidden services sin necesidad de usar browsers aparte, o configurar proxies, pero **NO** es una buena idea si tu objetivo es el anonimato. Este método NO garantiza que se conserve este ya que cualquier consulta por fuera de los *.onion* revelarían tu IP.

# Introducción a los Hidden Service

> Si ya sabés los que son podes pasar al siguiente punto

[TOR (The Onion Router)](https://www.torproject.org/) es mas que una simple mixnet para comprar drogas y probar exploits remotos en páginas del gobierno, es, como su nombre lo dice un *Enrutador*, y aparte de la posibilidad de salir a internet desde dentro de la mixnet, también nos da la posibilidad de servir servicios (valga la redundancia) de forma anónima dentro de la red. De ahí los famosos dominios ```.onion```.

Mas allá del anonimato, también nos da unos buenos beneficios, primero que nada, los dominios *.onion* son una representación en texto de los primeros 80 bits del *fingerprint* en *sha1* de la llave *pública RSA*. Bueno si no entendiste nada no importa, básicamente quiere decir que cada dominio *.onion* es único y muy seguro para ser *clonado* o hacer un *MITM* en el medio. Es como si cada hidden service tuviera su propia implementación de SSL y el dominio fuera el fingerprint (o la "huella") del certificado (algo así), por lo que nuestros datos irán cifrados sin necesidad de usar otros protocolos como TLS/SSL, etc.

Por otro lado, aparte de la seguridad que nos brinda, nos permite tener servicios independientemente de donde se corra, sea dentro de una red privada, una wifi pública, etc, por lo que no necesitaremos una conexión a internet que nos permita abrir puertos entrantes, ni una ip fija, ni servicios como DynDNS, etc,. Esto quiere decir que si por ejemplo creamos un hidden service al SSH de una laptop, siempre que esté prendida y conectada a internet, voy a poder acceder sin importar donde esté conectada.

# Instalando lo necesario

Primero que nada me voy a basar en un post anterior sobre [configurar DNCrypt con caché en ArchLinux](http://log.exodica.com.ar/3s), ya que vamos a usar PDNSD para los DNS.

De por si vamos a necesitar instalar:

* tor
* pdnsd
* iptables-persistent

Instalamos todo:

    # pacman -S pdnsd iptables-persistent tor

Bueno como la configuración de este esta en el otro post, salto directo a la parte de la resolución de dominios *.onion*.

# Configurando TOR

Primero vamos a retocar la configuración de TOR que está en ```/etc/tor/torrc```, y vamos a agregar estas opciones:

    AutomapHostsSuffixes .onion,.exit
    VirtualAddrNetworkIPv4 10.192.0.0/10
    AutomapHostsOnResolve 1
    TransPort 9040
    DNSPort 9053
    TransListenAddress 127.0.0.1
    DNSListenAddress 127.0.0.1

Con esto abrimos el puerto 9040 para TOR transparente y el 9053 para la resolución de DNS, asignándoles ips de red.

Una vez realizado esto iniciamos el servicio:

    # systemctl start tor

Podemos probar que ande haciendo:

    $ nslookup -port=9053 pasternjaui2k53d.onion 127.0.0.1

Lo que nos debería devolver algo así:

    Server:         127.0.0.1
    Address:        127.0.0.1#9053
    
    Non-authoritative answer:
    Name:   pasternjaui2k53d.onion
    Address: 10.227.241.187

Si lo hace, ya tenemos nuestro DNS levantado y resolviendo ips, pero... esta ip es de red, interna, y obviamente no rutea a ningún lado..
Bueno por ahora no nos importa, lo que vamos a hacer ahora es incluir este DNS en la resolución de dominios de **pdnsd**.

# Configurando PDNSD

Partiendo de la configuración del otro post (donde dejamos pdnsd andando), editamos el archivo ```/etc/pdnsd.conf```, ahora vamos a tener un NS (o varios) configurados, lo primero que tenemos que hacer el definir su política como inclusora, y añadir una regla para que excluya los dominios *.onion* y *.exit*:

    server {
        label = "dnscrypt-proxy";
        ip = 127.0.0.2;
        port = 53;
        timeout = 30;
        proxy_only = on;
        policy=included;
        exclude = ".onion", ".exit";
        # If is OpenDNS, reject their shit
        reject = 208.69.32.0/24,
                 208.69.34.0/24,
                 208.67.219.0/24;
    }

Por las dudas, las reglas extras que incluimos son:

        policy=included;
        exclude = ".onion", ".exit";

Ahora solo nos queda agregar el nuevos NS (el de TOR), pero de modo exclusivo (osea que excluya todo), y que solo resuelva los *.onion* y *.exit*:

    server {
        label = "tor dns";
        ip = 127.0.0.1;
        port = 9053;
        timeout = 30;
        policy=excluded;
        include=".onion", ".exit";
        proxy_only = on;
    }

Reiniciamos:

    # systemctl restart pdnsd

Y ahora el pdnsd nos debería resolver estos dominios, podemos probar como antes pero sin necesidad de especificar el NS:

    $ nslookup pasternjaui2k53d.onion                     
    Server:         127.0.0.1
    Address:        127.0.0.1#53
    
    Non-authoritative answer:
    Name:   pasternjaui2k53d.onion
    Address: 10.227.241.187

> Nota; Si no seguiste el tutorial del PDNSD, recuerda que hay que definir los DNS en 127.0.0.1

# Enrutando hacia TOR

Bueno seguimos con el tema de que las ips que resuelven no van a ningún lado, y es que hay que pasar por TOR para que enruten con el service en cuestión, ahí donde viene la parte de **iptables**.

En el archivo ```/etc/iptables/iptables.rules``` (depende de la distro), agregamos la siguiente regla dentro de la sección ```*nat```:

    -A OUTPUT -p tcp -d 10.192.0.0/10 -j REDIRECT --to-ports 9040

Y reiniciamos iptables-persistent:

    # systemctl restart iptables-persistent

Ahora solo probamos con por ejemplo, curl:

    $ curl -I http://pasternjaui2k53d.onion
    HTTP/1.1 200 OK
    Server: nginx/1.6.2
    Date: Tue, 13 Dec 2016 10:25:53 GMT
    Content-Type: text/html
    Content-Length: 2537
    Last-Modified: Tue, 29 Nov 2016 09:59:17 GMT
    Connection: keep-alive
    ETag: "583d5175-9e9"
    Accept-Ranges: bytes

En este caso si nos devuelve el header es que la petición se hizo lo mas bien, igualmente ya se pueden entrar a webs *.onion* desde el navegador normal, cualquier conexión hacia un *.onion* será resuelta por pdnsd y enrutada hacia TOR.

Que te diviertas...