---
title: "Password Rules / UITextInputPasswordRules"
author: Mattt
translator: Juan F. Sagasti
category: "Cocoa"
excerpt: 
    A menos que sea el título de una película de hackers de los 90 o la solución al rompecabezas de una escape room, una contraseña debería estar completamente desprovista de significado.
hiddenlang: ""
status:
    swift: "4.2"
---

No es ningún secreto que los hipsters sienten pasión por lo artesanal; lo hecho a mano. Ya sea una buena tosta de aguacate, una botella de leche de un lote pequeño o la taza de café perfecta.
No hay nada como el toque humano.

En cambio, una buena contraseña es todo lo contrario; es lo puesto a lo artesanal. A menos que sea el título de una película de hackers de los 90 o la solución al rompecabezas de una escape room, una contraseña debería estar completamente desprovista de significado.

Con Safari, en iOS 12 y macOS Mojave, será más fácil que nunca generar la contraseña más robusta, más carente de significado y más difícil de adivinar que puedas imaginar.
Y todo gracias a unas pocas características nuevas.

---

Una política de contraseñas apropiada debe ser simple: requerir un número mínimo de caracteres (al menos 8) y permitir contraseñas largas (64 caracteres o más). Cualquier cosa adicional más elaborada, como preguntas de seguridad, tiempos de expiración periódicos o requisitos forzados de caracteres, no hacen más que incordiar a los usuarios que dichas políticas tratan de proteger.

> No tomes mis palabras muy en serio, no soy ningún experto en seguridad.
> En cambio, echa un vistazo a las [Digital Identity Guidelines](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-63b.pdf) del <abbr title="National Institute of Standards and Technology">NIST</abbr> (publicado en junio de 2017).


La buena noticia es que cada vez más empresas y organizaciones prestan atención a las buenas prácticas sobre seguridad. 
La mala noticia es que requirió una serie de brechas de seguridad que afectaron a millones de personas antes de que empezaran a cambiar las cosas.
Y la terrible verdad es que muchos de los anti-patrones de seguridad existentes no se corregirán pronto debido a la lentitud que tienen los gobiernos y las corporaciones en hacer las cosas.

## Contraseñas Seguras Automáticas

El autocompletado de Safari (`AutoFill`) presente desde iOS 8 es capaz de generar contraseñas, pero uno de sus puntos débiles era que no aseguraba que una contraseña generada satisfaciera los requisitos de un servicio en particular. 
Pero Apple apunta a solventar este problema con una característica nueva (`Atomatic Strong Passwords`) en Safari, iOS 12 y macOS Mojave.

