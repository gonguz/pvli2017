# Ejercicios de laboratorio

Probad estos ejemplos y tratad de responder a las preguntas. Si os atascáis con
lo que hace una función, buscad en Internet la función acompañado de "mdn".

**1. Valores booleanos en JavaScript.**

En JavaScript cualquier valor puede considerarse verdadero o falso según el
contexto. Por ejemplo, el `0` es considerado `false` y cualquier número distinto
de `0` es `true`:

```js
var v = 0; // después de esta prueba, cambia el valor de v por otro número.
!!v; // la doble negación al comienzo lo convierte en booleano.
```

Descubre qué valores son ciertos y cuales falsos para todos los tipos: números,
cadenas, objetos, funciones y undefined.

**2. Expresiones booleanas en JavaScript.**

En JavaScript, las expresiones booleanas son _vagas_, esto significa que en
cuanto sabemos lo que va a valer la expresión, dejamos de evaluar. Por ejemplo,
¿qué crees que le pasará a la siguiente expressión?

```js
var hero = { name: 'Link', weapon: null };
console.log('Hero weapon power is:', hero.weapon.power);
```

Pero, ¿y ahora?

```js
var hero = { name: 'Link', weapon: null };
if (hero.weapon && hero.weapon.power) {
  console.log('Hero weapon power is:', hero.weapon.power);
} else {
  console.log('The hero has no weapon.');
}
```

En caso de expresiones `&&` (_and_ o _y_), la evaluación termina tan pronto como
encontramos un término falso.

En caso de expresiones `||` (_or_ u _o_), la evaluación termina tan pronto como
encontramos un término verdadero.

**3. El resultado de las expresiones booleanas.**

Contra el sentido común, el resultado de una expresión booleana no es un
booleano sino el último término evaluado. Recuerda que la evaluación es _vaga_
y dejamos de evaluar tan pronto como podemos determinar el resultado de la
expresión. Con esto en cuenta, trata de predecir el resultado de las siguientes
expresiones:

```js
var v;
function noop() { return; };

1 && true && { name: 'Link' };
[] && null && "Spam!";
null || v || noop || true;
null || v || void "Eggs!" || 0;
```

**4. Parámetros por defecto.**

Puedes ver una aplicación real de lo anterior en esta función para rellenar
números. En JavaScript no hay parámetros por defecto, pero los parámetros
omitidos tienen el valor especial `undefined` que es falso.

```js
function pad(target, targetLength, fill) {
  var result = target.toString();
  var targetLength = targetLength || result.length + 1;
  var fill = fill || '0';
  while (result.length < targetLength) {
    result = fill + result;
  }
  return result;
}

// intenta predecir el resultado de las siguientes llamadas
pad(3);
pad(2, 5);
pad(2, 5, '*');
```

**5. Buenas prácticas en el diseño de APIs.**

Hemos dicho muchas veces que el estado no se debería exponer pero siempre
acabamos enseñando este tipo de modelado para los puntos:

```js
var p = { x: 5, y: 5 };

function scale(point, factor) {
  point.x = point.x * factor;
  point.y = point.y * factor;
  return p;
}

scale(p, 10);
```

La implementación correcta sería:

```js
var p = {
  _x: 5,
  _y: 5,
  getX: function () {
    return this._x;
  },
  getY: function () {
    return this._y;
  },
  setX: function (v) {
    this._x = v;
  },
  setY: function (v) {
    this._y = v;
  }
};

function scale(point, factor) {
  point.setX(point.getX() * factor);
  point.setY(point.getY() * factor);
  return p;
}

scale(p, 10);
```

Pero reconozcámoslo, escribir esto es un rollo soberano.

**6. Propiedades computadas al rescate.**

JavaScript permite definir un tipo especial de propiedades llamadas normalmente
_propiedades computadas_ de esta guisa:

```js
var p = {
  _x: 5,
  _y: 5,
  get x() {
    return this._x;
  },
  get y() {
    return this._y;
  },
  set x(v) {
    this._x = v;
  },
  set y(v) {
    this._y = v;
  }
};

function scale(point, factor) {
  point.x = point.x * factor;
  point.y = point.y * factor;
  return p;
}

scale(p, 10);
```

Escribirlo sigue siendo un muermo (menos mal que estudiaremos como hacer
factorías de objetos) pero utilizarlo es mucho más claro. Así, si ahora decides
que sería mejor exponer el nombre de los ejes en mayúscula, puedes hacer:

