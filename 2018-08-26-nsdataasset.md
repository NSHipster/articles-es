---
title: NSDataAsset
author: Mattt
translator: Juan F. Sagasti
category: Cocoa
excerpt: >
  Existen varias maneras de acelerar una petición de red: <br/>
  comprimiendo y haciendo _streaming_, <br/>
  cacheando y precargando; <br/>
  reduciendo y alineando; <br/>
  haciendo pooling y multiplexando la conexión; <br/>
  postponiendo y lanzando en segundo plano. <br/>
  
  Pero existe una estrategia de optimización muy superior a todas ellas: _no  hacer ninguna petición_.
  
status:
  swift: 4.2
---

En la web, la velocidad no es un lujo; es una cuestión de supervivencia.

Estudios recientes sobre experiencia de usuario indican que *cualquier* latencia superior a 400 milisegundos en el tiempo de carga de una web (literalmente lo que dura un parpadeo), puede afectar negativamente a las tasas de conversión y fidelización. Por cada segundo adicional que una web tarda en cargar, el 10% de los usuarios volverán atrás o cerrarán la pestaña.

Para grandes empresas de internet como Google, Amazon y Netflix, un segundo extra aquí y allá puede suponer una pérdida de miles de millones de beneficio anual. Por ello, no es de extrañar que dichas empresas dediquen tanto esfuerzo de ingeniería en hacer que la web sea rápida.

Existen varias maneras de acelerar una petición de red: <br/>
comprimiendo y haciendo _streaming_, <br/>
cacheando y precargando; <br/>
reduciendo y alineando; <br/>
haciendo pooling y multiplexando la conexión; <br/>
postponiendo y lanzando en segundo plano. <br/>
  
Pero existe una estrategia de optimización muy superior a todas ellas: _no  hacer ninguna petición_.

A este respecto, las apps tienen ventaja en comparación a las webs convencionales gracias a que se descargan de antemano. <br/>

Esta semana en NSHipster, veremos cómo aprovechar el _Asset Catalog_ de una manera poco convencional para mejorar la experiencia de carga de tu app.

---

Los Asset Catalogs permiten organizar los recursos de acuerdo a las características del dispositivo. Puedes añadir diferentes versiones de una imagen dependiendo del dispositivo (iPhone, iPad, Apple Watch, Apple TV, Mac), resolución de pantalla (`@2x` / `@3x`) o gama de colores (sRGB / P3). Para otros tipos de recurso, puedes añadir variantes dependiendo de la memoria disponible o de la versión de Metal. Solicita un recurso por su nombre y te devolverá el más apropiado automáticamente.

