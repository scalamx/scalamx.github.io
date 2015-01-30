---
layout    : post
author    : "MontealegreLuis"
title     : "Funciones en Scala"
date      : 2015-01-29 18:10:38
categories: scala funciones lambda closures
---

# Funciones en Scala

En este post haremos un recorrido sobre el uso y definición de funciones en Scala. Mostraremos
con ejemplos:

* cómo definir funciones
* cómo crear funciones anónimas
* qué son y cómo se usan los closures
* qué son y cómo se usan las funciones de orden superior

## Definiendo funciones en Scala

En Scala las definiciones de funciones empiezan con la palabra reservada `def` seguidas del
identificador de la función, una lista de parámetros separados por comas y encerrados entre
paréntesis.

Scala  es un lenguage de tipado estático, así que debemos indicar el tipo de cada parámetro en
una función después de su nombre  y separado por `:`. Esto es debido a que Scala no infiere los
tipos de  los parámetros en las funciones.

Un ejemplo simple es una función que eleva al cuadrado un número entero.

{% highlight scala linenos %}
def square(x: Int): Int = {
  x * x
}
{% endhighlight %}

Después del nombre  y los parámetros debemos indicar el tipo de resultado que devuelve  una
función con una *anotación de tipo* ( `: Int` en nuestro ejemplo). El cuerpo de la función va
después del tipo de resultado, representado por un signo `=` seguido de corchetes.

El signo igual que precede al cuerpo hace referencia a la parte funcional del lenguage que indica
que una función define una expresión que resulta en un valor.

Un detalle que podemos señalar es que las funciones en Scala no requieren forzosamente de un
`return` como podemos observar en nuestro ejemplo. El resultado de la última expresión que se
evalúa en una función es interpretado de forma automática como el valor devuelto por dicha
función.

Un ejemplo donde debemos usar la palabra `return` es en el caso de las condicionales

{% highlight scala linenos %}
def min(a: Int, b: Int): Int = {
  if (a < b) {
    return a
  }
  b
}
{% endhighlight %}

En este ejemplo donde definimos una función `min`, si el valor de `a` es menor que `b` es
necesario usar un `return ` en la línea 3, ya que de otro modo, la función siempre devolvería el
valor de `b`.

Otro detalle de las funciones de Scala,  es que no es obligatorio especificar el tipo de resultado
de una función, ya que el compilador puede inferirlo.  Así nuestro ejemplo de función `square`
puede escribirse como:

{% highlight scala linenos %}
def square(x: Int) = {
  x * x
}
{% endhighlight %}

Hay dos excepciones para esta regla,  una es cuando usamos `return` en el cuerpo de la función.
Nuestra función `square` no puede entonces omitir el tipo devuelto.

La otra excepción ocurre para [funciones recursivas][1]. Un ejemplo de una función recursiva es
la implementación del [algoritmo de Euclides][2] para encontrar el máximo común divisor de dos
números. Por ejemplo:

{% highlight scala linenos %}
def gcd(a: Int, b: Int): Int = {
  if (b == 0) {
    a
  } else {
    gcd(b, a % b)
  }
}
{% endhighlight %}

La función `gcd` tampoco puede omitir el tipo de resultado.

Hay ocasiones en que necesitamos declarar funciones que no devuelven ningún valor interesante, por ejemplo:

{% highlight scala linenos %}
def greet() = {
  println("Hello, world!")
}
{% endhighlight %}

El tipo devuelto inferido por Scala en este ejemplo es `Unit`. El tipo `Unit` en Scala es similar al
`void` de Java.

Por último, si el cuerpo de una función puede escribirse en una solo línea, las llaves del cuerpo
pueden omitirse, así nuestras funciones `min`, `square`, `gcd` y `greet` pueden escribirse como:

{% highlight scala %}
def square(x: Int) =  x * x
def min(a: Int, b: Int) =  if (a < b) a else b
def gcd(a: Int, b: Int): Int = if (b == 0) a else gcd(b, a % b)
def greet() = println("Hello, world!")
{% endhighlight %}

## Funciones anónimas

Una función anónima, también llamada función literal o función [Lambda][3], es una definición
de una función que no está ligada a un identificador.

Scala tiene una sintaxis simple para definir funciones anónimas. La siguiente expresión crea una
función sucesor anónima para números enteros:

{% highlight scala linenos %}
(x: Int) => x + 1
{% endhighlight %}

El operador `=>` indica que la función convierte el argumento que pasemos del lado izquierdo
(cualquier número entero `x`) al resultado de evaluar la expresión que está del lado derecho (` x
+ 1` ). Por lo tanto, esta función mapea cualquier entero `x`  con `x + 1` (el sucesor de `x`).

Una función literal se compila como una *clase anónima* que cuando se instancia en tiempo de
ejecución se convierte en un *valor función*. De este modo, una función literal existe en el
código fuente, mientras que un valor función es un *objeto* en tiempo de ejecución. La
definición de clase para nuestra función anónima que genera el compilador de Scala es similar a
la siguiente:

{% highlight scala linenos %}
new Function1[Int, Int] {
  def apply(x: Int): Int = x + 1
}
{% endhighlight %}

Los valores función son objetos, así que podemos guardarlos en variables, pero como también
son funciones podemos invocarlos usando paréntesis.

{% highlight scala linenos %}
val successor = (x: Int) => x + 1
successor(10)
{% endhighlight %}

Una función anónima puede recibir cualquier numero de argumentos, por ejemplo:

{% highlight scala linenos %}
(x: Int, y: Int) => x + y
{% endhighlight %}

## Closures

Nuestro ejemplo de función Lambda, la función sucesor, es una función *completa*, ya que el
valor de  `x` es proporcionado como argumento a la función.

Pensemos ahora en la función:

{% highlight scala linenos %}
(x: Int) => x + y
{% endhighlight %}

En este caso, desde el punto de vista de la función, `y` es una variable *libre*, porque la función
literal, por sí sola, no le da un signifcado a `y`. La variable `x` por el contrario, es una variable
*ligada*, porque tiene significado en el  contexto de la función, está definida como su
parámetro. Por lo tanto, no será posible compilar esta función, si `y` no está dentro de su
alcance.

{% highlight scala linenos %}
val y = 3
val adder = (x: Int) => x + y
adder(4) // returns 7
{% endhighlight %}

Este ejemplo es posible, ya que al momento que se asigna el valor función `adder`, el valor se
*completa*, de ahí el término en inglés *closure* (conclusión o cierre). El compilador revisa
todas las variables en el alcance y asigna los valores necesarios al valor función (objeto) `adder`,
es decir, lo completa.

El valor función `adder` es a lo que se conoce como `closure` (conclusión) no a la declaración de
la función en sí. Una vez que el objeto (valor función) se concluye o completa, podemos moverlo
a otra parte de la aplicación donde `y` esté completamente fuera del alcance y aún así se referiría
al valor original que tenía en el alcance donde se completó el valor función.

## Funciones de orden superior

Uno de los usos más comunes de los closures es pasarlos como argumentos para funciones de
orden superior.  Una función de orden superior o *functor*, es una función que hace al menos
una de las siguientes cosas:

* Toma una o más funciones como entrada
* Devuelve una función

Por ejemplo, supongamos que tenemos una implementación del juego de [FizzBuzz][4] en la
siguiente función:

{% highlight scala linenos %}
def fizz_buzz() = {
  for (i <- 1 to 100) {
    if (i % 3 == 0) println("Fizz ")
    else if (i % 5 == 0) println("Buzz ")
    else println(i +  " ")
  }
}
{% endhighlight %}

Supongamos que queremos reescribirla para que funcione para cualquier variación del juego,
por ejemplo Fizz Buzz Woof, donde Woof corresponde a múltiplos de 7 y que además podamos
combinar palabras para cuando un número sea múltiplo de 3 y 5 o de 3 y 7 (FizzBuzz y FizzWoof
respectivamente). Si usamos closures podemos generalizar nuestro problema para cualquier
múltiplo que queramos agregar.

{% highlight scala linenos %}
def fizz_buzz() = {
  val lambdas = Array(
    (x: Int) => if (x % 3 == 0) "Fizz" else "",
    (x: Int) => if (x % 5 == 0) "Buzz" else "",
    (x: Int) => if (x % 7 == 0) "Woof" else ""
  )
  for (i <- 1 to 100) {
    println(next_value(lambdas, i))
  }
}
def next_value(lambdas: Array[(Int) => String], x: Int) = {
  var value = ""
  for (lambda <- lambdas) {
    value = value + lambda.apply(x)
  }
  if (value == "")  x.toString() else value
}
{% endhighlight %}

En este ejemplo la función `next_value` es una función de orden superior que recibe como
argumentos un arreglo de funciones del tipo `(Int) => String` que indica que cada elemento en el
arreglo debe ser una función que reciba como argumento un entero y que devuelva como
resultado una cadena (línea 11).

Dentro del método `next_value` lo que hacemos es aplicar cada una de las funciones del arreglo
`lambdas` a nuestro argumento `x` (`x` es uno de los valores del 1 al 100) usando el método
`apply`, esto es posible ya que como habíamos explicado antes los valores función son en
realidad objetos.

Así, dentro de la función `fizz_buzz` lo único que tenemos que hacer es generar el arreglo con las
funciones Lambda que reemplazan los múltiplos de un número por palabras. Así, si quisieramos
reemplazar los múltiplos de 11 por la palabra `Bazz` lo único que tenemos que hacer es agregar
una función más al arreglo `lambdas`.

Espero que este post te haya servido, si tienes alguna duda, queja, sugerencia o reclamación, no
dudes en buscarme en [Twitter][5]

[1]: http://en.wikipedia.org/wiki/Recursion_%28computer_science%29
[2]: http://en.wikipedia.org/wiki/Euclidean_algorithm
[3]: http://en.wikipedia.org/wiki/Lambda_calculus
[4]: http://en.wikipedia.org/wiki/Fizz_buzz
[5]: https://twitter.com/MontealegreLuis