```js
var p = {
  _x: 5,
  _y: 5,
  get X() {
    return this._x;
  },
  get Y() {
    return this._y;
  },
  set X(v) {
    this._x = v;
  },
  set Y(v) {
    this._y = v;
  }
};

function scale(point, factor) {
  point.X = point.X * factor;
  point.Y = point.Y * factor;
  return p;
}

scale(p, 10);
```

¿Se te ocurre la manera de hacer que una propiedad pueda ser de sólo lectura? Es
decir, que su valor no pueda cambiarse (asumiendo que el usuario no accederá a
las propiedades que comiencen por '_').

Si quisieras añadir una propiedad a un objeto ya existente tendrías que utilizar
[Object.defineProperty()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty):

```js
var point = {};
Object.defineProperty(point, '_x', { value: 5 });
Object.defineProperty(point, '_y', { value: 5 });
Object.defineProperty(point, 'x', {
  get: function () {
    return this._x;
  },
  set: function (v) {
    this._x = v;
  }
});
Object.defineProperty(point, 'y', {
  get: function () {
    return this._y;
  },
  set: function (v) {
    this._y = v;
  }
});
point; // no se observan propiedades...
point.x; // ...pero aquí están.
point.y;
```

¿Te atreves a decir por qué cuando inspeccionamos el objeto no aparecen sus
propiedades? ¿Cómo podrías arreglarlo? ¿Cómo harías para que sólo se vieran
las propiedades que son parte de la API?

No te lances a usar `Object.defineProperty()` si no tienes **muy claro** qué
significan los términos **configurable**, **enumerable** y **writable**.

**7. Usando funciones como si fueran métodos.**

Hemos visto que cualquier función puede usarse como un método si se referencia
como una propiedad de un objeto y entonces se llama. Pero lo cierto es que
también podemos hacer que una función cualquiera, sin estar referenciada desde
una propiedad, pueda ser usada como el método de un objeto si indicamos
explícitamente cual es el objeto destinatario. Esto puede hacerse con
[`.apply()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)
y con
[`.call()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call).

```js
var ship = { name: 'Death Star' };

function fire(shot) {
  console.log(this.name + ' is firing: ' + shot.toUpperCase() + '!!!');
}

ship.fire; // ¿qué crees que será esto?
fire.apply(ship, ['pichium']);
fire.call(ship, 'pañum');
```

¿Cuál es la diferencia entre `.apply()` y `.call()`?

**8. Propiedades dinámicas.**

La notación corchete para acceder a las propiedades de un objeto es
especialmente útil para acceder a propiedades de manera genérica. Por ejemplo,
imagina el siguiente código:

```js
var hero = {
  name: 'Link',
  hp: 10,
  stamina: 10,
  weapon: { name: 'sword', effect: { hp: -2 } }
};
var enemy = {
  name: 'Ganondorf',
  hp: 20,
  stamina: 5,
  weapon: { name: 'wand', effect: { hp: -1, stamina: -5 } }
};

function attack(character, target) {
  if (character.stamina > 0) {
    console.log(character.name + ' uses ' + character.weapon.name + '!');
    applyEffect(character.weapon.effect, target);
    character.stamina--;
  } else {
    console.log(character.name + ' is too tired to attack!');
  }
}

function applyEffect(effect, target) {
  // Obtiene los nombres de las propiedades del objeto. Búscalo en la MDN.
  var propertyNames = Object.keys(effect);
  for (var i = 0; i < propertyNames.length; i++) {
    var name = propertyNames[i];
    target[name] += effect[name];
  }
}

attack(hero, enemy);
attack(enemy, hero);
attack(hero, enemy);
attack(enemy, hero);
attack(hero, enemy);
```

¿Podrías modificar el efecto del arma del héroe para incapacitar al enemigo pero
no matarlo ni dañarlo? Intenta hacerlo sin reescribir el ejemplo entero.

**9. Objetos como algo más que objetos.**

Los objetos de JavaScript no solo sirven para modelar los objetos de la
programación orientada a objetos sino que permiten realizar clasificaciones por
nombre. Un histograma, es decir un conteo de un conjunto con repeticiones, es
un ejemplo clásico de la utilidad de un objeto JavaScript:

```js
function wordHistogram(text) {
  var wordList = text.split(' ');
  var histogram = {};
  for (var i = 0; i < wordList.length; i++) {
    var word = wordList[i];
    if (!histogram.hasOwnProperty(word)) {
      histogram[word] = 0;
    }
    histogram[word]++;
  }
  return histogram;
}
```

Prueba a usar la función por ti mismo.

Lo que JavaScript llama objetos se conoce en otros lenguajes de programación
como mapas o diccionarios y a los nombres de las propiedades se los llama
_claves_.

