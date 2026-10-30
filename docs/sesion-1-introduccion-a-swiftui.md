# 1- Introducción a SwiftUI

Esta sesión es un recorrido rápido por _SwiftUI_. Se asume que ya conoces _Swift_
(tipos, `struct`, `enum`, opcionales, clausuras), pero que no tienes experiencia con _SwiftUI_. 

## De UIKit a SwiftUI

Apple tiene hoy dos sistemas para construir interfaces en iOS. El tradicional es
**UIKit**: los componentes se colocan en un _storyboard_, unas _constraints_ fijan sus
posiciones, y unos _outlets_ conectan la vista con el código. El programa describe
**cómo** cambiar la pantalla: "coge esta etiqueta y ponle este texto".

**SwiftUI** es el sistema moderno, y es **declarativo**: el programa describe **qué**
debe verse en función de los datos que la app tiene en ese momento, y el sistema se
encarga de redibujar cuando esos datos cambian.

Aunque en este libro vamos a trabajar sólo con SwiftUI, conviene conocer la
correspondencia entre ambos, porque la mayor parte de la documentación, los ejemplos y
las respuestas que encontrarás por internet siguen estando escritos en UIKit:

| UIKit | SwiftUI |
|---|---|
| Storyboard + _constraints_ | Código, con contenedores que colocan las vistas |
| `UIViewController` | `struct` que cumple `View` |
| `@IBOutlet` + `label.text = "Hola"` | `Text(saludo)` |
| `UITableView` + `dataSource` / `delegate` | `List(datos) { ... }` |
| `UICollectionView` + _flow layout_ | `LazyVGrid` |
| `UINavigationController` | `NavigationStack` |
| `viewDidLoad`, `viewWillAppear` | `.onAppear`, `.task` |

Las diferencias prácticas más importantes son tres:

* **Desaparece el _storyboard_.** Un proyecto SwiftUI no tiene ficheros `.storyboard`
  ni `.xib`: la pantalla se construye componiendo vistas pequeñas, y no hay _outlets_
  que reconectar cuando se renombra algo.
* **Desaparecen las _constraints_.** La posición no se fija con reglas entre elementos,
  sino anidando contenedores que distribuyen el espacio.
* **Desaparece el código de actualización.** No se modifica la pantalla: se modifica
  una variable y la vista se redibuja sola. Este es un aspecto muy importante que veremos en detalle en la sección  [Estado](#estado-state-y-binding).

## Creando un proyecto SwiftUI

Abre Xcode y crea un proyecto _iOS > App_. Si te da la opción, en el diálogo de creación, en
**Interface** elige **SwiftUI** (en lugar de _Storyboard_). El proyecto que se genera
tiene solo dos ficheros de código:

```swift
// MiAppApp.swift  — el punto de entrada
import SwiftUI

@main
struct MiAppApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

```swift
// ContentView.swift — la primera pantalla
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack {
            Image(systemName: "globe")
            Text("Hello, world!")
        }
        .padding()
    }
}

#Preview {
    ContentView()
}
```

`@main` marca el punto de entrada de la app, y ocupa el lugar que en UIKit tenían 
`AppDelegate` y `SceneDelegate`. `WindowGroup` es la ventana, y dentro va la vista
que se muestra al arrancar: para cambiar la pantalla inicial basta con poner otra vista
ahí, sin tocar ningún _storyboard_ ni el `Info.plist`.


## La vista: `View` y `body`

Toda vista de _SwiftUI_ es una `struct` que cumple el protocolo `View` y que tiene una
propiedad `body`:

```swift
struct Saludo: View {
    var body: some View {
        Text("Hola")
    }
}
```

Las vistas son **tipos valor** (`struct`), no objetos. Crearlas es muy eficiente, y
_SwiftUI_ las construye y destruye continuamente. No guardes en ellas nada que deba
sobrevivir a un redibujado salvo con los mecanismos que veremos en
[Estado](#estado-state-y-binding).

`some View` significa "alguna vista concreta, pero sin especificar cuál". Es lo que permite
devolver estructuras anidadas complejas sin escribir su tipo completo.

## Modificadores

En UIKit se configuraba un componente asignando propiedades (`label.font = ...`). En
_SwiftUI_ se usan **modificadores**: métodos que se encadenan y que devuelven una vista
nueva ya configurada.

```swift
Text("Hola")
    .font(.title)
    .foregroundStyle(.secondary)
    .padding()
    .background(.yellow)
