---
title: UIFieldBehavior
author: Jordan Morgan
translator: Leo Picado
category: Cocoa
excerpt: >
  Como parte del rediseño de iOS en su séptima entrega,
  se dejó de lado el diseño esqueumorfista.
  Tomando su lugar emergió un nuevo paradigma
  en el que a los controles de UI se les permitió
  tener la _sensación_ de objetos físicos y no tan solo
  verse como tales.
status:
  swift: 4.2
---

La marca de la década para iOS ha llegado y se ha ido.
El desarrollo de iOS fue inicialmente un arte en ascenso,
pero ahora se siente usado y desgastado.

Sin embargo, cuando salgo de mi zona de confort de
tablas, etiquetas, botones y similares,
a menudo encuentro partes de Cocoa Touch que
pasé por alto u olvidé por completo.
Cuando sucede, es como tomar un viejo libro del estante;
la anticipación de lo que esconden sus páginas
inevitablemente te inunda.

`UIFieldBehavior` ha sido el último tomo empolvado
por descubrir dentro de UIKit.
Ciertamente usar o hablar de una API concebida para
modelar física compleja para elementos de UI no es
muy común. Pero cuando se ocupa, se _ocupa_, y no hay
alternativa. Y como los abastecedores de lo comúnmente
olvidado o poco usado es un tema excelente para el artículo
de esta semana de NSHipster.

---