El ingeniero de WebKit Daniel Bates envió [esta propuesta](https://github.com/whatwg/html/issues/3518) al comité <abbr title="Web Hypertext Application Technology Working Group">WHATWG</abbr> el 1 de marzo. 
Y el 6 de junio, el equipo de WebKit [anunció el lanzamiento de Safari Technology Preview 58](https://webkit.org/blog/8327/safari-technology-preview-58-with-safari-12-features-is-now-available/), con soporte para la generación de contraseñas seguras que usen el atributo `passwordrules`. Este anuncio coincidió con el lanzamiento de los SDKs beta de iOS 12 en la WWDC, los cuales incluían una API nueva de `UITextInputPasswordRules` junto con otra serie de características nuevas para la gestión de contraseñas que incluían, por ejemplo, el autorrelleno de códigos de seguridad.


## Requisitos de Contraseñas

Los requisitos de contraseñas son como una receta para los generadores de contraseñas. Siguiendo una serie de reglas simples, el generador puede crear aleatoriamente contraseñas seguras que satisfacen una serie de requisitos especificados por el proveedor del servicio.

Estas reglas consisten en uno o más pares clave-valor en la forma:

`required: lower; required: upper; required: digit; allowed: ascii-printable; max-consecutive: 3;`

### Claves

Cada regla puede especificar una de las siguientes claves:

- `required`: Tipos de caracteres requeridos
- `allowed`: Tipos de caracteres permitidos
- `max-consecutive`: Número máximo de caracteres consecutivos permitido
- `minlength`: Longitud mínima de la contraseña
- `maxlength`: Longitud máxima de la contraseña

Las claves `max-consecutive`, `minlength`, and `maxlength` tienen un entero no negativo como valor.
Las claves `required` y `allowed` tienen una de las siguientes clases como valor:


### Clases de Caracteres

- `upper` (`A-Z`)
- `lower` (`a-z`)
- `digits` (`0-9`)
- `special` (`` -~!@#$%^&\*\_+=`|(){}[:;"'<>,.? ] `` and space)
- `ascii-printable` (U+0020 — 007f)
- `unicode` (U+0 — 10FFFF)


También puedes declarar una clase de caracteres personalizada especificando una secuencia de caracteres ASCII entre corchetes (por ejemplo, `[abc]`).

---

La [herramienta de validación de contraseñas](https://developer.apple.com/password-rules/) de Apple te permite experimentar con diferentes reglas y obtener feedback en tiempo real de los resultados. ¡Puedes generar y descargar miles de contraseñas para desarrollo y testing!

{% asset password-rules-validation-tool.png alt="Herramienta de Validación de Contraseñas" %}

Para más información sobre la sintaxis de las reglas, echa un vistazo a ["Customizing Password AutoFill Rules"](https://developer.apple.com/documentation/security/password_autofill/customizing_password_autofill_rules).

---

## Definiendo Reglas de Contraseña

En iOS, inicializa la propiedad `passwordRules` de un `UITextField` con un objeto de tipo `UITextInputPasswordRules` (inicializa también `textContentType` al valor  `.newPassword` ):

```swift
let newPasswordTextField = UITextField()
newPasswordTextField.textContentType = .newPassword
newPasswordTextField.passwordRules = UITextInputPasswordRules(descriptor: "required: upper; required: lower; required: digit; max-consecutive: 2; minlength: 8;")
```

En web, inicializa el atributo `passwordrules` a un elemento `<input>` con `type="password"`:

```html
<input type="password" passwordrules="required: upper; required: lower; required: special; max-consecutive: 3;"/>
```

> Si no se especifica, se aplica la regla `allowed: ascii-printable` por defecto.
> Aunque si tu formulario tiene un campo de confirmación de contraseña se aplicarán automáticamente las reglas del campo precedente.

## Generando Reglas de Contraseña en Swift

Si la idea de trabajar con un formato basado en ristras sin una abstracción adecuada te revuelve las tripas tranquilo, no estás solo. Aquí tienes una forma de encapsular las reglas en una API de Swift:

```swift
enum PasswordRule {
    enum CharacterClass {
        case upper, lower, digits, special, asciiPrintable, unicode
        case custom(Set<Character>)
    }

    case required(CharacterClass)
    case allowed(CharacterClass)
    case maxConsecutive(UInt)
    case minLength(UInt)
    case maxLength(UInt)
}

extension PasswordRule: CustomStringConvertible {
    var description: String {
        switch self {
        case .required(let characterClass):
            return "required: \(characterClass)"
        case .allowed(let characterClass):
            return "allowed: \(characterClass)"
        case .maxConsecutive(let length):
            return "max-consecutive: \(length)"
        case .minLength(let length):
            return "minlength: \(length)"
        case .maxLength(let length):
            return "maxlength: \(length)"
        }
    }
}

extension PasswordRule.CharacterClass: CustomStringConvertible {
    var description: String {
        switch self {
        case .upper: return "upper"
        case .lower: return "lower"
        case .digits: return "digits"
        case .special: return "special"
        case .asciiPrintable: return "ascii-printable"
        case .unicode: return "unicode"
        case .custom(let characters):
            return "[" + String(characters) + "]"
        }
    }
}
```

Con lo anterior podemos declarar una serie de reglas en código y usarlas para generar una ristra con una sintaxis válida:

```swift
let rules: [PasswordRule] = [ .required(.upper),
                              .required(.lower),
                              .required(.special),
                              .minLength(20) ]

let descriptor = rules.map { "\($0.description);" }
                      .joined(separator: " ")

// "required: upper; required: lower; required: special; max-consecutive: 3;"
```

Si te atreves podrías incluso extender `UITextInputPasswordRules` para ofrecer un inicializador de conveniencia que acepte un vector de tipo `PasswordRule`:

```swift
extension UITextInputPasswordRules {

    convenience init(rules: [PasswordRule]) {
        let descriptor = rules.map{ $0.description }
                              .joined(separator: "; ")

        self.init(descriptor: descriptor)
    }
}
```

**BONUS!** He creado un paquete Swift con esta API llamado [PasswordRules](https://github.com/NSHipster/PasswordRules).

---

Si eres de aquellos sentimentales que disfruta usando el nombre de su perro o de su equipo favorito en los campos de contraseña, te recomiendo corregir esa costumbre.

Personalmente, no puedo imaginar el día a día sin mi gestor de contraseñas. Es difícil exagerar la tranquilidad que obtienes sabiendo que cualquier información que necesites es accesible en cualquier momento únicamente por ti.

Realizar este paso ahora te permitirá aprovechar las mejoras de Safari cuando iOS 12 y macOS Mojave lleguen en breve este año.
