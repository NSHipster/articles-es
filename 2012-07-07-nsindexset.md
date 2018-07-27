---
title: NSIndexSet
author: Mattt
translator: Juan F. Sagasti
category: Cocoa
excerpt:
    NSIndexSet (igual que su contraparte mutable, NSMutableIndexSet)
    es una colección ordenada de enteros únicos sin signo (`UInt`).
    Piensa en ella como un `NSRange` que soporta series no-contiguas.
    Sus operaciones para encontrar índices en rangos o realizar la
    intersección de sets son tremendamente rápidas, e implementa 
    todos los métodos de conveniencia que cabría esperar 
    de una Foundation class. 
status:
    swift: 2.0
    reviewed: September 8, 2015
---

`NSIndexSet` (igual que su contraparte mutable, NSMutableIndexSet) es una colección ordenada de enteros únicos sin signo (`UInt`). Piensa en ella como un `NSRange` que soporta series no-contiguas. Sus operaciones para encontrar índices en rangos o realizar la intersecciones entre sets son tremendamente rápidas, e implementa todos los métodos de conveniencia que cabría esperar de una clase Foundation. 

`NSIndexSet` se usa ampliamente en el propio Foundation framework. Cada vez que un método obtiene elementos de una colección ordenada, como en el caso de un array o del data source de una tableView, se usará `NSIndexSet` en algún punto intermedio.

Es muy probable que encuentres ciertos aspectos de tu modelo que podrían representarse también con `NSIndexSet`. Por ejemplo, AFNetworking usa un `NSIndexSet` para representar los códigos de estado de una respuesta HTTP: el usuario define un set de códigos "aceptables" (en el rango 2XX, por defecto), y la respuesta se comprueba usando `containsIndex:`.

---

La librería estándar de Swift incluye `PermutationGenerator`, un tipo que a menudo se pasa por alto y que encaja muy bien con `NSIndexSet`. `PermutationGenerator` empaqueta una colección y una secuencia de índices (o index set, ¿te suena familiar?) para facilitar la iteración:

```swift
let streetscape = ["Ashmead", "Belmont", "Clifton", "Douglas", "Euclid", "Fairmont",
						"Girard", "Harvard", "Irving", "Kenyon", "Lamont", "Monroe",
						"Newton", "Otis", "Perry", "Quincy"]

let selectedIndices = NSMutableIndexSet(indexesInRange: NSRange(0...2))
selectedIndices.addIndex(5)
selectedIndices.addIndexesInRange(NSRange(11...13))

for street in PermutationGenerator(elements: streetscape, indices: selectedIndices) {
    print(street)
}
// Ashmead, Belmont, Clifton, Fairmont, Monroe, Newton, Otis
```

Aquí tienes algunas ideas para hacerte pensar en términos de index sets:

- ¿Tienes una lista de preferencias de usuario y quieres guardar cúales están activadas y cuáles no? Usa un `NSIndexSet` en combinación con un `enum` `typedef`.
- ¿Quieres filtrar una lista de elementos por una serie de condiciones? Descarta `NSPredicate`; en su lugar cachea los índices de los objetos que satisfacen cada condición y haz la unión o intersección de esos índices a medida que se añaden o eliminan condiciones.

---

En general, `NSIndexSet` es una clase robusta. Quizá un poco más *nerd* que sus colecciones hermanas, pero tiene su lugar y, como mínimo, es un gran indicativo de la fantástica funcionalidad que puedes encontrar al poner un poco de atención en lo que Foundation usa en sus propias APIs. 