Aparte de ofrecer una API más apropiada, los Asset Catalogs permiten a las aplicaciones aprovechar [app thinning](https://help.apple.com/xcode/mac/current/#/devbbdc5ce4f), lo cual se traduce en instalaciones más ligeras optimizadas al dispositivo del usuario.

Las imágenes son, de lejos, el recurso más común, pero a partir de iOS 9 y macOS El Capitan, recursos como JSON, XML y otros tipos de archivos de datos se unieron a la fiesta gracias a [`NSDataAsset`](https://developer.apple.com/documentation/uikit/nsdataasset).

## Cómo almacenar y recuperar datos con Asset Catalog

Imaginemos una app para crear paletas digitales de colores.

Para distinguir entre diferentes tonos de gris, podríamos cargar una lista de colores y sus respectivos nombres. Podríamos considerar descargar esto de un servidor durante el arranque de la app, pero eso podría causar una experiencia de usuario mala si [condiciones de red adversas](https://nshipster.com/network-link-conditioner/) bloquean cierta funcionalidad de la aplicación. <br/>

Como esta lista es un conjunto de datos relativamente estático, ¿por qué no incluirlos en el propio bundle en la forma de Asset Catalog?

### Paso 1. Añadir un nuevo conjunto de datos al Asset Catalog

Cuando creas un nuevo proyecto en Xcode, se genera automáticamente un Asset Catalog. Selecciona `Assets.xcassets` desde el organizador del proyecto para abrir el editor del Asset Catalog. Haz click en el icono <kbd>+</kbd> de la esquina inferior izquierda y selecciona "New Data Set"

{% asset add-new-data-set.png %}

Esto creará un subdirectorio de `Assets.xcassets` con la extensión `.dataset`.

> El Finder utiliza ambos bundles como directorios, lo cual facilita inspeccionar y modificar sus contenidos como sea necesario.

### Paso 2. Añadir un archivo de datos

Abre el Finder, navega hasta el archivo de datos y arrástralo hasta el campo vacío reservado para tu conjunto de datos en Xcode.

{% asset asset-catalog-any-any-universal.png %}

Al hacer esto, Xcode copia el archivo al directorio `.dataset` y actualiza el fichero de metadatos`contents.json` con el nombre del archivo y el [Universal Type Identifier](https://es.wikipedia.org/wiki/Uniform_Type_Identifier).

```json
{
  "info": {
    "version": 1,
    "author": "xcode"
  },
  "data": [
    {
      "idiom": "universal",
      "filename": "colors.json",
      "universal-type-identifier": "public.json"
    }
  ]
}
```

### Paso 3. Acceder a los datos usando NSDataAsset

Puedes acceder a los datos del archivo con el siguiente código:

```swift
guard let asset = NSDataAsset(name: "NamedColors") else {
    fatalError("Missing data asset: NamedColors")
}

let data = asset.data
```

En el caso de nuestra app de colores, podríamos llamar a esto desde el método `viewDidLoad()` de un `UIViewController` y construir, con los datos devueltos, un array de modelos que mostrar en una `UITableView`:

```swift
let decoder = JSONDecoder()
self.colors = try! decoder.decode([NamedColor].self, from: asset.data)
```

## Mezclando todo

Los conjuntos de datos no se suelen beneficiar de app thinning de los Asset Catalogs (a la mayoría de archivos JSON, por ejemplo, le importa lo más mínimo qué versión de Metal soporta el dispositivo).

Pero para nuestra app de la paleta de colores, por ejemplo, podríamos ofrecer diferentes listas de colores en aquellos dispositivos con una pantalla con gama de colores amplia.

Para ello, selecciona el recurso en la barra lateral del Asset Catalog y haz click en el menú con la etiqueta _Gamut_ del Attributes Inspector.

{% asset select-color-gamut.png %}

Después de añadir los archivos de datos para cada gama, el contenido del fichero de metadatos`contents.json` debería parecerse a esto:

```json
{
  "info": {
    "version": 1,
    "author": "xcode"
  },
  "data": [
    {
      "idiom": "universal",
      "filename": "colors-srgb.json",
      "universal-type-identifier": "public.json",
      "display-gamut": "sRGB"
    },
    {
      "idiom": "universal",
      "filename": "colors-p3.json",
      "universal-type-identifier": "public.json",
      "display-gamut": "display-P3"
    }
  ]
}
```

## Manteniéndolo fresco 

Guardar y recuperar datos del Asset Catalog es trivial. Lo que es complejo, y mucho más importante, es mantener esos datos actualizados. 

Puedes mantener los datos actualizados usando`curl`, `rsync`, `sftp`, Dropbox, BitTorrent o Filecoin; ejecutando tareas desde un script de consola (ejecútalo desde una Build Phase de Xcode si lo prefieres); haciendo que sea parte de tu `Makefile`, `Rakefile`, `Fastfile` o lo que requiera el sistema de builds de tu elección; delegando la tarea a Jenkins, Travis o al becario con pinta de aburrido; usando alguna integración de Slack o creando un atajo de Siri con el que asombrar a tus compañeros diciendo _"Oye Siri, actualiza los datos antes de que caduquen"_

**Sea como sea que decidas mantener los datos actualizados, asegúrate de que sea algo automatizado y parte del proceso de _release_.

Aquí tienes un ejemplo de un script de consola que podrías ejecutar para descargar datos usando `curl`:

```shell
#!/bin/sh
CURL='/usr/bin/curl'
URL='https://example.com/path/to/data.json'
OUTPUT='./Assets.xcassets/Colors.dataset/data.json'

$CURL -fsSL -o $OUTPUT $URL
```

## Empaquetando

Aunque el Asset Catalog realiza una compresión sin pérdida de las imágenes, no hay ninguna evidencia en la documentación, ayuda de Xcode o sesión de WWDC que indique que se realice una optimización para archivos de datos (al menos, por ahora).

Para datos de, digamos, unos cientos de kilobytes, deberías considerar usar compresión. Esto es especialmente útil para archivos de texto como JSON, CSV, y XML, los cuales suelen comprimirse entre un 60% y 80% respecto al tamaño original.

Podemos añadir compresión a nuestro script de consola anterior haciendo un `pipe` de la salida del `curl` a `gzip` antes de escribir los datos en el archivo:

```shell
#!/bin/sh
CURL='/usr/bin/curl'
GZIP='/usr/bin/gzip'
URL='https://example.com/path/to/data.json'
OUTPUT='./Assets.xcassets/Colors.dataset/data.json.gz'

$CURL -fsSL $URL | $GZIP -c > $OUTPUT
```

Si decides aplicar la compresión, asegúrate de que el campo del `"universal-type-identifier"` lo indica:

```json
{
  "info": {
    "version": 1,
    "author": "xcode"
  },
  "data": [
    {
      "idiom": "universal",
      "filename": "colors.json.gz",
      "universal-type-identifier": "org.gnu.gnu-zip-archive"
    }
  ]
}
```

Es tarea del lado del cliente el descomprimir la información antes de su uso. Si tuvieras un módulo `Gzip`, podrías hacer lo siguiente:

```swift
do {
    let data = try Gzip.decompress(data: asset.data)
} catch {
    fatalError(error.localizedDescription)
}
```

Si haces esto muchas veces en tu app, podrías crear un método de conveniencia en una extensión de `NSDataAsset`:

```swift
extension NSDataAsset {
    func decompressedData() throws -> Data {
        return try Gzip.decompress(data: self.data)
    }
}
```

> Podrías incluso considerar gestionar grandes recursos de datos mediante control de versiones usando [Git Large File Storage (LFS)](https://git-lfs.github.com).

---

Aunque es tentador asumir que todos nuestros usuarios contarán, en todas partes, con una rápida y constante WiFi o LTE, esto no es cierto para todo el mundo, y mucho menos todo el tiempo. 

Dedica un momento a analizar las peticiones de red que hace tu aplicación al arrancar y considera si alguna de ellas puede beneficiarse de contenido precargado. Dejar una primera impresión buena puede marcar la diferencia haciendo que el usuario realice un uso activo de la aplicación y no la borre en cuestión de segundos.

