---
title: numericCast(_:)
author: Mattt
translator: Leo Picado
category: Swift
excerpt: >
  Lograr que el código compile es distinto a hacer las cosas bien, pero a veces necesitamos lo primero para llegar a lo
   segundo.
status:
  swift: 4.2
---

Todos tenemos nuestra analogía preferida para describir la programación, desde carpintería, bordado o hasta jardinería. Quizás lo reducimos a resolver problemas, contar una historia o quizás solo como un arte.

Programar es muy similar a escribir, la pregunta es ¿es prosa o poesía? ¿Y si fuera música? ¿Qué género sería? Sería jazz, sin duda alguna.

Tal vez los cuentos del medio oeste brinden la comparación más cercana sobre lo que hacemos a diario. Al ojear cualquier edición de _Las mil y una noches_ (<bdi lang="ar">أَلْف لَيْلَة وَلَيْلَة</bdi>) vemos descripciones de entes sobrenaturales conocidos como <dfn>geniecillos</dfn>, <dfn>genios</dfn> o 🧞. Sin importar cómo se les llame, todos estamos familiarizados con su habilidad de conceder deseos y con las tragedias que siempre causan.

Podríamos decir que, de muchas maneras, las computadoras representan una constitución física de un deseo. Una computadora, al igual que un genio, felizmente cumplirá cualquier deseo solicitado, sin miramiento alguno y para cuando nos demos cuenta que se cometió un error, puede ser demasiado tarde para solucionarlo.

Como desarrolladores en Swift es muy probable que en algún momento hayamos caído presa de errores al intentar convertir tipos enteros y que también, hayamos deseado que los errores desaparecieran y el código compilara.

Si eso le suena familiar recibirás con agrado el tema de esta semana, se trata de `numericCast(_:)`, una pequeña función incluida en la librería estándar del lenguaje que puede ser justamente lo que tanto deseabas. Sin embargo siempre debemos ser cuidadosos con lo que deseamos, porque podría convertirse en realidad.

---

