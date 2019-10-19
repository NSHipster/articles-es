---
title: TimeInterval, Date, y DateInterval
author: Mattt
translator: Leo Picado
category: Cocoa
excerpt: >
  Nuestro limitado entendimiento del tiempo se refleja, y posiblemente se desmejora, por la forma en que fueron nombradas las APIs para tiempo y fechas. Es hora de arreglarlo.
status:
  swift: 4.2
---

Cerca del Parque del Buen Retiro, entre los distritos Centro y Salamanca de Madrid, se encuentra el Museo Nacional del Prado; el mismo contiene una amplia colección de obras de los pintores europeos más famosos.

Si durante su visita se llegara a cansar de los retratos pagados por la monarquía española del siglo XVII podría considerar desviarse hacía la _Sala 002_. Ubicada en el sector norte del primer piso encontrará en su interior [éste óleo barroco de Simon Vouet](https://www.museodelprado.es/coleccion/obra-de-arte/el-tiempo-vencido-por-la-esperanza-y-la-belleza/ebaeb191-f3ff-43b1-9207-fb36a3e5ad5a?searchid=1abb4b06-1485-5ccd-8aff-315d550fc325).

Sin saber el título de la obra resulta comprensible el no tener del todo claro por qué dos mujeres con un gancho y una lanza posan amenazantes sobre un anciano, mientras una orda de querubines le ataca por la espalda.

En "El Tiempo vencido por la Esperanza y la Belleza" Vouet representa al Tiempo usando la figura del anciano, nótese el reloj de arena en su mano izquierda y la guadaña a sus pies. Teniendo un poco más de contexto, vale la pena tomarse unos minutos más analizando este cuadro mientras se contempla la naturaleza enigmática del tiempo.

Ahora, pensemos sobre nuestro propio, y limitado entendimiento del tiempo se refleja, y posiblemente se desmejora, por la forma en que fueron nombradas las APIs para tiempo y fechas.

Corrijamos ese error.

---

Los segundos son la unidad fundamental de tiempo, aparte de eso son la única unidad que tiene una duración fija.

Los meses varían en duración (algunos tienen 30 otros 31, otros 28 o 29), al igual que los años. Algunos años tienen un día adicional y algunos días ganan o pierden una hora por el horario de verano. Todo esto dejando de lado los segundos intercalares que son responsables de los minutos de 61 segundos, las horas de 3601 segundos y las bisemanas de 1209601 segundos.

`TimeInterval` (antes conocido como `NSTimeInterval`) es un alias para el tipo `Double` que se usa para representar una duración como cantidad de segundos. Se le puede encontrar como argumento o retorno en las APIs que interactuan con una duración de tiempo. Al ser un un número flotante de doble precisión `TimeInterval` puede representar submúltiplos, sin embargo si la meta es tener precisión más allá de milisegundos, es mejor usar otra cosa.


## Fecha y hora

Lastimosamente el tipo usado en Foundation para representar tiempo se llama `Date` (fecha en inglés), ya que generalmente usamos «fecha» para hablar de días calendario y «hora» para las hora en un día. Sin embargo, `Date` es independiente de los calendarios y contrario a su nombre, representa un punto absoluto en el tiempo.

{% info %}

¿Porqué `NSDate` y no `NSTime`? Creemos que los creadores de esta API querían que fuera homóloga a [su contraparte `java.util.date`](https://docs.oracle.com/javase/7/docs/api/java/util/Date.html) cuando el <abbr title="Enterprise Objects Framework">EOF</abbr> estaba orientado tanto a Java como Objective-C.

{% endinfo %}

Otro punto confuso con respecto a `Date` es que, aunque representa un punto absoluto en el tiempo, está [definido como un intervalo de tiempo desde una fecha dada](https://github.com/apple/swift-corelibs-foundation/blob/master/Foundation/Date.swift#L17-L20):

```swift
public struct Date : ReferenceConvertible, Comparable, Equatable {
    public typealias ReferenceType = NSDate

    fileprivate var _time: TimeInterval

    // ...
}
```

En este caso la fecha usada como referencia es el 1 de enero del 2001, bajo la zona horaria de Greenwich (GMT).

{% info %}
Siguiendo con las conjeturas sobre esta API, ¿alguien sabe porqué Apple creó un nuevo estándard en vez de usar el Tiempo Unix (Unix Epoch [1 de enero de 1970])? El 2001 fue el año en que se lanzó Mac OS X, pero `NSDate` fue creado desde la época NeXT, ¿se habrán protegido así del [Y2K](https://es.wikipedia.org/wiki/Problema_del_a%C3%B1o_2000)?

{% endinfo %}

## Intérvalos entre tiempos y fechas

La entidad `DateInterval`, agregada en iOS 10 y macOS Sierra a Foundation, representa un intervalo cerrado entre dos puntos absolutos en el tiempo, no confundir con `TimeInterval` que representa una duración en segundos.

¿Para qué sirve `DateInterval`? Para casos como los siguientes:

### Obtener el intervalo de fechas para una unidad calendario

Para obtener la hora en un día de cierto punto en el tiempo o el día para ese punto en el tiempo el primer paso es consultar un calendario. Teniendo lo anterior, podemos determinar el rango de una unidad calendario, como día, mes o año. El método `dateInterval(of:for:)` de `Calendar` nos viene como anillo al dedo para esta tarea:

```swift
let calendar = Calendar.current
let date = Date()
let dateInterval = calendar.dateInterval(of: .month, for: date)
```

Aprovechemos la referencia a `Calendar` y veamos como maneja sin problema alguno la transición a horario de verano:

```swift
let dstComponents = DateComponents(year: 2018,
                                   month: 11,
                                   day: 4)
calendar.dateInterval(of: .day,
                      for: calendar.date(from: dstComponents)!)?.duration
// 90000 segundos
```

_Estamos en el {{ site.time | date: '%Y' }}.
¿No es tiempo ya de dejar de hardcodear 86400 la cantidad de segundos en un día?_

## ¿Cómo calcular intersecciones entre intérvalos de fechas?

Volvamos al Museo Nacional del Prado para el próximo ejemplo. Dentro de su extensa colección de obras de Rubens podemos encontrar en este óleo al que [parecer ser el Dios de la programación en Swift](https://www.museodelprado.es/coleccion/obra-de-arte/eolo/e447dadb-b93f-4ce5-84e9-e6ae1d95c6cd).

Tanto Rubens como Vouet eran pintores barrocos, de hecho eran contemporáneos y podemos determinar con toda exactitud la longitud de tiempo en que ambos estuvieron vivos usando `DateInterval`:

```swift
import Foundation

let calendar = Calendar.current

// Simon Vouet
// 9 de enero de 1590 – 30 de junio de 1649
let vouet =
    DateInterval(start: calendar.date(from:
        DateComponents(year: 1590, month: 1, day: 9))!,
                 end: calendar.date(from:
                    DateComponents(year: 1649, month: 6, day: 30))!)

// Peter Paul Rubens
// 28 de junio de 1577 – 30 de mayo de 1640
let rubens =
    DateInterval(start: calendar.date(from:
                            DateComponents(year: 1577, month: 6, day: 28))!,
                 end: calendar.date(from:
                            DateComponents(year: 1640, month: 5, day: 30))!)

let overlap = rubens.intersection(with: vouet)!

calendar.dateComponents([.year],
                        from: overlap.start,
                        to: overlap.end) // 50 años
```

De acuerdo a nuestros cálculos, Rubens y Vouet coincidieron en el mundo de los vivos por un espacio de 50 años.

Para ganar puntos extra podemos usar `DateIntervalFormatter` para obtener una representación más amigable del período:

```swift
let formatter = DateIntervalFormatter()
formatter.timeStyle = .none
formatter.dateTemplate = "%Y"
formatter.string(from: overlap)
// "1590 – 1640"
```

_Bello._
Tan bello que el código se podría imprimir, enmarcar y colgar junto a [El juicio de Paris](https://www.museodelprado.es/coleccion/obra-de-arte/el-juicio-de-paris/918bc2de-00a9-480d-87ba-96ac25f22bf4?searchMeta=juicio%20de%20par).

---

Al fin y al cabo, nunca sabremos qué `hora` es o si tan siquiera existe realmente el tiempo. Sin embargo, intentemos apreciar la belleza en las APIs de `Date` de Foundation y con ellas, poder superar nuestra falta de entendimiento.

Eso es todo es todo por esta semana, hasta la próxima.
