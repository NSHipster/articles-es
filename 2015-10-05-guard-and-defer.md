---
title: guard & defer
author: Mattt & Nate Cook
authors:
    - Nate Cook
    - Mattt
translator: Jhonny Bill Mena
category: Swift
excerpt: >
    Swift 2.0 introdujo dos nuevas instrucciones de control 
    buscando simplificar los programas que escribimos.
    El primero, por su naturaleza, hace nuestro código más lineal,
    mientras que el último hace lo opuesto, retrasando la ejecución
    de su contenido.
revisions:
    "2015-10-05": First Publication
    "2018-08-01": Updated for Swift 4.2
status:
    swift: 4.2
    reviewed: August 1, 2018
---

> "Como programadores conscientes de nuestras propias limitaciones, 
> debemos intentar que la correspondencia 
> entre el programa (esparcido en el espacio del texto)
> y el proceso (tiempo) sea lo más sencilla posible."

> —[Edsger W. Dijkstra](https://es.wikipedia.org/wiki/Edsger_Dijkstra),
> ["Go To Considered Harmful"](https://homepages.cwi.nl/~storm/teaching/reader/Dijkstra68.pdf)

Es una lástima que su ensayo sea aún más recordado por popularizar el meme "\_\_\_\_ Consider Harmful"
entre los programadores y sus mal consideradas diatribas en línea.
Porque (como usualmente) Dijkstra estaba formulando un excelente punto:
**la estructura del código debe reflejar su comportamiento**.

Swift 2.0 introdujo dos nuevas instrucciones de control buscando simplificar los programas que escribimos:
`guard` y `defer`.
Por su naturaleza, el primero hace nuestro código más lineal, mientras que el último hace lo opuesto, retrasando la ejecución de su contenido.

¿Cómo deberíamos abordar estas instrucciones de control?
¿Cómo pueden `guard` y `defer` ayudarnos a aclarar la correspondencia entre el programa y el proceso?

Vamos a diferir `defer` y empecemos hablando de `guard`.

---

## guard

`guard` es una instrucción condicional que requiere una expresión que resulte `true` para continuar la ejecución del código.
Si la expresión resulta `false`, se ejecutará la cláusula obligatoria `else`.

```swift
func sayHello(numberOfTimes: Int) {
    guard numberOfTimes > 0 else {
        return
    }

    for _ in 1...numberOfTimes {
        print("Hola!")
    }
}
```

La cláusula `else` en la instrucción `guard` debe salir del alcance actual utilizando `return` para abandonar la función, `continue` o `break` para salir de un bucle 
o una función que retorna [`Never`](https://nshipster.es/never), como `fatalError(_:file:line:)`.

Las instrucciones `guard` son muy útiles cuando se combinan con optional bindings.
Cuando se crea un nuevo optional binding en la condición de la instrucción `guard`, este está disponible para el resto de la función o el bloque.

Compara cómo funciona optional binding en una instrucción `guard-let` con una instrucción `if-let`. 

```swift
var name: String?

if let name = name {
    // Aquí dentro `name` no es opcional (name is String)
}
// Aquí fuera `name` es opcional (name is String?)

guard let name = name else {
    return
}

// De ahora en adelante `name` no es opcional (name is String)
```

Si la sintáxis de múltiple optional bindings introducida en
[Swift 1.2](https://nshipster.com/swift-1.2/)
anunció la renovación de
[la pirámide de la condena](http://www.scottlogic.com/blog/2014/12/08/swift-optional-pyramids-of-doom.html),
la instrucción `guard` la derriba completamente.

```swift
for imageName in imageNamesList {
    guard let image = UIImage(named: imageName)
        else { continue }

    // haz algo con `image`
}
```

### Vigilando la indentación excesiva y los errores

Vamos a hacer un antes-y-después de cómo `guard` puede mejorar nuestro código y prevenir errores. 

Como ejemplo, vamos a implementar una función `readBedtimeStory()`:

```swift
enum StoryError: Error {
    case missing
    case illegible
    case tooScary
}

func readBedtimeStory() throws {
    if let url = Bundle.main.url(forResource: "book",
                               withExtension: "txt")
    {
        if let data = try? Data(contentsOf: url),
            let story = String(data: data, encoding: .utf8)
        {
            if story.contains("👹") {
                throw StoryError.tooScary
            } else {
                print("Érase una vez... \(story)")
            }
        } else {
            throw StoryError.illegible
        }
    } else {
        throw StoryError.missing
    }
}
```

Para leer una historia de cuna necesitamos poder encontrar el libro, debe ser descifrable y no puede ser de terror (Sin monstruos en el libro, ¡por favor y gracias!).

Pero nota que tan lejos está la instrucción `throw` de la validación misma.
Para saber qué pasa cuando no puedes encontrar `book.txt`, necesitas leer hasta el final del método.

Como un buen libro, el código debería contar una historia con una trama fácil de seguir, 
y un claro inicio, desarrollo y final.
(Solo intenta no escribir mucho código del género "posmodernista").

El uso estratégico de la instrucción `guard` nos permite organizar nuestro código para que sea más legible de manera lineal.

```swift
func readBedtimeStory() throws {
    guard let url = Bundle.main.url(forResource: "book",
                                  withExtension: "txt")
    else {
        throw StoryError.missing
    }

    guard let data = try? Data(contentsOf: url),
        let story = String(data: data, encoding: .utf8)
    else {
        throw StoryError.illegible
    }

    if story.contains("👹") {
        throw StoryError.tooScary
    }

    print("Érase una vez... \(story)")
}
```

_¡Mucho mejor!_
Cada caso de error es manejado inmediatamente es validado. De esta forma, podemos seguir el flujo de ejecución hacia abajo.

### Evita no vigilar dobles negativos

Un hábito que debemos vigilar en lo que adoptamos este nuevo mecanismo de control, es el de utilizarlo excesivamente ---
particularmente cuando la condición evaluada ya está negada.

Por ejemplo, 
si quieres retornar cuando una cadena de texto está vacía, no escribas:

```swift
// ¿Eh?
guard !string.isEmpty else {
    return
}
```

Mantenlo simple.
Ve con el flujo (de control).
Evita el doble negativo.

```swift
// ¡Ajá!
if string.isEmpty {
    return
}
```

## defer

Entre `guard` y la nueva instrucción `throw` para el manejo de errores, Swift fomenta el estilo de retornar temprano (favorito de NSHipster) en vez de instrucciones `if` anidadas. Sin embargo, retornar de manera anticipada plantea un reto distinto, cuando los recursos han sido inicializados (y pueden aún estar en uso), se debe limpiar antes de retornar.

La palabra clave `defer` permite manejar este reto de forma fácil y segura, declarando un bloque que será ejecutado solo cuando termine la ejecución del alcance actual.

Considera la siguiente función que envuelve una llamada al sistema a `gethostname(2)`
para retornar el actual [nombre de equipo](https://es.wikipedia.org/wiki/Nombre_de_equipo) del sistema.

```swift
import Darwin

func currentHostName() -> String {
    let capacity = Int(NI_MAXHOST)
    let buffer = UnsafeMutablePointer<Int8>.allocate(capacity: capacity)

    guard gethostname(buffer, capacity) == 0 else {
        buffer.deallocate()
        return "localhost"
    }

    let hostname = String(cString: buffer)
    buffer.deallocate()

    return hostname
}
```
Aquí hemos asignado un espacio en memoria a `UnsafeMutablePointer<Int8>` de manera anticipada
pero necesitamos recordar liberar ese espacio tanto en el fallo de la condición, como al terminar de utilizar el espacio temporal de memoria.

¿Propenso a errores? _Sí._   
¿Frustrantemente repetitivo? _Listo._

Utilizando la instrucción `defer` podemos remover el potencial error de los programadores, y simplificar nuestro código:

```swift
func currentHostName() -> String {
    let capacity = Int(NI_MAXHOST)
    let buffer = UnsafeMutablePointer<Int8>.allocate(capacity: capacity)
    defer { buffer.deallocate() }

    guard gethostname(buffer, capacity) == 0 else {
        return "localhost"
    }

    return String(cString: buffer)
}
```
Aunque `defer` está inmediatamente después de la llamada a `allocate(capacity)`,
su ejecución es retrasada hasta el final del alcance actual.
Gracias a `defer`, `buffer` será liberado apropiadamente sin importar
desde dónde retorne la función.

Considera utilizar `defer` cuando un API requiera llamadas balanceadas,
como `allocate(capacity:)` / `deallocate()`,
`wait()` / `signal()`, o
`open()` / `close()`.
De esta forma, no solo eliminas la potencial fuente de error del programador, sino que también rindes honor a Dijkstra.
_"Goed gedaan!" Diría el, en su nativo Holandés_.

### Difiriendo frecuentemente

Si utilizas múltiples instrucciones `defer` en el mismo alcance,
estas serán ejecutadas en el orden contrario ---
como una pila.
Este orden reverso es un detalle vital, debemos asegurarnos de que todo lo que estaba al alcance cuando un bloque diferido fue creado va a seguir estando al alcance cuando este bloque diferido se ejecute.

Por ejemplo, 
al ejecutar el siguiente código, se imprime el resultado mostrado abajo:

```swift
func procrastinate() {
    defer { print("lavar los platos") }
    defer { print("sacar la basura") }
    defer { print("limpiar el refrigerador") }

    print("jugar videojuegos")
}
```

<samp>
jugar videojuegos<br/>
sacar la basura<br/>
sacar el reciclaje<br/>
lavar los platos<br/>
</samp>

> ¿Qué pasa si anidas instrucciones `defer` de la siguiente manera?

```swift
defer { defer { print("limpiar la canaleta") } }
```

> Tu primer pensamiento puede ser que empuja la instrucción al fondo de la pila.
> Pero eso no es lo que pasa.
> Piénsalo, luego prueba tu hipótesis en un Playground.

### Difiriendo juiciosamente

Si se referencia a una variable dentro de un bloque defer se obtiene su valor final, lo cual quiere decir que el bloque no captura el valor actual de una variable.

Si ejecutas la próxima muestra de código, obtendrás el siguiente resultado:

```swift
func flipFlop() {
    var position = "Se pronuncia /ɡɪf/"
    defer { print(position) }

    position = "Se pronuncia /dʒɪf/"
    defer { print(position) }
}
```

<samp>
Se pronuncia /dʒɪf/ <br/>
Se pronuncia /dʒɪf/
</samp>

### Difiriendo moderadamente

Otra cosa a tener en mente es que los bloques de`defer` no pueden romper fuera de su alcance.
Entonces, si tratas de llamar un método que puede lanzar excepciones,
el error no puede ser pasado al contexto de alrededor.

```swift
func burnAfterReading(file url: URL) throws {
    defer { try FileManager.default.removeItem(at: url) }
    // 🛑 Errores no manejados

    let string = try String(contentsOf: url)
}
```

En cambio, puedes ignorar el error utilizando `try?` o simplemente mover la instrucción fuera del bloque de `defer` hasta el final de la función para que se ejecute de manera convencional.

### (Cualquier otro) Defer se considera perjudicial

Aún lo útil que es la instrucción `defer`, debes ser consciente de cómo sus capacidades pueden llevar a
un código confuso y difícil de rastrear.
Puede ser tentador usar `defer` en casos donde una función necesita retornar un valor que necesita ser modificado, como esta típica implementación del operador sufijo `++`:

```swift
postfix func ++(inout x: Int) -> Int {
    let current = x
    x += 1
    return current
}
```

En este caso, `defer` ofrece una alternativa inteligente.
¿Por qué crear una variable temporal cuando podemos diferir el incremento? 

```swift
postfix func ++(inout x: Int) -> Int {
    defer { x += 1 }
    return x
}
```

Muy listo, de igual forma, esta inversión en el flujo de la función perjudica la legibilidad.
Utilizar `defer` para explícitamente alterar el flujo de un programa, en vez de limpiar recursos asignados, nos llevará a una ejecución torcida y enredada.

---

"Como programadores conscientes de nuestras limitaciones,"
debemos balancear el beneficio de cada funcionalidad de un lenguaje contra su costo.

Una nueva instrucción como `guard` nos lleva a un programa más lineal y legible;
aplícalo lo más ampliamente posible.

Igualmente, `defer` soluciona un reto significativo, pero nos fuerza a mantener un rastreo de las declaraciones; úsalo con cautela para evitar confusión en el código.