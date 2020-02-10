<!-- TITLE: Usando KDE Connect en Archlinux con i3 -->
<!-- SUBTITLE:Válido para otras distros y configuraciones -->

> Originalmente posteado en [log/43](https://log.exos.ninja/43)

## Primero instalamos **kdeconnect**:

    # pacman -S kdeconnect

## Agregamos el unit file:

    # vim /usr/lib/systemd/user/kdeconnectd.service

Y le metemos el siguiente contenido:

    [Unit]

    Description=KDE Connect

    After=graphical.target

    

    [Service]

    Environment=DISPLAY=:0

    ExecStart=/usr/lib/kdeconnectd

    ExecStop=/usr/bin/kquitapp5 kdeconnectd

    Restart=on-failure

    BusName=org.kde.kdeconnect

    

    [Install]

    WantedBy=default.target


> Robado de [acá](https://gist.github.com/PedroHLC/7764e82fde94f54829c4fd47a0eaa822)

## Arrancamos y habilitamos:

    $ systemctl --user start kdeconnectd
    $ systemctl --user enable kdeconnectd

## Agregamos reglas al firewall *(de ser necesario)*

Necesitamos los puertos tanto TCP como UDP `1714:1764`, usando iptables sería:

    # iptables -A INPUT -s 192.168.0.0/16 -p udp --dport 1714:1764 -m conntrack --ctstate NEW -j ACCEPT
    # iptables -A OUTPUT -p udp --sport 1714:1764 -m conntrack --ctstate NEW -j ACCEPT
    # iptables -A INPUT -s 192.168.0.0/16 -p tcp --dport 1714:1764 -m conntrack --ctstate NEW -j ACCEPT
    # iptables -A OUTPUT -p tcp --sport 1714:1764  -m conntrack --ctstate NEW -j ACCEPT

> En este caso especifiqué que solo acepte conexiones desde ips de red local empezadas con `192.168`, pero se puede especificar el device con `-i` e `-o`. No recomendaría dejarlos habilitados a toda la red.

## Iniciamos el tray-icon

Finalmente, podemos iniciar `kdeconnect-indicator` para tener el tray icon, para que i3 lo haga automáticamente podemos agregar:

    exec --no-startup-id kdeconnect-indicator
