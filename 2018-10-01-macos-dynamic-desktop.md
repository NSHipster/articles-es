---
title: Escritorio Dinámico de macOS
author: Mattt
translator: Leo Picado
category: Miscellaneous
excerpt: >
  Completando la terna de personalización de temas oscuros, Night Shift, agregado ya hace un par de años, llegan en macOS Mojave: Modo Oscuro y Escritorio Dinámico.
status:
  swift: 4.2
---

Para nosotros, los desarrolladores que tendemos a preferir temas claros sobre fondos oscuros en nuestros editores de texto, pero también apreciamos la consistencia a través de todo el sistema operativo, el nuevo Modo Oscuro representa una de las características más agradables agregadas a macOS en los últimos años.

Night Shift se llevó las palmas hace un par de años por aliviar la fatiga visual en esas sesiones de programación que terminan ya sea muy tarde en la noche o muy temprano en la mañana.

Al combinar ambas funcionalidades, tenemos como resultado la nueva característica introducida en Mojave, Escritorio Dinámico. La opción se encuentra disponible al acceder a "Preferencias del Sistema > Escritorio y Salvapantallas", la imagen cambiará durante el día, basado en tu ubicación.

{% asset desktop-and-screen-saver-preference-pane.png %}

El resultado es sutil pero significativo. El ver cómo transiciona el fondo de pantalla durante el día, sincronizado con el mundo real, hace que nuestro escritorio cobre vida o por lo menos, nos da un hermoso efecto visual durante la transición al encender y apagar Modo Oscuro.

_Pero, la pregunta es ¿cómo funciona?_<br/>
Ése es el tema de esta semana en NSHipster.

La respuesta involucra adentrarnos en las profundidades de los distintos formatos de imágenes, un poco de ingeniería inversa e incluso una pizca de trigonometría esférica.

---

Para entender cómo funciona Escritorio Dinámico el primer paso es obtener una imagen dinámica.

Si estás usando macOS Mojave, abre el Finder y selecciona "Ir > Ir a la carpeta..." (<kbd>⇧</kbd><kbd>⌘</kbd><kbd>G</kbd>), e ingresa la siguiente ruta: "/Library/Desktop Pictures/".

{% asset go-to-library-desktop-pictures.png %}

En esta carpeta debería existir un archivo llamado "Mojave.heic", dale doble clic para abrirlo con Vista Previa.

{% asset mojave-heic.png %}

Usando Vista Previa, podemos ver que la barra lateral muestra miniaturas numeradas del 1 al 16, cada una de ellas contiene una vista distinta de la escena del desierto.

{% asset mojave-dynamic-desktop-images.png %}

Al seleccionar "Herramientas > Mostrar Inspector (<kbd>⌘</kbd><kbd>I</kbd>), podemos ver información general acerca de la imagen:

{% asset mojave-heic-preview-info.png %}

Desafortunadamente, la información que nos da Vista Previa, por lo menos al momento de escritura del artículo, es insuficiente. Inclusive, si hacemos clic al panel contiguo llamado "Inspector de información adicional", los datos obtenidos resultan poco satisfactorios.

|              |            |
| ------------ | ---------- |
| Color Model  | RGB        |
| Depth:       | 8          |
| Pixel Height | 2,880      |
| Pixel Width  | 5,120      |
| Profile Name | Display P3 |

{% info %}

