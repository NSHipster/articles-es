---
title: Rastreo de Contactos
author: Mattt
translator: Nicolás García
category: Miscellaneous
excerpt: >-
  Apple y Google anunciaron una iniciativa en conjunto 
  para implementar la funcionalidad de 
  rastreo de contactos 
  para miles de millones de dispositivos con iOS o Android 
  en los próximos meses.
  En este articulo, 
  daremos un primer vistazo a estas especificaciones,
  particularmente al framework ExposureNotification,
  propuesto por Apple,
  en un esfuerzo por anticipar cómo se verá todo esto en
  práctica.
status:
  swift: 5.2
revisions:
  "2020-04-13": First Publication
  "2020-04-29": Updated for Exposure Notification v1.2
---

> Más vale prevenir que lamentar.

Una intervención temprana es una de las estrategias más efectivas para el trato de enfermedades. Lo cual no solo se aplica al cuerpo humano, sino también a la sociedad en su conjunto. Es por eso que funcionarios de la salud pública usan el rastreo de contactos como su primera línea de defensa contra la propagación de enfermedades contagiosas en una población.

Estos días, mucho escuchamos acerca del rastreo de contactos, pero la técnica ha sido utilizada durante décadas. Lo que ha cambiado es que, gracias a la ubicuidad de los dispositivos electrónicos personales, podemos automatizar lo que antes era - hasta hoy - un proceso manual e intensivo. Así como un "computador" solía ser un puesto de trabajo para humanos, el rol the un "rastreador de contactos" podría pronto ser realizado principalmente por aplicaciones.

El 10 de abril, Apple y Google [anunciaron][apple press release] en conjunto una iniciativa para desplegar, en los próximos meses, la funcionalidad de rastreo de contactos en billones de dispositivos Android e iOS. Como parte de este anuncio, las compañías compartieron bosquejos de las especificaciones para la [criptografía][cryptography specification], [hardware][hardware specification] y [software][software specification] involucrados en su solución propuesta.

En este artículo, vamos a dar un primer vistazo a estas especificaciones - particularmente al framework `ExposureNotification` propuesto por Apple - y usaremos lo aprendido para anticipar como se verá esto una vez puesto en práctica.

{% warning %}

El 29 de abril, Apple lanzó iOS 13.5 beta 1, el cual incluye el primer release del framework `ExposureNotification` (previamente `ContactTracing`). El contenido de este artículo ha sido actualizado para reflejar estos cambios.

{% endwarning %}

* * *

## ¿Qué es el rastreo de contactos?

<dfn>El rastreo de contactos</dfn> es una técnica utilizada por los funcionarios de salud pública para identificar a las personas que han sido expuestas a una enfermedad infecciosa para frenar la propagación de esa enfermedad dentro de una población.

Cuando un paciente ingresa en un hospital, diagnosticado con una nueva enfermedad transmisible, es entrevistado por trabajadores de la salud para saber con quién ha interactuado recientemente. Cualquier contacto que haya interactuado con el paciente es evaluado, y si se le diagnostica la enfermedad, el proceso se repite con sus contactos recientes conocidos.

El rastreo de contactos interrumpe la cadena de transmisión. Le da a la gente la oportunidad de aislarse antes de infectar a otros y de buscar tratamiento antes de que presenten los síntomas. También permite a aquellos que toman las decisiones, realizar, de manera informada, recomendaciones y decisiones políticas sobre medidas adicionales a tomar.

Si se comienza temprano y se actúa rápidamente, el rastreo de contactos brinda la posibilidad de combatir un brote antes de que se salga de control.

Desafortunadamente, esta vez no fuimos tan afortunados.

Con más de un millón de casos confirmados de <abbr>COVID-19</abbr> en todo el mundo, muchas regiones ya han pasado el punto en el cual el rastreo de contactos es práctico. Pero eso no quiere decir que no pueda desempeñar un papel esencial en las próximas semanas y meses.

## "Solo Apple <ins>(y Google)</ins> pueden hacerlo"

