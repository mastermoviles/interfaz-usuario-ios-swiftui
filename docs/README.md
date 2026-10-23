# Presentación

En estos apuntes se extienden los contenidos de iOS para la asignatura <a href="http://cvnet.cpd.ua.es/Guia-Docente/?wcodasi=43282&wlengua=es&scaca=2022-23">Interfaz de Usuario en Dispositivos Móviles</a>, perteneciente al <a href="http://www.eps.ua.es/es/master-moviles/">Máster en Desarrollo de Software para Dispositivos Móviles</a> de la <a href="http://www.ua.es">Universidad de Alicante</a>. Estos materiales están elaborados por <a href="http://www.dlsi.ua.es/~pertusa">Antonio Pertusa</a>.

Las interfaces de este libro se construyen con **SwiftUI**. Se da por supuesto que
conoces el lenguaje _Swift_, pero no que hayas usado _SwiftUI_ antes: la primera
sesión es un recorrido por lo imprescindible para seguir el resto del libro, y a
partir de ahí los ejercicios se implementan desde cero.

Empezaremos con una introducción a _SwiftUI_ y a su forma declarativa de construir
interfaces. A continuación veremos la guía de estilo que propone _Apple_ —
actualizada al rediseño _Liquid Glass_ de iOS 26 — y cómo personalizar listas y
barras. Después veremos contenedores y navegación: el controlador paginado
(`TabView` en modo página), las rejillas de contenido (`LazyVGrid`) y las barras de
búsqueda (`.searchable`). Por último trataremos el diseño de aplicaciones
universales con `NavigationSplitView` y las _size classes_.

Los ejercicios y ejemplos de este libro se han desarrollado con **Swift 5** y
**SwiftUI**, sobre **Xcode 27** y probados en **iOS 26**. Todo el material funciona
también con **Xcode 15** y **iOS 17**, que es el que hay instalado en los laboratorios.

<!---!!! note "Soluciones"
    En la carpeta `ejercicios/2627_swiftui` tienes la versión terminada de cada
    ejercicio. Úsala para comprobar tu solución **después** de intentarlo, no antes:
    los ejercicios están pensados para que escribas tú una parte del código.
--->
