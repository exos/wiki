<!-- TITLE: Consultar IP de salida con dig -->
<!-- SUBTITLE: Rápido y en una sola linea -->

> Posteado originalmente en [log/H](https://log.exos.ninja/H)

---

Antes usaba el buen servicio [ifconfig.me](http://ifconfig.me) que al ser consultado con ```curl``` devuelve una salida limpia como para usar en la consola. Útil para scripts:

    $ curl http://ifconfig.me

Pero me pasaron un método mas rápido y limpio que hace uso del comando ```dig```:

    $ dig +short myip.opendns.com @resolver1.opendns.com

Si bien es mas difícil de recordar, la velocidad es asombrosamente mas rápida:

    curl http://ifconfig.me  0,00s user 0,01s system 0% cpu 4,638 total
    dig +short myip.opendns.com @resolver1.opendns.com  0,01s user 0,01s system 3% cpu 0,314 total
