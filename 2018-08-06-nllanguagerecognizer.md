---
title: NLLanguageRecognizer
author: Mattt
translator: Juan F. Sagasti
category: Cocoa
tags: language
excerpt: >
    El _Machine learning_ (Aprendizaje Automático) ha sido el núcleo del procesamiento del lenguaje natural en las plataformas de Apple durante mucho tiempo, pero ha sido recientemente cuando los desarrolladores externos han tenido acceso para su aprovechamiento.
status:
    swift: 4.2
---

Cuando viajo, una de mis actividades favoritas es escuchar a la gente cuando pasa y tratar de adivinar en qué idioma hablan. Y me inclino a pensar que, con el tiempo, me he vuelvo muy bueno en ello (aunque rara vez llego a saber si acierto).

Con suerte, reconozco alguna palabra o frase del idioma en cuestión con la que puedo estar familiarizado y deduzco cosas a partir de ahí. En cualquier otro caso, lo que hago es construir un _inventario_ fonético con los tipos de sonido presentes. Por ejemplo, ¿está usando el hablante vibrantes múltiples alveolares sonoras [`⟨r⟩`](https://es.wikipedia.org/wiki/Vibrante_alveolar_simple),
vibrantes alveolares simples [`⟨ɾ⟩`](https://es.wikipedia.org/wiki/Vibrante_alveolar_simple),
o aproximantes alveolares [`⟨ɹ⟩`](https://es.wikipedia.org/wiki/Aproximante_alveolar)? ¿Son las vocales en su mayoría abiertas / cerradas; anteriores / posteriores? ¿Algún sonido inusual, como [`⟨ʇ⟩`](https://es.wikipedia.org/wiki/Chasquido_consonántico)?

O al menos, eso es lo que creo que hago. Sinceramente, todo esto ocurre inconsciente y automáticamente para todos nosotros respecto a cualquier tarea de reconocimiento del lenguaje, de forma que solo tenemos una remota idea de cómo llegamos de la entrada a la salida.

Las computadoras funcionan de manera similar. Después de muchas horas de entrenamiento, los modelos de aprendizaje automático pueden predecir el idioma de un texto con una precisión mucho mayor de la que se obtiene con un enfoque formalizado y secuencial.

El _Machine learning_ (Aprendizaje Automático) ha sido el núcleo del procesamiento del lenguaje natural en las plataformas de Apple durante mucho tiempo, pero ha sido recientemente cuando los desarrolladores externos han tenido acceso para su aprovechamiento.

---

El [framework de Lenguaje Natural](https://developer.apple.com/documentation/naturallanguage), nuevo en iOS 12 y macOS 10.14, refina APIs lingüísticas existentes y expone nueva funcionalidad a los desarrolladores.

[`NLTagger`](https://developer.apple.com/documentation/naturallanguage/nltagger)
es un [`NSLinguisticTagger`](https://nshipster.com/nslinguistictagger/) renovado.
[`NLTokenizer`](https://developer.apple.com/documentation/naturallanguage/nltokenizer) es un reemplazo de [`enumerateSubstrings(in:options:using:)`](https://developer.apple.com/documentation/foundation/nsstring/1416774-enumeratesubstrings) (originalmente [`CFStringTokenizer`](https://developer.apple.com/documentation/corefoundation/cfstringtokenizer-rf8)).
[`NLLanguageRecognizer`](https://developer.apple.com/documentation/naturallanguage/nllanguagerecognizer) extiende la funcionalidad comentada en  `NSLinguisticTagger` a través de `dominantLanguage`, brindándonos pistas y predicciones acerca del texto analizado. 

## Reconociendo el idioma de un texto

Así se utiliza `NLLanguageRecognizer` para adivinar el idioma dominante en un texto dado:

```swift
import NaturalLanguage

let string = """
私はガラスを食べられます。それは私を傷つけません。
"""

let recognizer = NLLanguageRecognizer()
recognizer.processString(string)
recognizer.dominantLanguage // ja
```

Crea una instancia de `NLLanguageRecognizer` y llama al método  `processString(_:)` pasándole una ristra. A partir de la propiedad `dominantLanguage` se accede a un objeto de tipo `NLLanguage` que contiene el tag BCP-47 del idioma predicho.
En este caso, `"ja"`, del japonés (日本語).

### Obteniendo hipótesis de varios idiomas

Si estudiaste lingüística en la universidad o cursaste latín en el instituto, estarás familiarizado con algunos de los ejemplos más graciosos de _homonimia lingüística_ entre el latín y el italiano moderno.

Por ejemplo, considera la siguiente frase:

> CANE NERO MAGNA BELLA PERSICA!

| Idioma   | Traducción                                  |
| -------- | ------------------------------------------- |
| Latin    | ¡Canta, Nero, la gran Guerra Pérsica!       |
| Italiano | ¡El perro negro se come un bonito melocotón!|

Para disgusto de [Max Fisher](<https://en.wikipedia.org/wiki/Rushmore_(film)>),
el latín no es uno de los idiomas soportados por `NLLanguageRecognizer`, así que
no contaremos con ejemplos tan entretenidos como el anterior. 

A poco que experimentes, te darás cuenta de que es bastante difícil que `NLLanguageRecognizer` adivine incorrectamente o con mala precisión. Es capaz de pasar de la desviación estándar al 95% de certeza con tan solo un puñado de palabras.

Después de un poco de ensayo y error, logramos que `NLLanguageRecognizer` se equivoque con un texto de tamaño no trivial, como el [Artículo I de la Declaración Universal de Derechos Humanos en noruego bokmål](https://www.ohchr.org/EN/UDHR/Pages/Language.aspx?LangID=nrr):

```swift
let string = """
Alle mennesker er født frie og med samme menneskeverd og menneskerettigheter.
De er utstyrt med fornuft og samvittighet og bør handle mot hverandre i brorskapets ånd.
"""

let languageRecognizer = NLLanguageRecognizer()
languageRecognizer.processString(string)
recognizer.dominantLanguage // da (!)
```

> La [Declaración Universal de Derechos Humanos](http://www.un.org/es/universal-declaration-human-rights/), es uno de los documentos más traducidos del mundo, con traducciones en más de 500 idiomas. Por esta razón, se usa muy a menudo para tareas de procesamiento del lenguaje natural.

El danés y el noruego bokmål son lenguajes muy similares, por lo que no es de extrañar que `NLLanguageRecognizer` predijera incorrectamente.
(Para comparar, aquí tienes el [texto equivalente en danés](https://www.ohchr.org/EN/UDHR/Pages/Language.aspx?LangID=dns))

Podemos usar el método `languageHypotheses(withMaximum:)` para tener una idea de cuán segura era la conjetura de `dominantLanguage`:

```swift
languageRecognizer.languageHypotheses(withMaximum: 2)
```

| Idioma                  | Fiabilidad |
| ----------------------- | ---------- |
| Danés (`da`)            | 56%        |
| Noruego bokmål (`nb`)   | 43%        |

En el momento en que escribí este artículo, la propiedad [`languageHints`](https://developer.apple.com/documentation/naturallanguage/nllanguagerecognizer/3017455-languagehints) no estaba documentada, así que no está muy claro cómo debería usarse. Sin embargo, si pasamos un diccionario ponderado de pistas, parece tener el efecto esperado, reforzando las hipótesis:

```swift
languageRecognizer.languageHints = [.danish: 0.25, .norwegian: 0.75]
```

| Idioma                  | Fiabilidad (con pistas) |
| ----------------------- | ----------------------- |
| Danish (`da`)           | 30%                     |
| Noruego Bokmål (`nb`)   | 70%                     |

¿Y qué podemos hacer una vez sabemos el idioma de una ristra?

Aquí tienes algunos casos de uso a considerar:

## Comprobación ortográfica

Combina `NLLanguageRecognizer` con [`UITextChecker`](https://nshipster.es/uitextchecker/) para comprobar la ortografía de las palabras de una ristra:

Crea un `NLLanguageRecognizer` e inicialízalo con una ristra llamando al método  `processString(_:)`:

```swift
let string = """
Wenn ist das Nunstück git und Slotermeyer?
Ja! Beiherhund das Oder die Flipperwaldt gersput!
"""

let languageRecognizer = NLLanguageRecognizer()
languageRecognizer.processString(string)
let dominantLanguage = languageRecognizer.dominantLanguage! // de
```

Luego, pasa el `rawValue` del objeto `NLLanguage` devuelto por la propiedad  `dominantLanguage` al parámetro `language` de
`rangeOfMisspelledWord(in:range:startingAt:wrap:language:)`:

```swift
let textChecker = UITextChecker()

let nsString = NSString(string: string)
let stringRange = NSRange(location: 0, length: nsString.length)
var offset = 0

repeat {
    let wordRange =
            textChecker.rangeOfMisspelledWord(in: string,
                                              range: stringRange,
                                              startingAt: offset,
                                              wrap: false,
                                              language: dominantLanguage.rawValue)
    guard wordRange.location != NSNotFound else {
        break
    }

    print(nsString.substring(with: wordRange))

    offset = wordRange.upperBound
} while true
```

Cuando se le pasa [El chiste más gracioso del mundo](https://es.wikipedia.org/wiki/El_chiste_más_gracioso_del_mundo), se identifican las siguientes palabras como mal escritas:

- Nunstück
- Slotermeyer
- Beiherhund
- Flipperwaldt
- gersput

## Sintetización del habla

Puedes combinar `NLLanguageRecognizer` con [`AVSpeechSynthesizer`](https://nshipster.com/avspeechsynthesizer/) para escuchar cualquier texto de un idioma leído en voz alta: 

```swift
let string = """
Je m'baladais sur l'avenue le cœur ouvert à l'inconnu
    J'avais envie de dire bonjour à n'importe qui.
N'importe qui et ce fut toi, je t'ai dit n'importe quoi
    Il suffisait de te parler, pour t'apprivoiser.
"""

let languageRecognizer = NLLanguageRecognizer()
languageRecognizer.processString(string)
let language = languageRecognizer.dominantLanguage!.rawValue // fr

let speechSynthesizer = AVSpeechSynthesizer()
let utterance = AVSpeechUtterance(string: string)
utterance.voice = AVSpeechSynthesisVoice(language: language)
speechSynthesizer.speak(utterance)
```

No tiene la delicadeza lírica de [Joe Dassin](https://itunes.apple.com/us/album/les-champs-%C3%A9lys%C3%A9es/311331439?i=311331447), pero _ainsi va la vie_.

---

Antes de entender algo, primero tienes que querer entender. Y el primer paso para entender el lenguaje natural es determinar su idioma.

`NLLanguageRecognizer` ofrece una potente interfaz a la funcionalidad responsable de muchas de las funciones inteligentes presentes en iOS y macOS. Intenta sacarle partido para tener un mayor entendimiento sobre tus usuarios.
