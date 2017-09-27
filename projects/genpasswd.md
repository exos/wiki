<!-- TITLE: Genpasswd - Generador de contraseñas seguras -->
<!-- SUBTITLE: CLI y con opciones determinísticas -->

`genpasswd` es una utilidad que desarrollé utilizando node.js y sirve para generar contraseñas fuertes, con la opción de hacerlas determinísticas.

# Instalación

Se puede instalar a través de [npm](https://www.npmjs.com/), se debe hacer con la opción `-g` de global:

    # npm install -g genpasswd
		
# Uso 

## CLI (como comando)

El uso es bastante simple, por defecto el comando genera una contraseña de 15 caracteres usando letras, números y símbolos:

    $ genpasswd 
    h8d=1Cs<;V}[Az/

Se pueden ver todas las opciones ejecutando:

    $ genpasswd --help

## Como librería (node.js)

Se puede usar dese node.js para eso solo hay que instalarla de forma local, y requerirla:

    $npm install --save genpasswd

Y

```JavaScript
const genPasswd = require('genpasswd');

let options = {
    type: genPasswd.password.TYPE_COMPLEX,
    passwordLength: 30
};

genPasswd.password.generate(options, function (err, password) {
    // password es un string
});
```

Se pueden usar todas las opciones pasadas por CLI a la librería como `options`.

# Tipos de passwords

Los tipos de passwords se pueden usar para generar claves en un formato específico, estas son:

    alpha: Alphabetics chars (only letters, from a to z)
    alphanumeric: Alphabetics chars with numbers
    numeric: Numbers
    symbols: Symbols like #, !, etc.
    complex: Use a mix of Alphanumerics and symbols
    custom: Yo can set what groups of chars and add chars with --chars

Esta lista se optiene usando:

    $ genpasswd --types

Por defecto se utiliza el tipo `complex`, pero se puede customizar seleccionado varios tipos:

    $ genpasswd -l 30 -t custom --numeric --symbols 
    :`:73(;%}6{=!;452?//~@$??7+1^=

Esto genera una password con solo número y símbolos, también se pueden espesificar caracteres y combinarlos:

    $ genpasswd -l 30 -t custom --chars abcDEF123
    1c2bcF2baaab2EF2F31aDFD32Ec31a
    $genpasswd -l 30 -t custom --chars abcDEF --numeric
    ac29b9863210F57Dc87Fc922ba92aa

# Contraseñas determinísticas
Se pueden generar contraseñas basadas en funciones determinísticas, basicamente basadas en 3 variables; *una password*, *un salt* y un número de *iteraciones*, por defecto las iteraciones están en `1000` *(Aunque puede variar en versiones futuras)*, pero la password y el salt son obligatorios.

Esto permite generar contraseñas realmente fuertes para diferentes servicios basados en dos o tres semillas fáciles de recordar.

Por ejemplo, si se necesita obtener una contraseña para una cuenta de email, se puede usar el nombre de la cuenta como password y una password **maestra** como salt (o biceversa):

    $ genpasswd -d -l 30
    genpass: phrasepass:  <- Tu cuenta de email, por ejemplo yourself@host.com
    genpass: salt:  <- Alguna clave que uses y te acuerdes (coldplay2012$) 
    fQz":Jib&7H,d}W6j+sA{)jnS~}u]3"
		
Cada vez que utilices el algoritmo determinístico para generar la clave, la contraseña será identica. Con esto se puden generar cada vez que la necesites y no tenerla almacenada. El algoritmo interno para hacer esto es [PBKDF2](https://en.wikipedia.org/wiki/Pbkdf2).

La ventaja de usar como contraseña (o salt) el nombre del servicio/cuenta, es que se puede generar una contraseña diferente para cada uno de estas.

# Proyecto

## Sitio web
El proyecto tiene su sitio web:

http://genpasswd.js.org

## Repositorio

Y si repositorio en GitHub:

https://github.com/exos/genpasswd

## Colaboraciones

* Reportar bugs e ideas en [el tracker](https://github.com/exos/genpasswd/issues)
* Hacer forks y enviar *pull requests* con resoluciones de errores o nuevos features
* Revisar mi ortografía (es horrible, también válido para la documentación en ingles)
 
 ## Donativos
 
 Se puede donar a través de Bitcoins hacia la villetera: `1E9A4Jg1tckJGD8rUx1WogBEL7uPXAktNP`