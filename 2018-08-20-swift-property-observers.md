---
title: Observadores de propiedad de Swift
author: Mattt
translator: Juan F. Sagasti
category: Swift
excerpt: >
  El desarrollo de software moderno se ha convertido en lo que podría
  considerarse la quintaesencia de un artilugio de Goldberg. Y aún así hay cabida, en ocasiones, en las que la [_acción a distancia_](https://es.wikipedia.org/wiki/Antipatrón_de_diseño) ayuda a aclarar más que a confundir.
status:
  swift: 4.2
---

En la década de 1930, Rube Goldberg ya era un nombre familiar; sinónimo de invenciones increíblemente complejas y retorcidas representadas en forma de tiras cómicas, como la de ["La servilleta automática"](https://upload.wikimedia.org/wikipedia/commons/a/a9/Rube_Goldberg%27s_%22Self-Operating_Napkin%22_%28cropped%29.gif).
Por aquel entonces, Albert Einstein popularizó la frase «acción espeluznante a distancia» en su [crítica](https://es.wikipedia.org/wiki/Paradoja_EPR) hacia la interpretación de la mecánica cuántica de Niels Bohr, predominante por aquel entonces.

Casi un siglo después, el desarrollo de software se ha convertido en lo que podría considerarse la quintaesencia de un artilugio de Goldberg, extendiéndose incluso por ese reino espeluznante gracias a la computación cuántica. 

Como desarrolladores de software, se nos alienta a reducir esas _acciónes a distancia_ en nuestro código cuanto sea posible. Esto puede observarse en patrones con nombres contundentes, como el [Principio de responsabilidad única](https://es.wikipedia.org/wiki/Principio_de_responsabilidad_única), el [Principio de la mínima sorpresa](https://es.wikipedia.org/wiki/Principio_de_la_M%C3%ADnima_Sorpresa) y la [Ley de Demeter](https://es.wikipedia.org/wiki/Ley_de_Demeter). Y a pesar de las dudas que puedan despertar estos principios acerca de los efectos secundarios, aún hay cabida, en ocasiones, para que tales efectos ayuden a aclarar, más que a confundir.

El artículo de esta semana se centra en los observadores de propiedad de Swift, los cuales ofrecen, de manera nativa, una alternativa ligera a soluciones más formales como programación funcional reactiva (FRP) con model-view-viewmodel (MVVM).

---

Hay dos tipos de propiedades en Swift: _stored properties_, las cuales asocian estado con un objeto, y _computed properties_, que realizan un cálculo en base a dicho estado. Por ejemplo:

```swift
struct S {
    // Stored Property
    var stored: String = "stored"

    // Computed Property
    var computed: String {
        return "computed"
    }
}
```

Cuando declaras una stored property, tienes la opción de definir observadores mediante bloques de código que se ejecutan cuando se le asigna un valor a la propiedad. El observador `willSet` se ejecuta justo antes de que se asigne el nuevo valor, y `didSet` justo después. Se ejecutan siempre, incluso aunque el valor anterior sea igual que el nuevo.

```swift
struct S {
    var stored: String {
        willSet {
            print("se llamó a willSet")
            print("stored es ahora igual a \(self.stored)")
            print("se asignará \(newValue) a stored")
        }

        didSet {
            print("se llamó a didSet")
            print("stored es ahora igual a \(self.stored)")
            print("stored tenía previamente el valor \(oldValue)")
        }
    }
}
```


```swift
var s = S(stored: "first")
s.stored = "second"
```

Por ejemplo, al ejecutar el código anterior se imprimirá por consola el siguiente resultado:

- <samp>se llamó a willSet</samp>
- <samp>stored es ahora igual a first</samp>
- <samp>se asignará second a stored</samp>
- <samp>se llamó a didSet</samp>
- <samp>stored es ahora igual a second</samp>
- <samp>stored tenía previamente el first</samp>

> Advertencia: los observadores no se ejecutan si inicializas una propiedad en un método _init_. Hasta Swift 4.2, puedes salvar esto empaquetando la llamada _setter_ en un bloque `defer`, pero esto es [un bug que se arreglará en breve](https://twitter.com/jckarter/status/926459181661536256), por lo que no deberías depender de él.

---

Los observadores de propiedades en Swift han sido parte del lenguaje prácticamente desde sus inicios. Para entender por qué, echemos un vistazo a cómo son las cosas en Objective-C:


## Propiedades en Objective-C

En Objective-C, todas las propiedades son, en cierto sentido, computadas. Cada vez que una propiedad se accede mediante la notación del punto, equivale a llamar a su método _getter_ o _setter_. Esto, a su vez, se compila en un paso de mensaje que ejecuta una función que lee o escribe una variable de instancia.


```objc
// El acceso por punto
person.name = @"Johnny";

// ...equivale a
[person setName:@"Johnny"];

// ... que es compilado a
objc_msgSend(person, @selector(setName:), @"Johnny");

// ...cuya implementación sintetizada representa
person->_name = @"Johnny";
```

Los efectos secundarios son, generalmente, algo a evitar en la programación debido a que dificulta la compresión del comportamiento de un programa. Pero muchos desarrolladores de Objective-C han terminado apoyándose en la capacidad de inyectar comportamiento adicional a un getter o setter según sea necesario.

El diseño de Swift ha formalizado estos patrones y creado distinción entre los efectos secundarios derivados del acceso a estado (stored properties) de aquellos que redireccionan acceso a estado (computed properties). Para stored properties, los observadores `willSet` y `didSet` reemplazan el código que de lo contrario incluirías junto con el acceso a la ivar. Para computed properties, los métodos `get` y `set` reemplazan el código que podrías implementar para propiedades `@dynamic` en Objective-C.

Como resultado, obtenemos una semántica más consistente y mayores garantías en mecanismos que interactúan con propiedades, como el Key-Value Observing (KVO) y Key-Value Coding (KVC).

---

Entonces, ¿qué podemos hacer con observadores en Swift? Aquí tienes un par de ideas a considerar:

---

## Validación / normalización de valores

A veces es necesario aplicar ciertas restricciones sobre qué valores son aceptables para un tipo.

Por ejemplo, si estuvieras desarrollando una app que interactúa con burocracia del gobierno, necesitas asegurar que el usuario no puede enviar un formulario en el que falte algún campo requerido, o que contenga algún valor inválido.

Si, por ejemplo, un formulario requiere que los nombres usen letras mayúsculas sin acentos, podrías usar el observador `didSet` para eliminar tildes y aplicar mayúsculas al nuevo valor:


```swift
var name: String? {
    didSet {
        self.name = self.name?
                        .applyingTransform(.stripDiacritics,
                                            reverse: false)?
                        .uppercased()
    }
}
```

Por suerte, asignar un valor a una propiedad en el cuerpo de un observador `didSet` no dispara nuevas llamadas, por lo que no creamos un bucle infinito. En cambio esto no pasa con el observador `willSet`; cualquier valor asignado en el cuerpo se sobreescribe inmediatamente cuando a la propiedad se le asigna el `newValue`.

Si bien es cierto que este enfoque puede funcionar para problemas puntuales, repetir su uso es un fuerte indicativo de que la lógica de negocio puede formalizarse en un tipo.

Sería mejor crear un tipo `NormalizedText` que encapsulara los requerimientos de texto del formulario:

```swift
struct NormalizedText {
    enum Error: Swift.Error {
        case empty
        case excessiveLength
        case unsupportedCharacters
    }

    static let maximumLength = 32

    var value: String

    init(_ string: String) throws {
        if string.isEmpty {
            throw Error.empty
        }

        guard let value = string.applyingTransform(.stripDiacritics,
                                                   reverse: false)?
                                .uppercased(),
              value.canBeConverted(to: .ascii)
        else {
             throw Error.unsupportedCharacters
        }

        guard value.count < NormalizedText.maximumLength else {
            throw Error.excessiveLength
        }

        self.value = value
    }
}
```

Un inicializador que falle o lance excepciones puede indicar errores al que hace la llamada de una manera en que `didSet` no puede. Ahora, si un alborotador como _Jøhnny_ de [Llanfairpwllgwyngyllgogerychwyrndrobwllllantysiliogogogoch](https://es.wikipedia.org/wiki/Llanfairpwllgwyngyllgogerychwyrndrobwllllantysiliogogogoch)
viene buscando pelea, ¡podemos darle su merecido!

(Es decir, comunicarle errores de una forma razonable en lugar de fallar de manera silenciosa o permitir información inválida)

## Propagando estado dependiente 

Otro uso potencial de los observadores es propagar estado a componentes dependientes en un view controller.

Considera el siguiente ejemplo de un modelo `Track` y un `TrackViewController` que lo presenta:

```swift
struct Track {
    var title: String
    var audioURL: URL
}

class TrackViewController: UIViewController {
    var player: AVPlayer?

    var track: Track? {
        willSet {
            self.player?.pause()
        }

        didSet {
            self.title = self.track.title

            let item = AVPlayerItem(url: self.track.audioURL)
            self.player = AVPlayer(playerItem: item)
            self.player?.play()
        }
    }
}
```

Cuando se asigna un valor a la propiedad `track` del view controller, pasa lo siguiente de manera automática:

1. Se pausa cualquier pista de audio anterior.
2. Se actualiza el `title` del view controller al título de la pista nueva.
3. Se carga y reproduce la nueva pista de audio.

_Fantástico, ¿verdad?_

Podrías incluso implementar un comportamiento en cascada a través de múltiples propiedades observadas, al estilo [de esta escena de _Mousehunt_](https://www.youtube.com/watch?v=TVAhhVrpkwM).

---

Como regla general, los efectos secundarios son algo a evitar en la programación debido a que dificulta la compresión de comportamiento complejo. Tenlo en mente la próxima vez que vayas a utilizar esta nueva herramienta.

Sin embargo, desde la punta afilada de esta tambaleante torre de abstracción que es la programación, puede ser tentador (y quizá, a veces, gratificante), abrazar el caos del sistema.
Seguir siempre las reglas es _a**Bohr**rido_.
