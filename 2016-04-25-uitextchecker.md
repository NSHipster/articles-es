---
title: UITextChecker
author: Croath Liu
translator: Juan F. Sagasti
category: Cocoa
excerpt: "Un teclado diminuto dibujado sobre un trozo de cristal no siempre permite una escritura perfecta. Y ya sea porque te corrige con destreza o con algún divertido sinsentido, cualquiera que escriba en un dispositivo iOS se da cuenta cuando entra en acción el autocorrector. Quizá no sepas que UIKit cuenta con una clase para asistir la escritura de los usuarios dentro de tu app."
status:
    swift: 4.2
---

Un teclado diminuto dibujado sobre un trozo de cristal no siempre permite una escritura perfecta. Y ya sea porque te corrige con destreza o con algún divertido sinsentido, cualquiera que escriba en un dispositivo iOS se da cuenta rápidamente cuando entra en acción el autocorrector. Quizá no sepas que UIKit cuenta con una clase para asistir la escritura de los usuarios dentro de tu app.

Introducido en iOS 3.2 (¿o deberíamos llamarlo iPhone OS 3.2, dada la fecha?),  `UITextChecker` hace lo que su propio nombre indica: comprueba texto. Sigue leyendo para aprender cómo usar esta clase para comprobación de ortografía y completado de palabras. 

---

## Comprobación de ortografía

¿Qué ocurre si escribes mal una palabra en iOS? Si escribes «hipstar» en un campo de texto, iOS te sugerirá corregirlo a «hipster» la mayoría de las veces.

{% asset uitextchecker-hipstar.png alt="Autocorrecting 'hipstar'" %}

Podemos conseguir la misma sugerencia usando `UITextChecker`:

```swift
import UIKit

let str = "hipstar"
let textChecker = UITextChecker()
let misspelledRange = textChecker.rangeOfMisspelledWord(in: str,
                                                        range: NSRange(0..<str.utf16.count),
                                                        startingAt: 0,
                                                        wrap: false,
                                                        language: "en_US")

if misspelledRange.location != NSNotFound,
    let firstGuess = textChecker.guesses(forWordRange: misspelledRange,
                                         in: str,
                                         language: "en_US")?.first
{
    print("First guess: \(firstGuess)") // First guess: hipster
} else {
    print("Not found")
}
```

```objc
NSString *str = @"hipstar";
UITextChecker *textChecker = [[UITextChecker alloc] init];
NSRange misspelledRange = [textChecker
                           rangeOfMisspelledWordInString:str
                           range:NSMakeRange(0, [str length])
                           startingAt:0
                           wrap:NO
                           language:@"en_US"];

NSArray *guesses = [NSArray array];
if (misspelledRange.location != NSNotFound) {
    guesses = [textChecker guessesForWordRange:misspelledRange
                                      inString:str
                                      language:@"en_US"];
    NSLog(@"First guess: %@", [guesses firstObject]);
    // First guess: hipster
} else {
    NSLog(@"Not found");
}
```

El array de ristras devuelvo _podría_ parecerse a este:

```swift
["hipster", "hip star", "hip-star", "hips tar", "hips-tar"]
```

O quizá no, ya que `UITextChecker` sugiere en base al contexto y al dispositivo. De acuerdo a la documentación, `guesses(forWordRange:in:language:)` devuelve un array de ristras en el orden en el que deberían mostrarse, representando sugerencias de palabras que podrían aplicarse en lugar de la palabra mal escrita presente en el rango y ristra dados. 

Por lo tanto, no se asegura idempotencia o corrección, lo cual tiene sentido para un método que cuenta con `guesses` en su propio nombre. ¿Cómo podrían fiarse los NSHipsters de su valor de retorno? Encontraremos la respuesta si escarbamos un poco más.

## Aprendiendo palabras nuevas

Asumamos que quieres que tus usuarios puedan escribir `«hipstar»` exactamente. Puedes decirle a tu app que aprenda la palabra usando el método de clase `UITextChecker.learnWord(_:)`:

```swift
UITextChecker.learnWord(str)
```

```objc
[UITextChecker learnWord:str];
```

Así que `«hipstar»` es ahora una palabra reconocida para el dispositivo entero y no aparecerá como incorrecta en futuras comprobaciones.

```swift
let misspelledRange = textChecker.rangeOfMisspelledWord(in: str,
                                                        range: NSRange(0..<str.utf16.count),
                                                        startingAt: 0, wrap: false, language: "en_US")
misspelledRange.location == NSNotFound // true
```

```objc
NSRange misspelledRange = [textChecker
                           rangeOfMisspelledWordInString:str
                           range:NSMakeRange(0, [str length])
                           startingAt:0
                           wrap:NO
                           language:@"en_US"];
misspelledRange.location == NSNotFound // YES
```

La búsqueda del ejemplo superior devuelve `NSNotFound` porque `UITextChecker` aprendió la palabra que habíamos creado. `UITextChecker` también cuenta con métodos de clase para comprobar y olvidar palabras: `UITextChecker.hasLearnedWord(_:)` y `UITextChecker.unlearnWord(_:)`, respectivamente..

## Sugerencias de completado

Existe una API de `UITextChecker` para encontrar posibles completados dada una palabra parcial:

```swift
let partial = "hipst"
let completions = textChecker.completions(forPartialWordRange: NSRange(0..<partial.utf16.count),
                                          in: partial,
                                          language: "en_US")
completions == ["hipster", "hipsters", "hipster's"] // true
```

```objc
NSString *partial = @"hipst";
NSArray *completions = [textChecker
                        completionsForPartialWordRange:NSMakeRange(0, [partial length])
                        inString:partial
                        language:@"en_US"];
completions == ["hipster", "hipsters", "hipster's"] // YES
```

`completions(forPartialWordRange:in:language:)` devuelve un array de posibles palabras dada una ristra parcial inicial. Aunque la documentación dice que el array de ristras devuelto está ordenado por probabilidad, en realidad está ordenado alfabéticamente. La versión hermana de `UITextChecker` en macOS, llamada `NSSpellChecker`, sí se comporta como describe la documentación. 

> No verás ninguna de las palabras personalizadas que le enseñes a `UITextChecker` aparecer como sugerencias de completado. Esto es así porque el vocabulario añadido vía `UITextChecker.learnWord(_:)` es global al dispositivo y así se evita que las palabras aprendidas en tu app aparezcan en las autocorrecciones de otra.

---

Si estás construyendo una app con una gran carga textual, usa `UITextChecker` para asegurarte de que el sistema no corrija tu propio vocabulario. Si estás escribiendo una extensión para el teclado, usa `UITextChecker` y `UILexicon` para proporcionar palabras del diccionario del sistema y los nombres y apellidos de los usuarios de la libreta de direcciones. ¡Puedes soportar prácticamente cualquier idioma sin necesidad de crear tus propios diccionarios!
