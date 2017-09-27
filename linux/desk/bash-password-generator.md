<!-- TITLE: Generador de passwords fuertes en bash -->
<!-- SUBTITLE: Utilizando /dev/random -->

> Originalmente posteado en [log/37](https://log.exos.ninja/37)

---

> Para tener un software generador de contraseñas fuertes y con varias opciones, podés hecharle un vistazo a [genpasswd](projects/genpasswd).

---

Nada siempre que configuro servers me gusta ponerle una contraseña fuerte, si después el usuario la cambia por una mas devil ya no es mi culpa, pero bueno, para hacerlo desde la consola me hice un pequeño script en bash:

```bash
#!/usr/bin/env bash

CANT=40

if [ "x$1" != "x" ]; then
		if [ $(echo $1 | grep -E ^[0-9]+$ | wc -l ) -eq 1  ]; then
				CANT=$1
		else
				echo "Firt argument need to be a number"
				exit 1
		fi
fi

PASS=$(dd if=/dev/random bs=1 count=$CANT 2> /dev/null | base64 -w0) 

echo $PASS | cut -b 1-$CANT 

exit 0
```

En fin no es la gran cosa, pero es bastante útil, para usarlo:

    $ getpass
    CBYLF1osdNiajfNb/H5gNwfAdrRVqdqfLpU7bW9O

Eso nos devuelve una password al azar de 40 caracteres. Si le mandamos un número como argumento, nos devolverá una contraseña de esa longitud.

    $ getpass 10
    oQjCLfuiJr