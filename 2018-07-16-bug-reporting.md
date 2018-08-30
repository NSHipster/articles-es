---
title: Reportes de error
author: Mattt
translator: Juan F. Sagasti
category: Miscellanea
excerpt: >-
  Si alguna vez te han dicho «reporta un Radar» 
  (en inglés _"file a Radar"_) 
  y te has preguntado qué significa eso, 
  este artículo va sobre ello.
hiddenlang: ""
status:
  swift: --
---

«Reporta un _Radar_.» 

Es una frase familiar para todos aquellos que desarrollamos en las plataformas de Apple. 

Es lo que oyes cuando te quejas de un componente de UIKit que no funciona como debería. <br/>
La respuesta que te dan cuando compartes un trozo de documentación ridículamente desactualizada. <br/>
Es la voz en tu cabeza cuando abres Xcode por décima vez en el día. <br/>

Si alguna vez te han dicho «reporta un _Radar_» (en inglés _"file a Radar"_) y te has preguntado qué significa eso, este artículo va sobre ello.

---

Radar es el sistema de seguimiento de errores de Apple. Cualquier empleado allí con un puesto de ingeniero interactúa con este sistema a diario.

Radar se usa para el seguimiento de *features* y *bugs* por igual en software, hardware y prácticamente todo lo demás: documentación, localización, propiedades web e incluso las respuestas que obtienes de Siri. Pero más que la propia aplicación o la mismísima base de datos, Radar es un flujo de trabajo que guía la resolución de problemas desde su reporte hasta su verificación. 

Cuando se crea un Radar, se le asigna un ID único y permanente. Estos IDs son enteros autoincrementales, de forma que te puedes hacer una idea de cuándo fue creado el reporte con solo ver el número. <br/>
(En el momento en el que escribí este artículo, los Radars nuevos tienen IDs de 8 dígitos que empiezan por el 4.)