Desmitifiquemos a `numericCast(_:)` [viendo su implementación](https://github.com/apple/swift/blob/7f7b4f12d3138c5c259547c49c3b41415cd4206e/stdlib/public/core/Integers.swift#L3508-L3510):

```swift
public func numericCast<T : BinaryInteger, U : BinaryInteger>(_ x: T) -> U {
  return U(x)
}
```

(Tal y como aprendimos en [nuestro artículo sobre Never](https://nshipster.es/never/), hasta la cantidad más pequeña de Swift puede tener un gran impacto).

El protocolo [`BinaryInteger`](https://developer.apple.com/documentation/swift/binaryinteger) fue introducido en Swift 4 como parte de un esfuerzo para unificar la interfaz y el manejo de enteros, tanto con y sin signo, sin importar su talla ó tamaño.

Cuando se convierte un entero a otro tipo, es posible que el valor no pueda representarse por ese tipo, esto sucede cuando se intenta convertir un entero con signo a un tipo sin signo (por ejemplo `-42` hacia `UInt`) o cuando el valor excede el rango representable del tipo de destino (por ejemplo `UInt8` solo puede representar números entre `0` y `255`).

`BinaryInteger` define cuatro estrategias para conversiones entre tipos de entero, cada una de ellas con distinto comportamiento para manejar valores fuera de rango:

- **Conversión con chequeo de rango**
  ([`init(_:)`](https://developer.apple.com/documentation/swift/binaryinteger/2885704-init)):
  Dispara un error en tiempo de ejecución para valores fuera de rango
- **Conversión exacta**
  ([`init?(exactly:)`](https://developer.apple.com/documentation/swift/binaryinteger/2925955-init)):
  Devuelve `nil` para valores fuera de rango
- **Conversión de prensa**
  ([`init(clamping:)`](https://developer.apple.com/documentation/swift/binaryinteger/2886143-init)):
  Usa el valor más cercano para representar valores fuera de rango
- **Conversión de patrón de bits**
  ([`init(truncatingIfNeeded:)`](https://developer.apple.com/documentation/swift/binaryinteger/2925529-init)):
  Trunca el ancho del tipo destino 

La estrategia correcta la dicta el contexto, en algunas ocasiones es preferible prensar los valores a un rango que pueda ser representado, en otras es mejor no tener un valor del todo. En el caso de `numericCast(_:)`, el chequeo de rango se usa por conveniencia. La desventaja de este enfoque es que al invocar esta función con valores fuera de rango produce un error en tiempo de ejecución. Específicamente, produce un `trap` en `overflow` para `-O` y `-Onone`.

{% info %}

Para profundizar en los cambios en la forma que los números funcionan en Swift se recomienda consultar [SE-0104: "Protocol-oriented integers"](https://github.com/apple/swift-evolution/blob/master/proposals/0104-improved-integers.md).

Este mismo tema se cubre a fondo en [Flight School Guide to Numbers](https://gumroad.com/l/swift-numbers).

{% endinfo %}

## Pensando literal y críticamente

Antes de seguir, tomemos un momento para discutir enteros literales.

Hemos [discutido en artículos anteriores](https://nshipster.com/swift-literals/) sobre las distintas maneras, tanto convenientes como extensibles, que nos brinda Swift para representar valores en nuestro código.

Esto, combinado con la inferencia de tipos provista por el lenguaje, la mayoría del tiempo, las cosas "simplemente funcionan", lo que es maravilloso, excepto en las confusas ocasiones en que "simplemente no funcionan".

Usemos el siguiente ejemplo de arrays, con y sin signo, que contienen idénticos valores literales:

```swift
let arrayOfInt: [Int] = [1, 2, 3]
let arrayOfUInt: [UInt] = [1, 2, 3]
```

A pesar de su equivalente apariencia, no podemos hacer esto:

```swift
arrayOfInt as [UInt] // Error: Cannot convert value of type '[Int]' to type '[UInt]' in coercion
```

Una forma de solventar este problema sería pasar la función `numericCast` como parámetro a `map(_:)`:

```swift
arrayOfInt.map(numericCast) as [UInt]
```

Esto equivale a invocar directamente el inicializador de `UInt` con chequeo de rango: 

```swift
arrayOfInt.map(UInt.init)
```

Volvamos a nuestro ejemplo pero usando valores ligeramente distintos esta vez:

```swift
let arrayOfNegativeInt: [Int] = [-1, -2, -3]
arrayOfNegativeInt.map(numericCast) as [UInt] // 🧞‍ Fatal error: Negative value is not representable
```

Asemejando una funcionalidad en tiempo de ejecución a otra en tiempo de compilación, `numericCast (_ :)` está más cerca de `as!` que `as` o` as? `.

Comparemos este comportamiento a lo que sucede si usamos el inicializador de conversión exacta `init?(exactly:)`:

```swift
let arrayOfNegativeInt: [Int] = [-1, -2, -3]
arrayOfNegativeInt.map(UInt.init(exactly:)) // [nil, nil, nil]
```

`numericCast(_:)`, al igual que su conversión con chequeo de rango, es una herramienta contundente, por lo que es importante entender las concesiones que se hacen al usarla.

## El costo de tener la razón

Generalmente en Swift se recomienda usar `Int` para enteros (y `Double` para valores de punto flotante) a menos que exista una *muy* buena razón para hacerlo.

La propiedad `count` de una ``Collection`` nunca será, por definición, un valor negativo, sin embargo se usa un `Int` en vez de un `UInt` por que el costo de los viajes ida y vuelta entre tipos al interactuar con otras `API`s es más alto que el uso de un tipo más preciso.

Por la misma razón, casi siempre es mejor usar `Int` para representar números pequeños, como [días de la semana](https://nshipster.com/datecomponents/), a pesar que cualquier otro valor estaría contenido, con espacio de sobra, en un tipo de 8 bits.

El caso de uso perfecto para justificar esta práctica es usar Swift para interactuar con una `API` en C.

Algunas `API`s más antiguas y de más bajo nivel están compuestas de selecciones de tipos basados tanto en la arquitectura sobre la que se ejecutan y un austero uso del espacio.

Encima de todos los problemas que trae la interoperabilidad, como encabezados y punteros, todo esto puede ser un punto de quiebre para algunos.

`numericCast(_:)` existe para los momentos en que estamos cansados de ver rojo en nuestro código y solo queremos que las cosas compilen.

## Actos aleatorios de compilación

El [ejemplo en la documentación oficial](https://developer.apple.com/documentation/swift/2884564-numericcast) podría sernos familiar, antes de [SE-0202](https://github.com/apple/swift-evolution/blob/master/proposals/0202-random-unification.md) para generar números aleatorios en Swift solíamos importar el framework `Darwin` e invocar la función `arc4random_uniform(3)`:

```c
uint32_t arc4random_uniform(uint32_t __upper_bound)
```

El problema es que para usar `arc4random` en Swift, ocupamos no una, sino dos conversiones de tipo: la primera para el límite superior (`Int` → `UInt32`) y la segunda para el valor que deseamos obtener (`UInt32` → `Int`):

```swift
import Darwin

func random(in range: Range<Int>) -> Int {
    return Int(arc4random_uniform(UInt32(range.count))) + range.lowerBound
}
```

_Asqueroso._

Mediante el uso de `numericCast(_:)` podemos mejorar la lectura al costo de un método un poco más largo.

```swift
import Darwin

func random(in range: Range<Int>) -> Int {
    return numericCast(arc4random_uniform(numericCast(range.count))) + range.lowerBound
}
```

Podemos ver que `numericCast(_:)` no está realizando ninguna tarea que no fuese posible mediante el correcto inicializador de cada tipo. Al contrario, está siendo usado como un indicador de la conversión, es efectivamente, la cantidad mínima de código para compilar.

Pero como hemos aprendido en nuestras aventuras con los genios, debemos tener cuidado al formular nuestros deseos.

Inspeccionando más a fondo nos damos cuenta que el uso de `numericCast(_:)` tiene una falla crítica, _!genera un `trap` con valores que exceden a `UInt32.Max`!_

```swift
random(in: 0..<0x1_0000_0000) // 🧞‍ Fatal error: Not enough bits to represent the passed value
```

Si observamos la [implementación de la librería estándar](https://github.com/apple/swift/blob/7f7b4f12d3138c5c259547c49c3b41415cd4206e/stdlib/public/core/Integers.swift#L2537-L2560), la que nos permite usar `Int.random(in: 0...10)`, veremos que usa una prensa, en vez una conversión de chequeo de rango. También vemos que en vez de delegar su trabajo a un método como `arc4random_uniform`, [llena una serie de valores desde un buffer hacia bytes aleatorios](https://github.com/apple/swift/blob/7f7b4f12d3138c5c259547c49c3b41415cd4206e/stdlib/public/core/Random.swift#L156-L177).

---

Lograr que el código compile es distinto a hacer las cosas bien, pero a veces ocupamos lo primero para llegar a lo segundo. Cuando se usa cuidadosamente `numericCast(_:)` es una  herramienta útil para resolver problemas rápidamente.

Un segundo beneficio es el de indicarnos posibles comportamientos erróneos más claramente que al usar inicializadores convencionales.

Al fin y al cabo, la programación no es otra cosa que describir de manera exacta lo que queremos. Normalmente con un nivel de detalle obsesivo.

No hay una instrucción para el CPU que sea equivalente para "haz lo correcto" (y ¿aunque lo existiera, [la usaríamos](https://github.com/FixIssue/FixCode)?). Afortunadamente Swift nos permite hacerlo de una manera segura y más concisa que en muchos otros lenguajes y, honestamente, ¿qué más podríamos desear?

