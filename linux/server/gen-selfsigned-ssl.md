<!-- TITLE: Generar certificado SSL autofirmado con un solo comando -->

> Posteado originalmente en [log/R](https://log.exos.ninja/R)

---

    # openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /ruta/de/llave-privada.key -out /ruta/de/certificado.crt

> Robado de acá: https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04