```

Dos reglas que conviene tener en cuenta desde el principio:

**1. El orden importa.** Cada modificador envuelve lo anterior. No es lo mismo:

```swift
Text("Hola").padding().background(.yellow)   // el amarillo incluye el margen
Text("Hola").background(.yellow).padding()   // el amarillo sólo cubre el texto
```

**2. Se aplican a todo lo que tienen debajo.** Si pones un modificador sobre un
contenedor, lo heredan sus vistas hijas:

```swift
VStack {
    Text("Uno")
    Text("Dos")
}
.font(.title)     // las dos etiquetas salen en grande
```

## Colocando vistas: `VStack`, `HStack`, `ZStack` y `Spacer`

Aquí no hay _constraints_ ni coordenadas. La posición se consigue anidando
contenedores que reparten el espacio disponible:

* `VStack` apila en vertical (_column_).
* `HStack` apila en horizontal (_row_).
* `ZStack` apila en profundidad, una vista encima de otra.
* `Spacer` es un hueco elástico que empuja al resto.

```swift
HStack {
    Text("Izquierda")
    Spacer()              // empuja las dos etiquetas a los extremos
    Text("Derecha")
}
.padding()
```

Los contenedores admiten alineación y separación:

```swift
VStack(alignment: .leading, spacing: 12) {
    Text("Título").font(.headline)
    Text("Subtítulo").foregroundStyle(.secondary)
}
```

!!! tip "Equivalencia mental"
    `VStack` + `HStack` + `Spacer` cubre la mayor parte de lo que en UIKit se resolvía
    con _constraints_. Si te encuentras calculando posiciones o tamaños a mano,
    probablemente haya una combinación de contenedores que lo resuelve mejor y que
    además se adaptará de forma adaptativa a pantallas de cualquier tamaño.

## Imágenes

```swift
Image("logo-master")          // imagen de los Assets
Image(systemName: "star.fill") // icono del sistema (SF Symbols)
```

Una imagen de _Assets_ se muestra por defecto a su tamaño original. Para que se adapte hay
que hacerla redimensionable e indicar cómo encajarla:

```swift
Image("portada")
    .resizable()
    .scaledToFit()        // cabe entera, puede dejar huecos
    .frame(width: 100, height: 140)
```

`scaledToFit` mantiene la proporción y hace que quepa entera; `scaledToFill` llena el
espacio y recorta lo que sobra (normalmente se acompaña de `.clipped()`).

!!! warning "Error habitual"
    Si olvidas `.resizable()`, el `.frame` no tendrá ningún efecto visible sobre la
    imagen: seguirá dibujándose a su tamaño original.

## Estado: `@State` y `Binding`

Esta es la idea central de _SwiftUI_, y la diferencia conceptual más importante con
UIKit: **no se modifica la pantalla, se modifica una variable**, y la pantalla se
vuelve a dibujar sola. Donde en UIKit escribirías `label.text = "Van 3"`, aquí cambias
el número y la etiqueta se actualiza sin que tú la toques.

Una propiedad marcada con `@State` pertenece a la vista y, cuando cambia, provoca el
redibujado:

```swift
struct Contador: View {
    @State private var cuenta = 0

    var body: some View {
        VStack {
            Text("Llevamos \(cuenta)")
            Button("Suma uno") {
                cuenta += 1        // esto redibuja la vista
            }
        }
    }
}
```

`@State` se declara siempre `private`, porque es estado interno de esa vista.

Cuando un componente necesita **leer y escribir** una variable que vive en otra vista
(por ejemplo un `TextField` que escribe en `texto`), se pasa un **binding**, que se
escribe con `$`:

```swift
struct Formulario: View {
    @State private var nombre = ""

    var body: some View {
        VStack {
            TextField("Escribe tu nombre", text: $nombre)   // $ es el binding
            Text("Hola, \(nombre)")                         // sin $ es sólo de lectura
        }
        .padding()
    }
}
```

La regla es sencilla: **`nombre` es el valor, `$nombre` es la conexión de dos
sentidos**. Si el componente sólo muestra el dato, no lleva `$`; si además lo
modifica, sí.

### Propiedades calculadas

Muchas veces no hace falta estado: basta con calcular el valor a partir del que ya
tienes. Es preferible, porque no puede quedar desincronizado:

```swift
struct Busqueda: View {
    let palabras = ["uno", "dos", "tres"]
    @State private var filtro = ""

    // No es @State: se recalcula solo cuando cambia `filtro`
    var resultados: [String] {
        filtro.isEmpty ? palabras : palabras.filter { $0.hasPrefix(filtro) }
    }

    var body: some View { /* ... */ Text("\(resultados.count) resultados") }
}
```

## Listas

`List` es el equivalente de `UITableView`, pero sin `dataSource` ni `delegate`:
recibe directamente los datos y una clausura que describe cómo se dibuja cada fila:

```swift
List(["En", "un", "lugar"], id: \.self) { palabra in
    Text(palabra)
}
```

El parámetro `id` le dice a _SwiftUI_ cómo distinguir una fila de otra. Si los
elementos son de un tipo propio, lo normal es que cumplan `Identifiable`, y entonces
no hace falta indicar `id`:

```swift
struct Libro: Identifiable {
    let id = UUID()
    let titulo: String
}

struct Lista: View {
    let libros = [Libro(titulo: "Uno"), Libro(titulo: "Dos")]