Como parte del rediseño de iOS en su sétima entrega,
se dejó de lado el diseño esqueumorfista.
Tomando su lugar emergió un nuevo pardigma,
en el que a los controles de UI se les permitió
tener la _sensación_ de objetos físicos y no tan solo
verse como tales.
Se necesitarían nuevas API para iniciar esta
nueva era de diseño de UI, y así conocimos 
[UIKit Dynamics](https://developer.apple.com/documentation/uikit/animation_and_haptics/uikit_dynamics).

Hay muchos ejemplos del alcance de UIKit Dynamics
através de todo el SO:
la  pantalla de bloqueo que brinca,
las fotos deslizables,
las burbujeantes burbujas de mensajes
entre otras tantas interacciones que usan alguno
de los muchos sabores de la API:

- `UIAttachmentBehavior`:
  Crea una relación entre dos elementos,
  o un artículo y un punto ancla.
- `UICollisionBehavior`:
  Hace que uno o más objetos reboten entre sí
  en vez de traslaparse sin interacción.
- `UIFieldBehavior`:
  Permite que un elemento participe en interacciones físicas.
- `UIGravityBehavior`:
  Aplica una fuerza gravitacional, o jalón.
- `UIPushBehavior`:
  Crea una fuerza instantánea o continua.
- `UISnapBehavior`:
  Produce un movimiento que reduce la intensidad con el tiempo.

Para este artículo daremos un vistazo a `UIFieldBehavior`,
que nuestros buenos amigos de Cupertino usaron para construir
la funcionalidad PiP de las llamadas de FaceTime.

{% asset facetime-picture-in-picture.png alt="FaceTime" title="Imagen: Apple Inc. Todos los derechos reservados." %}

## Entendiendo cómo funcionan los comportamientos de campo

Apple menciona que UIFieldBehavior aplica "física de campo",
¿Pero qué significa eso exactamente?
Afortunadamente, es más fácil de entender de lo que parece.

Hay muchos ejemplos de física de campo en el mundo real,
ya sea el tirón de un imán,
el rebote de un resorte,
La fuerza de la gravedad tirando de ti hacia la tierra.
Usando `UIFieldBehavior`, podemos designar áreas de
nuestras vistas para aplicar efectos de física
cuando interactúen entre en ellas.

El accesible diseño de la API nos permite realizar
efectos complejos de física, sin mucho más que una
llamada a un método fábrica:

```swift
let drag = UIFieldBehavior.dragField()
```

```objc
UIFieldBehavior *drag = [UIFieldBehavior dragField];
```

Una vez que tenemos un campo de fuerza a nuestra
disposición, es cuestión de ubicarlo en la pantalla y
definir el área sobre el cual tendrá influencia.

```swift
drag.position = view.center
drag.region = UIRegion(size: bounds.size)
```

```objc
drag.position = self.view.center;
drag.region = [[UIRegion alloc] initWithSize:self.view.bounds.size];
```

Si necesitas mayor control sobre el
comportamiento de un campo, puedes
configurar sus propiedades `strength` y
`falloff` y cualquier otra propiedad adicional
relacionado a ese tipo de campo.

---

Todos los comportamientos de UIKit Dynamics
requieren ser configurados antes de verlos en
acción y`UIFieldBehavior` no es la excepción.
El flujo se ve así:

- Crear una instancia de un `UIDynamicAnimator` que
  provea el contexto para cualquier animación que
  afecte sus ítems dinámicos.
- Inicializar los comportamientos a usar.
- Agregar las vistas que estarán relacionadas a cada comportamiento.
- Agregar esos comportamientos al *animator* dinámico del primer paso.

```swift
lazy var animator:UIDynamicAnimator = {
    return UIDynamicAnimator(referenceView: view)
}()

let drag = UIFieldBehavior.dragField()

// viewDidLoad:
drag.addItem(anotherView)
animator.addBehavior(drag)
```

```objc
@property (strong, nonatomic, nonnull) UIDynamicAnimator *animator;
@property (strong, nonatomic, nonnull) UIFieldBehavior *drag;

// viewDidLoad:
self.animator = [[UIDynamicAnimator alloc] initWithReferenceView:self.view];
self.drag = [UIFieldBehavior dragField];

[self.drag addItem:self.anotherView];
[self.animator addBehavior:self.drag];
```

{% warning do %}

Recuerda mantener un referencia fuerte al objeto `UIKitDynamicAnimator`. Normalmente esto no es necesario para elementos `UIFieldBehavior` porque el animator se adueña de él una vez que se agrega.

{% endwarning %}

Como ejemplo ilustrativo de `UIFieldBehavior`, veamos cómo FaceTime lo aprovecha para adherir el pequeño visor rectangular de la cámara frontal a las esquinas de la vista.

## Cara a cara con campos elásticos

Durante una llamada de FaceTime se puede mover vista
de tu cámara a una de las esquinas de la pantalla. ¿Cómo
logramos que no solo se mueva fluidamente sino que
también se adhiera?

Una opción es verificar el estado final
de un reconocedor de gestos, calcular hacia qué esquina
se dirige y crear la animación.
El problema con ese enfoque es que perdemos el
«ingrediente secreto» que Apple aplica cuidadosamente a estas
pequeñas interacciones, como la interpolación y la reducción
de la fuerza aplicada a la imagen cuando se posiciona
en una esquina.

Éste es el ejemplo clásico para usar el campo elástico
provisto por `UIFieldBehavior`.  Si pensamos en cómo
funciona un resorte en la vida real, ejerce una fuerza
lineal igual a la cantidad de tensión que se le aplica. Por lo tanto,
si empujamos hacia abajo en un resorte en espiral
esperamos que vuelva a su lugar una vez que lo soltamos.

Esta es también la razón por la cual los campos elásticos
pueden ayudarnos a contener elementos de nuestra interfaz.
Puedes pensar en un ring de boxeo y en cómo sus cuerdas
elásticas mantienen a los boxeadores dentro del
ring. En cambio, con resortes, la cuerda se originaría
en el centro del ring y tiraría hacia cada extremo.

Un campo elástico funciona de una manera parecida. Imaginemos,
por un momento, que los límites de nuestra vista estuvieran
dividos en cuatro rectángulos y tuvieramos estos resortes tirando
hacia los límites de cada uno. Los resortes serían "empujados" hacia
abajo, desde el centro del rectángulo, hacia el borde de su esquina.
Cuando la imagen llega a alguna de las esquinas, el resorte
se "suelta" y nos da el pequeño empujón que buscamos.

{% info do %}

El campo elástico se crea usando la
[Ley de elasticidad de Hooke](https://phys.org/news/2015-02-law.html)
para calcular la cantidad de fuerza que debe aplicarse
a los objetos dentro del campo.

{% endinfo %}

Para encargarnos del efecto de adhesión de la imagen
a cada esquina, podemos hacer algo como esto:

```swift
let scale = CGAffineTransform(scaleX: 0.5, y: 0.5)

for vertical in [\UIEdgeInsets.left,
                 \UIEdgeInsets.right]
{
    for horizontal in [\UIEdgeInsets.top,
                       \UIEdgeInsets.bottom]
    {
        let springField = UIFieldBehavior.springField()
        springField.position =
            CGPoint(x: layoutMargins[keyPath: horizontal],
                    y: layoutMargins[keyPath: vertical])
        springField.region =
            UIRegion(size: view.bounds.size.applying(scale))

        animator.addBehavior(springField)
        springField.addItem(facetimeAvatar)
    }
}
```

```objc
UIFieldBehavior *topLeftCornerField = [UIFieldBehavior springField];

// Esquina superior izquierda
topLeftCornerField.position = CGPointMake(self.layoutMargins.left, self.layoutMargins.top);
topLeftCornerField.region = [[UIRegion alloc] initWithSize:CGSizeMake(self.bounds.size.width/2, self.bounds.size.height/2)];

[self.animator addBehavior:topLeftCornerField];
[self.topLeftCornerField addItem:self.facetimeAvatar];

// Y se sigue con el resto de las esquinas
```

## Resolviendo problemas de física

No siempre es fácil conceptualizar las interacciones de 
campos fuerza invisibles. Afortunadamente, Apple anticipó
esta necesidad y ofrece una solución más o menos estándar
de resolver esto.

Escondido dentro de `UIDynamicAnimator` existe una
propiedad Booleana llamada `debugEnabled` que al
cambiar su valor a `true` dibuja líneas rojas en la interfaz
para visualizar los efectos de los campos y su influencia.
Definitivamente una gran ayuda para poder entender cómo
interactúan entre ellos.

Esta API no está expuesta públicamente, pero se puede
desbloquar usando key-value coding:

```objc
@import UIKit;

#if DEBUG

@interface UIDynamicAnimator (Debugging)
@property (nonatomic, getter=isDebugEnabled) BOOL debugEnabled;
@end

#endif
```

ó

```swift
animator.setValue(true, forKey: "debugEnabled")
```

```objc
[self.animator setValue:@1 forKey:@"debugEnabled"];
```

Aunque crear la categoría requiere un poco más de trabajo,
es la mejor opción, dada la fragilidad del key-value coding.
Nuestro código se puede romper en futuras actualizaciones
de iOS. . El precio de la conveniencia suele salir caro.

Con la depuración activada, pareciera como si cada
esquina tuviera un efecto de resorte asociado.
Sin embargo, al correr nuestra aplicación se revela que
no es suficiente para lograr el efecto que buscamos.

{% asset uidynamicanimator-debug.jpg %}

## Añadiendo comportamientos

Ahondemos en nuestra situación actual. En este momento
tenemos algunos problemas:

1. La imagen puede salir volando fuera de la pantalla,
   ya que solamente nuestros campos elásticos la detienen.
2. Tiende a rotar en círculos.
3. Es un poco lenta.

UIKit Dynamics simula física; quizá demasiado bien.

Afortundamente, podemos mitigar todos estos indeseables
efectos secundarios. Realmente son arreglos triviales, lo
importante es saber el *porqué*.

El primer arreglo es bastante mundano, se soluciona con el
comportamiento más fácilmente entendido dentro de UIKit Dynamics:
las colisiones. Para mejorar la reacción de nuestra imagen
cuando interactúa con un campo elástico ocupamos definir
sus propiedades físicas de una manera más explícita.
Idóneamente se comportaría como lo haría en la vida real,
siendo sujeto a las fuerzas de gravedad y fricción para
disminuir su movimiento.

`UIDynamicItemBehavior`es ideal para estos casos. Nos
permite añadir propiedades físicas, a lo que de otra forma serían,
vistas abstractas que interactúan con un motor físico. Aunque
UIKit nos provee de valores predeterminados para cada una
de las propiedades que interactúan con el motor físico, no
representan nuestras necesidades actuales y UIKit Dynamics
casi siempre se clasifica como un "caso particular".

Resulta fácil ver cómo la falta de una API como este sería
muy problemático. Resultaría muy dificil modelar un empujón, un tirón o
velocidad sin tener cómo especificar la densidad o masa del objeto.

```swift
let avatarPhysicalProperties= UIDynamicItemBehavior(items: [facetimeAvatar])
avatarPhysicalProperties.allowsRotation = false
avatarPhysicalProperties.resistance = 8
avatarPhysicalProperties.density = 0.02
```

```objc
UIDynamicItemBehavior *avatarPhysicalProperties = [[UIDynamicItemBehavior alloc] initWithItems:@[self.facetimeAvatar]];
avatarPhysicalProperties.allowsRotation = NO;
avatarPhysicalProperties.resistance = 8;
avatarPhysicalProperties.density = 0.02;
```

El comportamiento de la imagen ahora se asemeja más
a la realidad: se desacelera un poco después de ser empujado
por un campo elástico. La opciones de personalización de `UIDynamicItemBehavior`
son impresionantes, incluyen soporte para elasticidad, embestidas 
y anclajes, todo lo que se ocupa para seguir afinando la interacción
hasta que se sienta correcto.

Además, tiene soporte para agregar velocidad linear o angular
a un objeto, lo cual nos viene muy bien para finalizar. Démosle un
pequeño empujón al final al reconocedor gestual para enviarlo
hacia la esquina más cercana y que el campo elástico termine
el trabajo:

```swift
// Inside a switch for a gesture recognizer...
case .canceled, .ended:
let velocity = panGesture.velocity(in: view)
facetimeAvatarBehavior.addLinearVelocity(velocity, for: facetimeAvatar)
```

```objc
// Inside a switch for a gesture recognizer...
case UIGestureRecognizerStateCancelled:
case UIGestureRecognizerStateEnded:
{
CGPoint velocity = [panGesture velocityInView:self.view];
[facetimeAvatarBehavior addLinearVelocity:velocity forItem:self.facetimeAvatar];
break;
}
```

Ya casi acabamos con nuestra imitación de la interfaz de FaceTime.

Para completar la experiencia, debemos tener en cuenta qué
debe hacer nuestro avatar de FaceTime cuando llegue a alguna
esquina de la vista del `animator`. Queremos que se quede
contenido dentro de los límites, pero actualmente nada lo
detiene para evitar que se salga de la pantalla. Por suerte,
UIKit Dynamics nos ofrece un comportamiento idóneo
para estas ocasiones: `UICollisionBehavior`.

Gracias a su consitente API, seguiremos el mismo patrón
que usamos para el resto comportamientos de UIKit Dynamics
a la hora de crear la colisión:

```swift
let parentViewBoundsCollision = UICollisionBehavior(items: [facetimeAvatar])
parentViewBoundsCollision.translatesReferenceBoundsIntoBoundary = true
```

```objc
UICollisionBehavior *parentViewBoundsCollision = [[UICollisionBehavior alloc] initWithItems:@[self.facetimeAvatar]];
parentViewBoundsCollision.translatesReferenceBoundsIntoBoundary = YES;
```

Es importante ver el efecto de `translatesReferenceBoundsIntoBoundary`.
Cuando el valor está en `true` toma los límites de nuestro `animator` como
los límites para la colisión. Recordemos que este fue el primer paso
cuando armamos nuestra pila de comportamientos dinámicos:

```swift
lazy var animator:UIDynamicAnimator = {
    return UIDynamicAnimator(referenceView: view)
}()
```

```objc
self.animator = [[UIDynamicAnimator alloc] initWithReferenceView:self.view];
```

Anidemos varios comportamientos para que funcionen como uno y deleitémonos con nuestro trabajo:

<video preload="none" src="{% asset uifieldbehavior-demo.mp4 @path %}" poster="{% asset uifieldbehavior-demo.png @path %}" width="320" controls></video>

<br/>

Es sencillo cambiar el comportamiento de las «esquinas pegajosas», ya que `UIFieldBehavior` tiene muchos más campos además del elástico. Un
posible experimento es remplazar el campo elástico con uno magnético
o hacer que la imagen gire constantemente alrededor de un punto dado.

---

iOS ya dejó de lado el diseño esqueumorfista y la experiencia
de usuario ha mejorado bastante como resultado de ello. Ya no
ocupamos de un [fieltro verde]({% asset game-center-felt.jpg @path %}) para indicarle a nuestros usuarios que el Game Center representa juegos y como manejarlos.

UIKit Dynamics es una muestra de este cambio. Los componentes de UI ya no son meras ilustraciones que se asemejan a elementos de la vida real, sino que se comportan como lo hacen en realidad.

Dejar de lado esta capa através de todo el SO abrió la puerta
para que UIKit Dynamics conectara nuestra expectativas sobre
cómo los elementos deben reaccionar a nuestras acciones. Estas
conexiones pueden parecer inconsecuentes al inicio, pero si no
estuvieran ahí, sería fácil darse cuenta que algo falta.

UIKit Dynamics nos ofrece muchos sabores en cuanto a comportamientos
físicos y sus campos son, tal vez, unos de los más versátiles e
interesantes. La próxima vez que vea una oportunidad de crear una
conexión en su aplicación, `UIFieldBehavior` puede darle el empujón
inicial.