¿Puedes pensar en al menos una aplicacion más?

**10. Funciones como parámetros.**

Las listas de JavaScript tiene algunos métodos que aceptan funciones como
parámetros, por ejemplo
[`.forEach()`](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Array/forEach).
De hecho es común encontrar `.forEach()` cuando se tiene la certeza de que se
van a recorrer **todos** los elementos de una lista.

```js
function wordHistogram(text) {
  var wordList = text.split(' ');
  var histogram = {};
  wordList.forEach(function (word) {
    if (!histogram.hasOwnProperty(word)) {
      histogram[word] = 0;
    }
    histogram[word]++;
  });
  return histogram;
}

var poem = 'Todo pasa y todo queda, ' +
           'pero lo nuestro es pasar, ' +
           'pasar haciendo caminos, ' +
           'caminos sobre la mar';

wordHistogram(poem);
```

El resultado no es correcto porque al separar las palabras por los espacios
estamos dejando caracteres que no son palabras como parte de ellas. Podemos
arreglarlo si en vez de partir el texto por los espacios usamos una
[expresión regular](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions)
para partir el texto por los límites de las palabras:

```js
function wordHistogram(text) {
  var wordList = text.split(/\b/); // Eso entre / / es una expresión regular.
  var histogram = {};
  wordList.forEach(function (word) {
    if (!histogram.hasOwnProperty(word)) {
      histogram[word] = 0;
    }
    histogram[word]++;
  });
  return histogram;
}

var poem = 'Todo pasa y todo queda, ' +
           'pero lo nuestro es pasar, ' +
           'pasar haciendo caminos, ' +
           'caminos sobre la mar';

wordHistogram(poem);
```

Pero ahora tenemos cosas que no son palabras (como espacios y comas). Podemos
filtrar una lista con
[`.filter()`](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Array/filter):

```js
function isEven(n) { return n % 2 === 0; }
[1, 2, 3, 4, 5, 6].filter(isEven);
```

Y así quitar lo que no sean palabras:

```js
function isWord(candidate) {
  return /\w+/.test(candidate);
}

function wordHistogram(text) {
  var wordList = text.split(/\b/);
  wordList = wordList.filter(isWord);
  var histogram = {};

  wordList.forEach(function (word) {
    if (!histogram.hasOwnProperty(word)) {
      histogram[word] = 0;
    }
    histogram[word]++;
  });
  return histogram;
}

var poem = 'Todo pasa y todo queda, ' +
           'pero lo nuestro es pasar, ' +
           'pasar haciendo caminos, ' +
           'caminos sobre la mar';

wordHistogram(poem);
```

También deberíamos normalizar las palabras (pasarlas a minúsculas por ejemplo)
para no encontrarnos con entradas distintas en el histograma para la misma
palabra. Para transformar una lista en otra lista de los mismos elementos,
usamos
[`.map()`](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Array/map).

```js
function isWord(candidate) {
  return /\w+/.test(candidate);
}

function toLowerCase(word) {
  return word.toLowerCase();
}

function wordHistogram(text) {
  var wordList = text.split(/\b/);
  wordList = wordList.filter(isWord);
  wordList = wordList.map(toLowerCase);
  var histogram = {};

  wordList.forEach(function (word) {
    if (!histogram.hasOwnProperty(word)) {
      histogram[word] = 0;
    }
    histogram[word]++;
  });
  return histogram;
}

var poem = 'Todo pasa y todo queda, ' +
           'pero lo nuestro es pasar, ' +
           'pasar haciendo caminos, ' +
           'caminos sobre la mar';

wordHistogram(poem);
```

Una última función nos permite transformar una lista en un sólo valor. Esto es
precisamente el histograma, una clasificación de todos los valores de la lista.
Esta transformación se consigue mediante
[`.reduce()`](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Array/reduce):

```js
function isWord(candidate) {
  return /\w+/.test(candidate);
}

function toLowerCase(word) {
  return word.toLowerCase();
}

function buildHistogram(inProgressHistogram, word) {
  if (!inProgressHistogram.hasOwnProperty(word)) {
    inProgressHistogram[word] = 0;
  }
  inProgressHistogram[word]++;
  return inProgressHistogram;
}

function wordHistogram(text) {
  var emptyHistogram = {};
  return text.split(/\b/)
             .filter(isWord)
             .map(toLowerCase)
             .reduce(buildHistogram, emptyHistogram);
}

var poem = 'Todo pasa y todo queda, ' +
           'pero lo nuestro es pasar, ' +
           'pasar haciendo caminos, ' +
           'caminos sobre la mar';

wordHistogram(poem);
```