> Radar.app usa el esquema de URL personalizado `rdar://`  <br/>
> Si un empleado de Apple hace click sobre un enlace [rdar://xxxxxxxx](rdar://30000000) con un ID, le llevará directamente a dicho Radar. <br/>
> En cambio, si hacemos click en este enlace se abrirá un diálogo con el mensaje «No hay ninguna aplicación definida para abrir la URL rdar://30000000.»


## Reportando errores como desarrolladores externos 

Desafortunadamente para todos los que no trabajamos en Apple, no podemos acceder a Radar directamente. En cambio, tenemos que enviar los reportes a través de sistemas indirectos que interactúan con Radar.

### Apple Bug Reporter

[Apple Bug Reporter](https://bugreport.apple.com) es la interfaz primaria de Radar para desarrolladores externos. <br/>

El Bug Reporter fue actualizado recientemente a una web app moderna que recuerda a Mail y a las apps de [iCloud.com](https://icloud.com). Aquellos que recuerden su predecesor estarán de acuerdo en que ha sido una mejora enorme.

{% asset apple-bug-reporter.png %}

Elige el producto relacionado con el problema que estás reportando e introduce un título descriptivo. <br/>
Si es un error, especifica el tipo (Rendimiento, Crash/Bloqueo/Pérdida de datos, UI/Usabilidad, etc.) y la frecuencia con la que puedes reproducirlo. 

Para terminar, redacta la descripción del problema incluyendo un resumen, pasos para reproducirlo, resultado esperado vs. resultado obtenido y la información de configuración y versión de tu sistema.

Esta información se compila en un Radar que los ingenieros valoran, asignan, priorizan y programan.

### Asistente de _Feedback_

Si estás participando en el [Programa de Software Beta de Apple](https://beta.apple.com/sp/betaprogram/welcome) y te topas con un problema en el OS beta, puedes reportarlo de forma alternativa a Radar usando el Assistente de Feedback (encuéntralo en macOS y iOS con Spotlight).

{% asset feedback-assistant.png %}

El Asistente de Feedback proporciona una experiencia más optimizada a la hora de reportar feedback de la plataforma que estés usando. Captura automáticamente un diagnóstico e información diversa del sistema con el fin de diagnosticar el problema que encontraste de forma más precisa.

Mientras que Bug Reporter es la primera opción a la hora de informar de problemas directamente relacionados con tu trabajo, el Asistente de Feedback es, a menudo, una opción más conveniente para cualquier problema detectado en el uso diario.

## Herramientas de reporte de errores de terceros

Cuando los desarrolladores encuentran un problema, suelen hacer algo sobre ello más allá de simplemente quejarse. Este es el motivo de que rellenen un informe de error.

Esta motivación que encontramos en el proceso mismo de informar de errores es la misma que nos empuja a crear herramientas para solucionar los problemas que encontramos.

Aquí tienes algunas herramientas esenciales de la comunidad de desarrollo de Apple para el reporte de errores:

### Open Radar

El principal problema de Radar como desarrollador externo es la falta de transparencia. <br/>
Esto se manifiesta claramente a la hora de saber lo que otros han reportado; no hay manera.
Ocurre muy a menudo que inviertes una gran cantidad de tiempo escribiendo un resumen detallado y creando un caso de prueba reproducible solo para que, sin mayor dilación, el reporte sea cerrado y marcado como duplicado.

[Open Radar](https://openradar.appspot.com), creado por [Tim Burks](https://github.com/timburks), es una base de datos pública de errores reportados a Apple. <br/>
A lo largo de sus muchos años de existencia, se ha acabado convirtiendo en la forma _de facto_ para coordinar nuestros reportes.

{% asset open-radar.png %}

Cuando envías un Radar a Apple, lo ideal es que también [lo hagas con Open Radar](https://openradar.appspot.com/myradars/add) (a menos que sea algo que no deba ser revelado públicamente). Tu contribución ayudará a otros que puedan encontrarse el mismo problema en el futuro.

### Brisk

Aunque la nueva versión de Bug Reporter es bastante agradable de usar, no hay nada como una app nativa.

[Brisk](https://github.com/br1sk/brisk/),
creada por [Keith Smiley](https://github.com/keith), es una app macOS para enviar Radars a través del Bug Reporter de Apple.

Es una app muy completa, con soporte para autenticación de dos factores,
guardado de borradores, duplicado automático de Radars por ID e incluso soporte para abrir URLs de tipo `rdar://`. <br/>
Pero su característica estrella es la capacidad de publicar el reporte también en Open Radar.

{% asset brisk-app.png %}

Para empezar, [descarga la última versión](https://github.com/br1sk/brisk/releases/latest)
desde GitHub o instálala a través de [Homebrew](https://brew.sh) con el siguiente comando:

`$ brew cask install Brisk`

## Consejos para escribir un _buen_ reporte de error

Ahora que sabes cómo escribir un reporte de error, hablemos de cómo escribir uno bueno.


### Un problema, un reporte

No le harás ningún favor a nadie reportando más de un problema en el mismo informe. Cada problemática adicional que plantees aumentará la dificultad de entendimiento del caso por el ingeniero al que se le haya asignado el informe.

Rellena varios Radars en su lugar y referencia por ID los problemas relacionados.

### Elige un título estratégico

Un reporte tiene que hallar su camino hasta un ingeniero antes de que empiece a ser resuelto. La mejor forma de asegurar de que las cosas acaban en manos de la persona correcta es resaltar la información más importante en el título. 

- Para problemas sobre una API en concreto, pon el símbolo completo en cuestión en el título (por ejemplo  `URLSession.dataTaskWithRequest(_:)`).
- Para problemas relacionados con documentación, incluye la lista completa de pasos de navegación en el título (por ejemplo "Foundation > URL Loading System > URLSession > dataTaskWithRequest:").
- Para problemas sobre una app en particular, referencia su nombre, versión y número de build. Esto puedes encontrarlo en el diálogo de «Acerca de» (por ejemplo, "Xcode 10.0 beta (10L176w)").

### No seas hostil

Lo más probable es que no estés en tu mejor momento cuando escribas un reporte de error.

Es inaceptable que no funcione como se espera. <br/>
Perdiste horas intentando depurar el problema. <br/>
Apple ya no se preocupa por la calidad del software. <br/>

Es un asco. Lo entendemos.

Sin embargo, nada de eso arreglará antes el problema. Como mucho, esa hostilidad podría hacer que el ingeniero sienta menos predisposición a la hora a abordar el tema.

Recuerda que hay una persona al otro lado de tu reporte. Practica la [empatía](https://nshipster.es/empathy/).

> Peter Steinberger tiene más consejos sobre escribir buenos reportes de error
> [en este artículo](https://pspdfkit.com/blog/2016/writing-good-bug-reports/) (en inglés).

## Cómo impulsar los reportes de error

Los desarrolladores externos a menudo comparan la experiencia de crear un Radar a enviar un mensaje a un agujero negro. Con miles de Radars entrando cada hora, la probabilidad de que la persona correcta vea tu reporte en un tiempo razonable parece ridícula.

Por suerte, aquí tienes unos cuantos consejos para aumentar tus probabilidades:

### Duplicar Radars existentes

El el proceso de varolación de errores de Apple, se realiza el seguimiento de cada problema por un único Radar (idealmente). Si varios Radars reportan el mismo error subyacente, el más antiguo o más específico es el que se utiliza, mientras que el resto se cierran como duplicados. Esto puede ser frustrante para desarrolladores externos, ya que el cierre de su reporte es único feedback que reciben del problema que estaban teniendo, y más teniendo en cuenta que el Radar original no es visible.

Dicho esto, tener un Radar cerrado como duplicado no siempre es malo. Puedes enviar a sabiendas un Radar duplicado de uno existente para hacer notar que _«Yo también tengo este problema»_ y _«Por favor, arregla esto»_.

A pesar de lo molesto que esto pueda llegar a ser para el ingeniero de Apple responsable de su valoración, una parte de mí no puede evitar proyectar coraje en estos reportes, condenados a sacrificarse con valor en nombre de la calidad del software.

_Semper fidelis_, buggos.

### Twitter

Algunos equipos dentro de Apple son bastante receptivos a los comentarios en Twitter.
Es difícil estar _in-the-loop_ cuando estás en The Loop, así que algunos ingenieros y superiores se ponen a menudo al corriente del _vox populi_.

Debido a las políticas de Apple sobre redes sociales, difícilmente recibirás una respuesta. Pero ten por seguro que tus tweets están apareciendo en alguna búsqueda de Twitter guardada en algún lugar de Cupertino.

### Blogging

Los ingenieros de Apple son desarrolladores como tú y como yo, y muchos prestan atención a lo que escribimos. Además de ser útil para otros desarrolladores, un simple escrito puede ser lo que convenza a ese ingeniero a echar otro vistazo.

---

Hablando desde mi experiencia personal trabajando en Apple, Radar es, de lejos, el mejor sistema de seguimiento de errores que he usado. Es un poco frustrante estar de nuevo de vuelta afuera sabiendo todo lo que nos estamos perdiendo como desarrolladores externos.

A diferencia del software de código abierto, que permite a cualquiera corregir cualquier error que pueda encontrar, la mayoría del software de Apple es propietario; hay muy poco que podamos hacer. Nuestra única opción es presentar un informe de error y esperar lo mejor.

Por suerte, las cosas han mejorado. El nuevo Bug Reporter es excelente y el proceso en sí parece estar buscando una mayor transparecia:

> Cambios esperanzadores en el Bug Reporter de @apple... Sé de gente que ha sido notificada cuando un original está «esperando verificación» y no solo al cerrarse.
> <cite>Dave DeLong ([@davedelong](https://twitter.com/davedelong))
> [via Twitter](https://twitter.com/davedelong/status/1017853619717079040)</cite>

La única forma de seguir mejorando es comunicándonos. <br/>
Así que recuerda, la próxima vez que encuentres algo extraño, _reporta un Radar_.