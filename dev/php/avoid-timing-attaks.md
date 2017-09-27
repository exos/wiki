<!-- TITLE: Evitar ataques de temporización o "Timing attacks" en PHP -->
<!-- SUBTITLE: Comparador seguro -->

> Originalmente posteado en [log/35](https://log.exos.ninja/35)

---

> Este artículo fue escrito originalmente a falta de una solución mas nativa, desde PHP 5.6 existe [`hash_equals()`](https://secure.php.net/manual/en/function.hash-equals.php)

---

Para los que no saben de que se trata, transcribo de la [esta web](http://docsetools.com/articulos-noticias-consejos/article_129177.html):

> En criptografía, un ataque de temporización es un ataque de canal lateral en el que el atacante intenta comprometer un criptosistema mediante el análisis del tiempo necesario para ejecutar los algoritmos criptográficos. Cada operación lógica en un equipo tarda en ejecutarse, y el tiempo puede variar en función de la entrada, con las medidas exactas del tiempo para cada operación, un atacante puede trabajar hacia atrás para la entrada.

En otras palabras, si por ejemplo, comparamos dos hashes como solemos hacer, el tiempo dependerá de cuándo frenemos la ejecución de ese método, por ejemplo:

```php
 function compare ($a, $b) {
		 if (strlen($a) !== strlen($b) {
					 return false;
		 }

		 for ($i = 0; $i < strlen($a); $i++) {
				 if ($a[$i] !== $b[i]) {
							return false;
				 }
				 sleep(1);
		 }

		 return true;

 }
```

En este caso ponemos el sleep para que sea obvio el ataque, y al hacerlo nos demos cuenta a simple vista.

Supongamos que tenemos que comparar *aaaaaaaaaa* con *aaaabaaaaa*:

```php
comparte("aaaaaaaaaa", "aaaabaaaaa");
```

En este caso, al llegar al **b**, la función se detendría y en caso de ser los primeros 4 caracteres correctos, la ejecución tardaría unos 4 segundos (recuerden que agregamos un sleep).

Los ataques de temporización se basan en esto para intentar, a través de diferencias de tiempos hasta que punto el la comparación es correcta.

Para evitar esto simplemente tenemos que tardar lo mismo tanto como si el primer carácter falla, a como el último.

Necesitaba hacer esto en PHP y cuando quise usar [hash_equals](https://php.net/manual/es/function.hash-equals.php) que es una función nativa me encontré con que está agregada recién en [PHP 5.6](http://log.exos.ninja/z), así que estaba por hacer una pero buscando en internet encontré una ya hecha, aparentemente parte de alguna clase de ZendFramework:

```php
/**
 * Securely compare two strings for equality while avoided C level memcmp()
 * optimisations capable of leaking timing information useful to an attacker
 * attempting to iteratively guess the unknown string (e.g. password) being
 * compared against.
 *
 * @param string $a
 * @param string $b
 * @return bool
 */
protected function _secureStringCompare($a, $b)
{
		if (strlen($a) !== strlen($b)) {
				return false;
		}
		$result = 0;
		for ($i = 0; $i < strlen($a); $i++) {
				$result |= ord($a[$i]) ^ ord($b[$i]);
		}
		return $result == 0;
}
```

Espero que les sirve, y si siguen comparando hashes carácter por carácter, espero que hayan tomado conciencia de esto :-).