<!-- TITLE: React Native -->
<!-- SUBTITLE: Alguanas memorias sobre react-native -->

Después de laburar con [cordova](https://cordova.apache.org/) y [ionic](https://ionicframework.com/) decidí pasarme a [react-native](https://facebook.github.io/react-native/) definitivamente, ya que en el momento en que agarré ionic no lo veía lo suficientemente maduro. Al igual que estas tecnologías, react-native permite crear aplicaciones mobile multiplataforma utilizando JavaScript, pero con la diferencia de utilizar interfaces nativas, algo que mejora muchisimo el rendimiento. También tiene de ventaja que incorpora [webpack](https://webpack.js.org/), y buenas herramientas de debug.

# Desventuras

## Versiones estables

Si bien react-native y react son productos de Facebook, la última versión estable hasta la fecha de ract-native `~0.44.0` depende de react `16.0.0-alpha.6`, una version ALPHA!!. Para resolver esto, descubrí gracias a [un post en StackOverflow](https://stackoverflow.com/questions/43937866/how-to-install-react-native-with-react-stable-version) la mejor convinación de versiones, que son, `0.42.0` para react-native y `~15.4.1` para react.

### react-native-splash-screen

En la versión `3.0.0` de [crazycodeboy/react-native-splash-screen](https://github.com/crazycodeboy/react-native-splash-screen) depreca react-native inferior a `0.47.0`, así que si usamos la versión `0.42.0`, deberemos utilizar la rama `2.1.0`. Como lo dice en el README del autor:

> For React Native >= 0.47.0 use v3.+, for React Native < 0.47.0 use v2.1.0

Para eso instalamos:

    $ npm install --save react-native-splash-screen@2.1.0

o

    $ yarn add react-native-splash-screen@2.1.0

Luego seguir todos los pasos, estando atentos a que estamos instalando la `2.1.0`.

## Navigation

Otra de las grandes deprecaciones de react-native, desde la `0.44.0` deprecan el componente `Navigator`, con lo que recomiendan usar [react-navigation](https://reactnavigation.org), así que utilizar este componente, aún utilizando react `0.42.0` sería una mala práctica. Yo recomiendo, no solo empezar con **react-navigation** como dice la [documentación de react-native](https://facebook.github.io/react-native/docs/navigation.html), sino hacer un componente de navegación propio, así en futuras deprecaciones se puede adaptar a otra lib.

Otra solución un poco menos recomendada, si se necesita si o si migrar una app a nuevas versiones de **react-native** y no hay tiempo, es utilizar el paquete `react-native-deprecated-custom-components` como se ve en [este post de stack overflow](https://stackoverflow.com/questions/44328243/react-native-navigator-is-deprecated-and-has-been-removed-from-this-package#45506347).

# Tips

## Cambiando package id en android

Por defecto cuando se crea un nuevo proyecto, se genera un id para la aplicación de android llamada `com.{nombreapp}`, el tema es que si queremos tener un id mas descriptivo, al estilo `ninja.exos.{nombreapp}` hay que hacer algunas cositas:

### Mover los archivos fuentes de android

Hay dos archivos que son el código de nuestro proyecto android, `MainActivity.java` y `MainApplication.java`, debemos verlos, suponiendo que nuestra aplicación se llame {NombreApp}, el directorio sería `android/app/src/main/java/com/nombreapp`, para eso creamos el arbol de directorios de nuestro id:

    $ mkdir android/app/src/main/java/ninja/exos
		
Y movemos los archivos:
	
	    $ mv android/app/src/main/java/com/nombreapp android/app/src/main/java/ninja/exos
			$ rm -rf android/app/src/main/java/com

### Cambiando el ID de los archivos fuentes y de build

Luego deberemos cambiar el ID de varios lugares, para lo que recomiendo hacer directamente:

    $ find android -type f -exec sed -i "s/com\.nombreapp/ninja.exos.nombreapp" {} \;
		
### Limpiar la cache

Por último limpiamos la cache

    $ cd android && ./gradlew clean
		
### Fuentes

* https://stackoverflow.com/questions/43937866/how-to-install-react-native-with-react-stable-version