    var body: some View {
        List(libros) { libro in
            Text(libro.titulo)
        }
    }
}
```

Dentro de una fila puedes poner cualquier vista, no sólo texto; de hecho así es como
se hacen las celdas personalizadas, como veremos en la sesión 2.

## Navegación

Para apilar pantallas se usa `NavigationStack` junto con `NavigationLink`:

```swift
NavigationStack {
    List(libros) { libro in
        NavigationLink(libro.titulo) {
            DetalleLibro(libro: libro)     // pantalla destino
        }
    }
    .navigationTitle("Biblioteca")
}
```

`.navigationTitle` pone el título de la barra. Para añadir botones a la barra se usa
`.toolbar`:

```swift
.toolbar {
    ToolbarItem(placement: .topBarTrailing) {
        Button("Añadir") { /* ... */ }
    }
}
```

## Extraer subvistas

Una `body` que crece mucho se vuelve ilegible, y además Xcode empieza a dar errores de
compilación poco claros. La solución es sacar trozos a vistas propias:

```swift
struct FilaLibro: View {
    let libro: Libro

    var body: some View {
        HStack {
            Text(libro.titulo)
            Spacer()
            Image(systemName: "book")
        }
    }
}

// y en la lista:
List(libros) { libro in
    FilaLibro(libro: libro)
}
```

Esto no tiene coste: las vistas son `struct` muy ligeras. Extraer subvistas es la
práctica normal en _SwiftUI_, no una optimización.

## Previews

`#Preview` permite ver la vista sin lanzar el simulador, y se actualiza mientras
escribes:

```swift
#Preview {
    ContentView()
}
```

Puedes tener varias _previews_ y configurarlas, por ejemplo para ver la vista en modo
oscuro o con una fila concreta:

```swift
#Preview("Modo oscuro") {
    ContentView().preferredColorScheme(.dark)
}
```

!!! tip
    Si la _preview_ deja de funcionar sin motivo aparente, prueba a pulsar el botón de
    refrescar (o _Option-Cmd-P_). Es habitual que se quede parada tras un error de
    compilación.

---

## Ejercicio 1 - Primera pantalla en SwiftUI

Vamos a construir una ficha de un libro para practicar lo visto. Este es el resultado
al que hay que llegar: la portada, el título, el autor y un botón que marca el libro
como favorito.

![Resultado de la ficha de libro](assets/img/ficha_libro_resultado.png "Ficha de libro")

### Creando el proyecto

Crea un proyecto _iOS > App_ con **Interface: SwiftUI** y nómbralo `ficha_libro`.
Descarga <a href="assets/descargas/imagenes_celdas.zip">imagenes_celdas.zip</a>,
descomprímelo y arrastra `historia.jpg` a los _Assets_ del proyecto.

### El modelo

Crea un fichero nuevo (_File > New > File > Swift File_) llamado `Libro.swift`:

```swift
import Foundation

struct Libro: Identifiable {
    let id = UUID()
    let titulo: String
    let autor: String
    let portada: String
}
```

### La vista

Sustituye el contenido de `ContentView.swift` por este esqueleto, que ya tiene la
estructura pero le faltan piezas:

```swift
import SwiftUI

struct ContentView: View {
    let libro = Libro(titulo: "La historia interminable",
                      autor: "Michael Ende",
                      portada: "historia")

    // a) Declara aquí el estado que recuerda si el libro es favorito

    var body: some View {
        VStack(spacing: 16) {
            // b) La portada, redimensionable, de 180 puntos de ancho

            Text(libro.titulo)
                .font(.title)
                .multilineTextAlignment(.center)

            // c) El autor, en un cuerpo más pequeño y en color secundario

            // d) El botón de favorito

            Spacer()
        }
        .padding()
    }
}

#Preview {
    ContentView()
}
```

### Se pide

_a)_ Declara una propiedad `@State private var esFavorito = false`. Recuerda que el
estado que pertenece a la vista se declara siempre `private`.

_b)_ Añade la portada con `Image(libro.portada)`. Tendrás que hacerla `.resizable()`,
mantener su proporción con `.scaledToFit()` y darle un `.frame(width: 180)`. Prueba a
quitar `.resizable()` para ver qué ocurre: es el error más habitual al empezar.

_c)_ Añade el autor con `Text(libro.autor)`, usando `.font(.subheadline)` y
`.foregroundStyle(.secondary)`.

_d)_ Añade un botón que cambie el valor de `esFavorito` al pulsarlo, y cuyo texto
dependa de ese valor. Es decir, debe poner "Quitar de favoritos" cuando ya lo sea y
"Marcar como favorito" cuando no. Una pista sobre la estructura:

```swift
Button(esFavorito ? "..." : "...") {
    esFavorito.toggle()
}
.buttonStyle(.borderedProminent)
```

_e)_ Comprueba en la _preview_ que al pulsar el botón cambia el texto. Fíjate en que
no has escrito en ningún momento «cambia el texto del botón»: sólo has cambiado una
variable, y la vista se ha redibujado sola. Esa es la idea central de _SwiftUI_.

_f)_ **Para practicar la extracción de subvistas**, saca el bloque del título y el
autor a una vista aparte llamada `DatosLibro`, que reciba el libro como propiedad, y
úsala desde `ContentView`. El resultado en pantalla debe ser idéntico.

<!--
!!! question "Para pensar"
    ¿Por qué `esFavorito` tiene que ser `@State` y no una variable normal? Prueba a
    declararla como `private var esFavorito = false` (sin `@State`) y observa el error
    que da Xcode al intentar modificarla dentro del botón.
-->