Desde el brote, varios [gobiernos](https://www.pepp-pt.org) y [académicos](https://github.com/DP-3T/documents) han propuesto estándares para el rastreo de contactos. Pero el desarrollo más significativo hasta ahora fue ayer con el anuncio de Apple y Google de una iniciativa en conjunto.

De acuerdo al <abbr title="United Kingdom National Health Service">NHS</abbr>, cerca del 60% de adultos de una población necesitarían participar para que el rastreo de contactos digital sea efectivo. Investigadores de las instituciones mencionadas anteriormente han notado que los límites impuestos por iOS para las aplicaciones de terceros hacen que este nivel de participación sea poco probable. 

Por un lado, es extraño felicitar a Apple por dar un paso adelante en la resolución de un problema que crearon ellos mismos. Pero, sin duda, este acuerdo es algo para celebrar. Y no es exagerado decir que esto no sería posible sin su ayuda.

## ¿Qué proponen Apple and Google como solución?

A grandes rasgos, Apple y Google están proponiendo un estándar común sobre como los dispositivos electrónicos personales (teléfonos, tablets y relojes) pueden automatizar el proceso de rastreo de contactos.

En lugar de que los trabajadores de salud deban llamar a los contactos por teléfono, un proceso que puede llevar horas o incluso días, el sistema propuesto podría identificar cada contacto reciente y notificarlos a todos al momento de tener un diagnóstico confirmado y positivo.

{% info %}

[Esta infografía][google presentation] del blog de Google, la cual anuncia la asociación, proporciona una buena explicación de las tecnologías involucradas.

{% endinfo %}

El CEO de Apple, Tim Cook, promete que ["el rastreo de contactos puede ayudar a retrasar la propagación de COVID-19 y puede hacerse sin comprometer la privacidad del usuario"](https://twitter.com/tim_cook/status/1248657931433693184). Las especificaciones que acompañan el anuncio muestran cómo eso es posible.

Revisaremos sus componentes uno a uno, comenzando con la [criptografía][cryptography specification] (derivación y rotación de claves), seguido por el [hardware][hardware specification] (Bluetooth) y el [software][software specification] (app).

### Criptografía

Cuando instalas una aplicación y la abres por primera vez, el framework `ExposureNotification` muestra un mensaje solicitando permiso para permitir el rastreo de contactos en el dispositivo.

Si el usuario acepta, el framework genera un número criptográfico aleatorio de 32 bytes que actuará como la <dfn>Llave de rastreo</dfn> del dispositivo. La Llave de rastreo es mantenida en secreto, nunca dejando el dispositivo.

{% info %}

Si el concepto de "datos binarios" le parece desalentador o sin sentido, puede ayudar ver algunos ejemplos de cómo esa información puede codificarse en una forma legible para los humanos.

32 bytes de datos binarios pueden ser representados por una cadena de 44 caracteres de longitud [codificada en Base64][base64] o una cadena de 64 dígitos [hexadecimales][hexadecimal]. Puede generarlos usted mismo desde la línea de comandos utilizando los siguientes comandos:

```terminal
$ head -c32 < /dev/urandom | xxd -p -c 64
211ad682549d92fbb6cd5dc42be5121b22f8864b3a7e93cedb9c43c83332440d

$ head -c32 < /dev/urandom | base64
2pNDyj5LSr0GGi1IL2VOvsovBwmG4Yp5YYP7leg928Y=
```

16 bytes de datos binarios también se podrían representar en Base64 o en formato hexadecimal, pero es más común y conveniente usar un [<abbr title="Universally Unique Identifier">UUID</abbr>][rfc4122].

```terminal
$ uuidgen
33F1C4D5-3F1C-4FF0-A05E-A267FAB237CB
```

{% endinfo %}

Cada 24 horas, el dispositivo toma la Llave de Rastreo y el número del día (0, 1, 2, ...) y usa [<abbr title="HMAC-based Extract-and-Expand Key Derivation Function">HKDF</abbr>][rfc5869] para derivar una <dfn><del>Llave de Rastreo diaria</del> <ins>Llave de Exposición Temporal</ins></dfn> de 16 bytes. Estas llaves se quedan en el dispositivo, a menos que des tu consentimiento para compartirlas. 

Cada 15 minutos, el dispositivo toma la _Llave de Exposición Temporal_ y el número de intervalos de 10 minutos a contar desde el comienzo del día (0 – 143), y usa [<abbr title="Keyed-Hashing for Message Authentication">HMAC</abbr>][rfc2104] para generar un nuevo <dfn>Identificador de Proximidad Rotativo</dfn> de 16 bytes. Este identificador es transmitido desde el dispositivo usando [Bluetooth <abbr title="Low Energy">LE</abbr>][ble].

Si alguien usando una aplicación de rastreo de contactos recibe un diagnóstico positivo, la autoridad sanitaria central solicita sus _Llaves de Exposición Temporal_ por el periodo de tiempo que fueron contagiosos. Si el paciente da su consentimiento, esas llaves son entonces añadidas a la base de datos de la autoridad sanitaria como <dfn>Llaves de Diagnóstico Positivo</dfn>. Esas llaves son compartidas con otros dispositivos para determinar si han tenido contacto durante ese período de tiempo. 

{% info %}

La [Especificación Criptográfica para Rastreo de Contactos][cryptography specification] es concisa, escrita claramente y notablemente accesible. A cualquiera para quién el nombre _[Diffie–Hellman][diffie–hellman]_ le es familiar se le recomienda darle una lectura.

{% endinfo %}

### Hardware

Bluetooth organiza las comunicaciones entre dispositivos bajo el concepto de <dfn>servicios</dfn>.

Un servicio describe un conjunto de características para realizar una tarea en particular. Un dispositivo puede comunicarse con múltiples servicios en el transcurso de su operación. Muchas definiciones de servicio son [estandarizadas](https://www.bluetooth.com/specifications/gatt/) para que los dispositivos que hacen el mismo tipo de cosas se comuniquen de la misma manera.

Por ejemplo, un monitor inalámbrico de frecuencia cardíaca que usa Bluetooth para comunicarse con su teléfono tendría un perfil que contiene dos servicios: un servicio primario de frecuencia cardíaca y un servicio secundario de batería.

El estándar de rastreo de contactos de Apple y Google define un nuevo servicio de Detección de Contactos (en inglés: Contact Detection).

Cuando una aplicación de rastreo de contratos se está ejecutando (ya sea en primer o segundo plano), actúa como un <dfn>periférico</dfn>, transmitiendo su soporte para el servicio de Detección de Contactos a cualquier otro dispositivo dentro de su alcance. El _Identificador de Proximidad Rotativo_, generado cada 15 minutos, se envía en el paquete transmitido junto con el UUID de 16 bits del servicio.

Aquí un ejemplo de código para hacer esto desde un dispositivo iOS usando el [framework CoreBluetooth][core bluetooth]:

```swift
import CoreBluetooth

// UUID del servicio Detección de Contactos
let serviceUUID = CBUUID(string: "FD6F")

// Identificador de Proximidad Rotativo
let identifier: Data = <#...#> // 16 bytes

let peripheralManager = CBPeripheralManager()

let advertisementData: [String: Any] = [
    CBAdvertisementDataServiceUUIDsKey: [serviceUUID]
    CBAdvertisementDataServiceDataKey: identifier
]

peripheralManager.startAdvertising(advertisementData)
```
Al mismo tiempo que el dispositivo transmite actuando como un periférico, busca _Identificadores de Proximidad Rotativos_ de otros dispositivos. Nuevamente, así es como podría hacer esto en iOS usando `CoreBluetooth`:

<aside class="parenthetical">

Condicionalmente, según las operaciones permitidas por el sistema.

</aside>

```swift
let delegate: CBCentralManagerDelegate = <#...#>
let centralManager = CBCentralManager(delegate: delegate, queue: .main)
centralManager.scanForPeripherals(withServices: [serviceUUID], options: [:])

extension <#DelegateClass#>: CBCentralManagerDelegate {
  func centralManager(_ central: CBCentralManager,
                      didDiscover peripheral: CBPeripheral,
                      advertisementData: [String : Any],
                      rssi RSSI: NSNumber)
  {
      let identifier = advertisementData[CBAdvertisementDataServiceDataKey] as! Data
      <#...#>
  }
}
```

Bluetooth es una tecnología casi ideal para el rastreo de contactos. Está en todos los teléfonos inteligentes de consumo. Opera con bajo requerimiento de energía, lo que le permite funcionar continuamente sin agotar su batería. Y _justamente_ sucede que tiene un rango de transmisión que se aproxima a la proximidad física requerida para la transmisión aérea de enfermedades infecciosas. Esta última cualidad es lo que permite realizar el rastreo de contactos sin recurrir a los datos de ubicación.

<aside class="parenthetical">

Como señalamos en [un artículo anterior](/device-identifiers/#fingerprinting-in-today-ios), los individuos dentro de una población puede ser identificados por tan solo cuatro coordenadas marcadas en el tiempo.

</aside>

### Software

Su dispositivo almacenará cualquier _Identificador de Proximidad Rotativo_ que descubra y periódicamente los verificará contra una lista de _Llaves de Diagnóstico Positivo_ enviadas por la autoridad sanitaria.

Cada _Llave de Diagnóstico Positivo_ corresponde a la _Llave de Exposición Temporal_ de otra persona. Con ella podemos derivar todos los posibles _Identificadores de Proximidad Rotativos_ que podría transmitir en el transcurso de ese día (utilizando el mismo algoritmo <abbr title = "Key-Hashing for Message Authentication">HMAC</abbr> que utilizamos para derivar nuestros propios _Identificadores de Proximidad Rotativos_). Si se encuentran coincidencias entre la lista de _Identificadores de Proximidad Rotativos_ de su dispositivo, significa que puede haber estado en contacto con una persona infectada.

Basta decir que el rastreo de contactos digital es realmente difícil de hacer. Dada la importancia de hacerlo bien,
tanto en términos de obtener resultados precisos como de preservar la privacidad, Apple y Google están proporcionando SDKs para que los desarrolladores de aplicaciones los usen para iOS y Android, respectivamente.

Todos los detalles que discutimos sobre criptografía y Bluetooth son administrados por el framework. Lo único que necesitamos hacer como desarrolladores es comunicarse con el usuario, específicamente, solicitando su permiso para comenzar el rastreo de contactos y notificándoles en caso de un diagnóstico positivo.

## ExposureNotification

El 10 de abril, cuando Apple anunció el framework `ContactTracing`, todo lo que teníamos que seguir eran algunos 
encabezados Objective-C anotados. Pero, desde la primera beta pública de iOS 13.5, tenemos ahora la [documentación oficial](https://developer.apple.com/documentation/exposurenotification) bajo el nombre de: `ExposureNotification`.


### Calculando el riesgo de exposición (Risk of Exposure)

Una aplicación de rastreo de contactos regularmente obtiene nuevas _Llaves de Diagnóstico Positivo_ de la autoridad sanitaria central. Luego verifica esas llaves contra los _Identificadores de Proximidad Rotativos_ del dispositivo. Cualquier coincidencia indicaría un posible riesgo de exposición. 

En la primera versión de `ContactTracing`, todo lo que podía aprender sobre un match positivo era cuánto tiempo estuvo expuesto _(en incrementos de 5 minutos)_ y cuando ocurrió el contacto _(con un nivel de precisión no especificado)_. Si bien aquí podríamos aplaudir el nivel de protección de privacidad, no nos ofrece mucho a nivel de información procesable. Dependiendo del individuo, una notificación push diciendo "Estuvo expuesto durante 5-10 minutos en algún momento hace 3 días" podría justificar una visita al hospital o no crear mas preocupación que una llamada perdida.

Con `ExposureNotification`, obtiene mucha más información, incluyendo:

- Días desde el último incidente de exposición
- Duración acumulativa de la exposición (limitada a 30 minutos)
- Atenuación mínima de la intensidad de la señal Bluetooth _(Potencia de transmisión - RSSI)_, lo que puede decirle cuán cerca estuvo de la persona infectada
- El Riesgo de transmisión, el cual es un valor definido por la aplicación que puede basarse en síntomas, nivel de 
verificación del diagnóstico u otra determinación de la aplicación o una autoridad de salud

Por cada caso de exposición, un objeto [`ENExposureInfo`](https://developer.apple.com/documentation/exposurenotification/enexposureinfo) provee toda la información mencionada anteriormente, junto a un puntaje de riesgo general
_([de 1 a 8](https://developer.apple.com/documentation/exposurenotification/enrisklevel))_, el cual es calculado desde [los valores de peso asignados a cada factor](https://developer.apple.com/documentation/exposurenotification/enexposureconfiguration), de acuerdo a esta ecuación:

<figure>
{% asset contact-tracing-equation.svg width="100%" %}

<figcaption hidden>
<em>S</em> is a score,
<em>W</em> is a weighting,
<em>r</em> is risk,
<em>d</em> is days since exposure,
<em>t</em> is duration of exposure,
<em>ɑ</em> is Bluetooth signal strength attenuation</em>
</figcaption>
</figure>

Apple provee este ejemplo en su [documentation PDF del framework](https://covid19-static.cdn-apple.com/applications/covid19/current/static/contact-tracing/pdf/ExposureNotification-FrameworkDocumentationv1.2.pdf):

{% asset contact-tracing-example-equation.png %}

### Administrando permisos y divulgaciones

El mayor desafío que encontramos en la API del framework Contact Tracing original era lidiar con todos sus _completion handlers_. La mayor parte de la funcionalidad era proporcionada mediante APIs asíncronas; sin una forma para [componer](/optional-throws-result-async-await/) estas operaciones, fácilmente puede verse anidando 4 o 5 _closures_, indentados en un lejano sector de su editor. 

<aside class="parenthetical">

Si alguna vez fue necesario [async/await](https://gist.github.com/lattner/429b9070918248274f25b714dcfc7619) en Swift, este es el momento.

</aside>

Afortunadamente, la última versión de `ExposureNotification` incluye una nueva clase
[`ENManager`](https://developer.apple.com/documentation/exposurenotification/enmanager), la cual simplifica en gran parte todo este manejo de estados asíncronos.

```swift
let manager = ENManager()
manager.activate { error in 
    guard error == nil else { <#...#> }

    manager.setExposureNotificationEnabled(true) { error in
        guard error == nil else { <#...#> }

		// la app está ahora transmitiendo y monitoreando identificadores de rastreo 
    }
}
```

* * *

## Rastreando un camino de regreso a la vida normal

Muchos hemos estado refugiados en casa durante semanas, sino meses. Hasta que se desarrolle una vacuna y esté ampliamente disponible, esta es la estrategia más efectiva que tenemos para detener la propagación de la enfermedad.

Pero los expertos están diciendo que podríamos estar a entre 9 y 18 meses de tener una vacuna. _"¿Qué haremos hasta ese entonces?"_

Al menos aquí en los Estados Unidos, aún no tenemos un plan nacional para regresar a la normalidad, entonces es dificil de decir. Lo que sí sabemos es que no será fácil, y no va todo a venir de una vez.

Una vez que la tasa de nuevos contagios se estabilice, nuestro foco será contener nuevos brotes en las comunidades. Y, para ese fin, la tecnología de rastreo de contactos puede desempeñar un rol crucial.

Desde una perspectiva técnica, la propuesta de Apple y Google nos da la razón para creer que _podemos_ hacer rastreo de contactos sin comprometer la privacidad. Sin embargo, cuanta fé pongan en esta solución depende, en primer lugar, de cuánto confíen en estas compañías y en nuestros gobiernos.

Personalmente, quedo cautelosamente optimista. El compromiso de Apple con la privacidad ha sido uno de sus mayores activos, y eso es ahora más importante que nunca.

[skip]: #tech-talk "Jump to code discussion"
[apple press release]: https://www.apple.com/newsroom/2020/04/apple-and-google-partner-on-covid-19-contact-tracing-technology/
[cryptography specification]: https://covid19-static.cdn-apple.com/applications/covid19/current/static/contact-tracing/pdf/ExposureNotification-CryptographySpecificationv1.2.pdf
[hardware specification]: https://covid19-static.cdn-apple.com/applications/covid19/current/static/contact-tracing/pdf/ExposureNotification-BluetoothSpecificationv1.2.pdf
[software specification]: https://covid19-static.cdn-apple.com/applications/covid19/current/static/contact-tracing/pdf/ExposureNotification-FrameworkDocumentationv1.2.pdf
[diffie–hellman]: https://en.wikipedia.org/wiki/Diffie–Hellman_key_exchange
[rfc2104]: https://tools.ietf.org/html/rfc2104 "HMAC: Keyed-Hashing for Message Authentication"
[rfc5869]: https://tools.ietf.org/html/rfc5869 "HMAC-based Extract-and-Expand Key Derivation Function (HKDF)"
[rfc4122]: https://tools.ietf.org/html/rfc4122 "A Universally Unique IDentifier (UUID) URN Namespace"
[hexadecimal]: https://en.wikipedia.org/wiki/Hexadecimal#Binary_conversion
[base64]: https://en.wikipedia.org/wiki/Base64
[ble]: https://en.wikipedia.org/wiki/Bluetooth_Low_Energy
[google presentation]: https://blog.google/documents/57/Overview_of_COVID-19_Contact_Tracing_Using_BLE.pdf
[core bluetooth]: https://developer.apple.com/documentation/corebluetooth
[android contact tracing]: https://blog.google/documents/55/Android_Contact_Tracing_API.pdf
[swift interface]: https://github.com/NSHipster/ContactTracing-Framework-Interface/blob/master/ContactTracing.swift
[swift docs]: https://contact-tracing-documentation.nshipster.com
[delegate pattern]: https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/DelegatesandDataSources/DelegatesandDataSources.html
[cllocationmanager]: https://developer.apple.com/documentation/corelocation/cllocationmanager