La extensión del archivo, `.heic`, es correspondiente a contenedores de imágenes que usan el formato <abbr title="High-Efficiency Image File Format">HEIF</abbr>, también llamado formato de archivo de imagen de alta eficiencia, está basada en el formato <abbr title="High-Efficiency Video Compression">HEVC</abbr>, conocido como codificación de vídeo de alta eficiencia ó H.265 video. Para más detalles vale la pena ver [la sesión 503 de la WWDC del 2017, llamada: "Introducing HEIF and HEVC"](https://developer.apple.com/videos/play/wwdc2017/503/)

{% endinfo %}

Para saber más sobre esta imagen, tocará ensuciarse las manos al interactuar con algunas APIs de bajo nivel.

## Investigando con CoreGraphics

Empecemos nuestra labor con un nuevo Playground en Xcode. Para no complicar de más las cosas, usemos la dirección a "Mojave.heic" en nuestra computadora.

```swift
import Foundation
import CoreGraphics

// macOS 10.14 Mojave Required
let url = URL(fileURLWithPath: "/Library/Desktop Pictures/Mojave.heic")
```

Vamos a crear una instancia de `CGImageSource`, copiar sus metadatos y luego iterar sobre cada una de sus etiquetas:

```swift
let source = CGImageSourceCreateWithURL(url as CFURL, nil)!
let metadata = CGImageSourceCopyMetadataAtIndex(source, 0, nil)!
let tags = CGImageMetadataCopyTags(metadata) as! [CGImageMetadataTag]
for tag in tags {
    guard let name = CGImageMetadataTagCopyName(tag),
        let value = CGImageMetadataTagCopyValue(tag)
    else {
        continue
    }

    print(name, value)
}
```
Al ejecutar este código obtenemos dos resultados, el primero `hasXMP` con un valor de `"True"` y el segundo `solar` con un valor no tan obvio:

```
YnBsaXN0MDDRAQJSc2mvEBADDBAUGBwgJCgsMDQ4PEFF1AQFBgcICQoLUWlRelFh
UW8QACNAcO7vOubr3yO/1e+pmkOtXBAB1AQFBgcNDg8LEAEjQFRxqCKOFiAjwCR6
waUkDgHUBAUGBxESEwsQAiNAVZV4BI4c+CPAEP2uFrMcrdQEBQYHFRYXCxADI0BW
tALKmrjwIz/2ObLnx6l21AQFBgcZGhsLEAQjQFfTrJlEjnwjQByrLle1Q0rUBAUG
Bx0eHwsQBSNAWPrrmI0ISCNAKiwhpSRpc9QEBQYHISIjCxAGI0BgJff9KDpyI0BE
NTOsilht1AQFBgclJicLEAcjQGbHdYIVQKojQEq3fAg86lXUBAUGBykqKwsQCCNA
bTGmpC2YRiNAQ2WFOZGjntQEBQYHLS4vCxAJI0BwXfII2B+SI0AmLcjfuC7g1AQF
BgcxMjMLEAojQHCnF6YrsxcjQBS9AVBLTq3UBAUGBzU2NwsQCyNAcTcSnimmjCPA
GP5E0ASXJtQEBQYHOTo7CxAMI0BxgSADjxK2I8AoalieOTyE1AQFBgc9Pj9AEA0j
QHNWsnnMcWIjwEO+oq1pXr8QANQEBQYHQkNEQBAOI0ABZpkFpAcAI8BKYGg/VvMf
1AQFBgdGR0hAEA8jQErBKblRzPgjwEMGElBIUO0ACAALAA4AIQAqACwALgAwADIA
NAA9AEYASABRAFMAXABlAG4AcAB5AIIAiwCNAJYAnwCoAKoAswC8AMUAxwDQANkA
4gDkAO0A9gD/AQEBCgETARwBHgEnATABOQE7AUQBTQFWAVgBYQFqAXMBdQF+AYcB
kAGSAZsBpAGtAa8BuAHBAcMBzAHOAdcB4AHpAesB9AAAAAAAAAIBAAAAAAAAAEkA
AAAAAAAAAAAAAAAAAAH9
```

### Desentrañando el secreto solar

Después de ver el valor de esa variable, la mayoría de nosotros apagaría prontamente nuestra computadora y seguiríamos con nuestro día. Sin embargo, como algunos habrán notado, la cadena de texto se parece mucho a una cadena en [base64](https://en.wikipedia.org/wiki/Base64).

Probemos nuestra hipótesis:

```swift
if name == "solar" {
    let data = Data(base64Encoded: value)!
    print(String(data: data, encoding: .ascii))
}
```

<samp>
bplist00Ò\u{01}\u{02}\u{03}...
</samp>

¿Qué se supone que significa `bplist`? ¿Y lo que sigue? Pensándolo bien, eso parece ser la [firma](https://es.wikipedia.org/wiki/N%C3%BAmero_m%C3%A1gico_(inform%C3%A1tica)) de un [archivo binario de propiedades](https://es.wikipedia.org/wiki/Lista_de_propiedades).

Pidamos ayuda a `PropertyListSerialization`:

```swift
if name == "solar" {
    let data = Data(base64Encoded: value)!
    let propertyList = try PropertyListSerialization
                            .propertyList(from: data,
                                          options: [],
                                          format: nil)
    print(propertyList)
}
```

```
(
    ap = {
        d = 15;
        l = 0;
    };
    si = (
        {
            a = "-0.3427528387535028";
            i = 0;
            z = "270.9334057827345";
        },
        ...
        {
            a = "-38.04743388682423";
            i = 15;
            z = "53.50908581251309";
        }
    )
)
```

_¡Finalmente!_

Al parecer tenemos dos llaves en el nivel más alto:
- La llave `ap` que corresponde a un diccionario de enteros para las llaves `d` y `l`.
- La llave `si` que corresponde a un arreglo de diccionarios con enteros y números punto flotantes. Dentro de las llaves anidadas `i` es la más fácil de entender, incrementa de 0 hasta 15 y representa el índice de la imagen dentro de la secuencia. Es difícil saber a ciencia cierta sin información adicional pero representan la altitud (`a`) y el azimuth (`z`) del sol en la imagen.

### Calculando la posición del sol

Al momento de escribir este artículo, aquellos de nosotros en el hemisferio norte estamos entrando al otoño, con sus días cortos y fríos. Mientras los que se encuentran en el hemisferio sur se preparan para días más largos y calientes. El cambio de estaciones nos recuerda que la duración de un día solar depende de varios factores, tu ubicación y la ubicación del planeta en su órbita alrededor del astro rey.

Tenemos buenas y malas noticias, las buenas son que los astrónomos nos pueden decir, sin lugar a equivocaciones, dónde se encuentra el sol en cualquier tiempo o lugar. Las malas noticias son que los cálculos son algo [complejos](https://es.wikipedia.org/wiki/Declinaci%C3%B3n_solar).

Sin ánimo de engañar al lector, nosotros mismos no lo entendemos y lo que hicimos fue migrar código que encontramos en línea. Después de algunas mejoras, a punta de prueba y error, logramos llegar a una [solución que parece funcionar](https://github.com/NSHipster/DynamicDesktop/blob/master/SolarPosition.playground) (aceptamos tus Pull Requests):

```swift
import Foundation
import CoreLocation

// Apple Park, Cupertino, CA
let location = CLLocation(latitude: 37.3327, longitude: -122.0053)
let time = Date()

let position = solarPosition(for: location, at: time)
let formattedDate = DateFormatter.localizedString(from: time,
                                                    dateStyle: .medium,
                                                    timeStyle: .short)
print("Solar Position on \(formattedDate)")
print("\(position.azimuth)° Az / \(position.elevation)° El")
```

<samp>
Solar Position on Oct 1, 2018 at 12:00
180.73470025840783° Az / 49.27482549913847° El
</samp>

El primero de octubre del 2018, a mediodía, el sol brilla sobre el Apple Park desde el sur, más o menos a la mitad entre el horizonte y encima de nosotros.

Si lleváramos un registro de la posición del sol durante todo un día, obtendríamos una onda sinusoidal, similar a la que se observa en la carátula llamada "Solar" del Apple Watch.

{% asset solar-position-watch-faces.jpg %}

### Cara a cara con XMP

Dejando de lado la clase de astronomía por un rato, enfoquémonos en algo menos emocionante, el uso de XML como la base de estándares para metadatos.

Hace algunos párrafos hablámos de la llave `hasXMP`, sip _ésa llave_. La Plataforma Extensible de Metadatos, conocida por sus siglas en inglés como <abbr title="Extensible Metadata Platform">XMP</abbr>, es un tipo de lenguaje de marcado para representar metadatos en archivos. Salta la pregunta obvia, ¿y cómo se ve? (No apto para aquellos con problemas cardíacos):

```swift
let xmpData = CGImageMetadataCreateXMPData(metadata, nil)
let xmp = String(data: xmpData as! Data, encoding: .utf8)!
print(xmp)
```

```xml
<x:xmpmeta xmlns:x="adobe:ns:meta/" x:xmptk="XMP Core 5.4.0">
   <rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
      <rdf:Description rdf:about=""
            xmlns:apple_desktop="http://ns.apple.com/namespace/1.0/">
         <apple_desktop:solar>
            <!-- (Base64-Encoded Metadata) -->
        </apple_desktop:solar>
      </rdf:Description>
   </rdf:RDF>
</x:xmpmeta>
```

_Un asco._

No todo es negativo, podemos rescatar de nuestra misión de excavación la declaración del `namespace` "apple_desktop", punto importante que nos indica que ocupamos el nombre para las etiquetas de nuestros XMLs.

Manos a la obra.

## Construyamos nuestro Escritorio Dinámico

Iniciemos con un modelo que represente un Escritorio Dinámico:

```swift
struct DynamicDesktop {
    let images: [Image]

    struct Image {
        let cgImage: CGImage
        let metadata: Metadata

        struct Metadata: Codable {
            let index: Int
            let altitude: Double
            let azimuth: Double

            private enum CodingKeys: String, CodingKey {
                case index = "i"
                case altitude = "a"
                case azimuth = "z"
            }
        }
    }
}
```

Cada Escritorio Dinámico está compuesto de una secuencia de imágenes, cada una de ellas tiene datos almacenados en un objeto de tipo `CGImage` y los metadatos cubiertos anteriormente. Implementamos `Codable` en la declaración de nuestro model `Metadata` para indicar al compilador que genere (o sintetice) automáticamente la conformación al protocolo. Aprovecharemos lo anterior cuando tengamos que generar la lista binaria de propiedades en Base64.

### Creando un destino para la imagen

Vamos a crear una instancia de `CGImageDestination` especificando la ruta final como destino, usando `heic` como el formato y el total de imágenes de entrada será igual a la cantidad de imágenes que vayamos a incluir. 

```swift
guard let imageDestination = CGImageDestinationCreateWithURL(
                                outputURL as CFURL,
                                AVFileType.heic as CFString,
                                dynamicDesktop.images.count,
                                nil
                             )
else {
    fatalError("Error creating image destination")
}
```

Seguidamente, usamos el método `enumerated()` para iterar sobre cada una de las imágenes de nuestro objeto `dynamicDesktop`. Vamos a ocupar el índice para cada iteración, para poder establecer los medatatos de la primera imagen.

```swift
for (index, image) in dynamicDesktop.images.enumerated() {
    if index == 0 {
        let imageMetadata = CGImageMetadataCreateMutable()
        guard let tag = CGImageMetadataTagCreate(
                            "http://ns.apple.com/namespace/1.0/" as CFString,
                            "apple_desktop" as CFString,
                            "solar" as CFString,
                            .string,
                            try! dynamicDesktop.base64EncodedMetadata() as CFString
                        ),
            CGImageMetadataSetTagWithPath(
                imageMetadata, nil, "xmp:solar" as CFString, tag
            )
        else {
            fatalError("Error creating image metadata")
        }

        CGImageDestinationAddImageAndMetadata(imageDestination,
                                              image.cgImage,
                                              imageMetadata,
                                              nil)
    } else {
        CGImageDestinationAddImage(imageDestination,
                                   image.cgImage,
                                   nil)
    }
}
```
Ignorando la crudeza de las APIs de Core Graphics, el código es fácil de seguir, la única parte que ocupa ser explicada es la llamada a `CGImageMetadataTagCreate(_:_:_:_:_:)`.

Debido a que existe una discrepancia entre la estructura de los metadatos de las imágenes y los del contenedor y su debida representación, debemos implementar `Encodable` para nuestro modelo `DynamicDesktop`:

```swift
extension DynamicDesktop: Encodable {
    private enum CodingKeys: String, CodingKey {
        case ap, si
    }

    private enum NestedCodingKeys: String, CodingKey {
        case d, l
    }

    func encode(to encoder: Encoder) throws {
        var keyedContainer =
            encoder.container(keyedBy: CodingKeys.self)

        var nestedKeyedContainer =
            keyedContainer.nestedContainer(keyedBy: NestedCodingKeys.self,
                                           forKey: .ap)

        // FIXME: Not sure what `l` and `d` keys indicate
        try nestedKeyedContainer.encode(0, forKey: .l)
        try nestedKeyedContainer.encode(self.images.count, forKey: .d)

        var unkeyedContainer =
            keyedContainer.nestedUnkeyedContainer(forKey: .si)
        for image in self.images {
            try unkeyedContainer.encode(image.metadata)
        }
    }
}
```

Usando lo anterior como base, podemos finalmente implementar el método `base64EncodedMetadata()` de la siguiente manera:

```swift
extension DynamicDesktop {
    func base64EncodedMetadata() throws -> String {
        let encoder = PropertyListEncoder()
        encoder.outputFormat = .binary

        let binaryPropertyListData = try encoder.encode(self)
        return binaryPropertyListData.base64EncodedString()
    }
}
```
Después de escribir los metadatos en todas las imágenes, debemos invocar a `CGImageDestinationFinalize(_:)` para dar fin a nuestra imagen y guardar el resultado en el disco:

```swift
guard CGImageDestinationFinalize(imageDestination) else {
    fatalError("Error finalizing image")
}
```

Si todo funcionó, seremos los orgullosos dueños de un nuevo un Escritorio Dinámico, ¡felicidades!

---

La nueva característica introducida en Mojave, Escritorio Dinámico, nos ha vuelto locos, ojalá y la veamos popularizarse al mismo nivel que lograron los fondos de pantalla de Windows 95. 

La siguiente sección está compuesta de ejercicios para el lector. Sin más demora:

### Generar un Escritorio Dinámico usando fotos

Es increíble que algo tan magno como el movimiento de cuerpos celestiales se pueda reducir a un sistema de ecuaciones con dos entradas: fecha y lugar.

En el ejemplo anterior, ésta información está *hardcoded*, sin embargo la misma información puede ser extraída automáticamente de las imágenes.

De manera predeterminada, las cámaras en los teléfonos modernos almacenan [metadatos Exif](https://es.wikipedia.org/wiki/Exchangeable_image_file_format) cada vez que se captura una foto. Dentro de la información almacenada se encuentra la fecha y las coordenadas GPS del dispositivo en el momento de la captura.

Usando la fecha y la ubicación de los metadatos de la imagen podemos determinar la posición solar y así simplificar el proceso de generar un Escritorio Dinámico desde un conjunto de fotos.

### Usar un iPhone para capturar un time-lapse

Pongamos ese nuevo iPhone XS a trabajar... o mejor dicho, pongamos el iPhone viejo a hacer algo útil mientras ignoramos que debíamos haberlo vendido.

Fijemos el teléfono contra una ventana, conéctalo a un cargador y establece el modo de la cámara a time-lapse y pulsa el botón "Grabar". Usando algunos cuadros clave de tu video podrás crear tu Escritorio Dinámico _a medida_.

Otra opción es usar alguna aplicación como [Skyflow](https://itunes.apple.com/us/app/skyflow-time-lapse-shooting/id937208291?mt=8) que permite tomar fotos usando intervalos definidos.

### Generar un paisaje usando datos GIS

En caso de no poder despegarte del teléfono un día entero (qué triste) o de no tener nada interesante por capturar (también muy triste) siempre existe la opción de crear tu propia realidad (suena más triste de lo que realmente es).

Puedes usar una aplicación como [Terragen](https://planetside.co.uk) para dibujar paisajes realistas en 3D teniendo control preciso sobre la tierra, el sol y el cielo.

Incluso te puedes facilitar la tarea un poco más al descargar mapas de elevación producto de los estudios geológicos tomados por el United States Geological Survey y disponibles en el sitio web de [The National Map](https://viewer.nationalmap.gov/basic/). Usando estos mapas como plantillas podrás acelerar la creación de tu paisaje 3D.

### Descargar Escritorios Dinámicos listos

Si, por el contrario, estás ocupado y no tienes tiempo para hacer fotos bonitas, siempre se puede pagar a alguien que lo haga por ti.

Personalmente nos gusta la aplicación [24 Hour Wallpaper](https://www.jetsoncreative.com/24hourwallpaper/), pero estamos abiertos a sugerencias, [contáctanos en Twitter con la tuya](https://twitter.com/NSHipster/).