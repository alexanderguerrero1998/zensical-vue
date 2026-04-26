# How Vue Works: The Basics

En el capítulo anterior, aprendiste las **herramientas esenciales** para construir una aplicación **Vue** y creaste tu **primera aplicación Vue**, preparándote para el siguiente paso: aprender **cómo funciona Vue** escribiendo código Vue.

Este capítulo te introduce a los conceptos de **Virtual Document Object Model (Virtual DOM)** y los fundamentos de escribir un **componente Vue** con **Vue Options API**. También explora **directivas Vue adicionales** y el **mecanismo de reactividad de Vue**. Al final del capítulo, entenderás **cómo funciona Vue** y podrás **escribir y registrar un componente Vue** para usarlo en tu aplicación.

## DOM virtual bajo el capó

Vue no trabaja directamente con el Modelo de Objetos del Documento (DOM).En su lugar, implementa su propio DOM Virtual para optimizar el rendimiento de la aplicación en tiempo de ejecución.

Para comprender a fondo cómo funciona el DOM Virtual, comenzamos con el concepto de DOM.

El DOM representa el contenido del documento HTML (o XML) en la web, en forma de una estructura de datos en memoria con forma de árbol (como se muestra en la Figura 2-1).
Actúa como una interfaz de programación que conecta la página web con el código de programación (como JavaScript). Las etiquetas, como `<div>` o `<section>`, en el documento HTML se representan como nodos y objetos programáticos.

![](domtree.png 'Example of a DOM tree' )

Después de que el navegador analiza el documento HTML, el DOM estará disponible para interacción inmediatamente. Ante cualquier cambio de diseño, el navegador pinta y repinta el DOM constantemente en segundo plano. A este proceso de analizar y pintar el DOM lo llamamos rasterización o el pipeline de píxel a pantalla. La figura 2-2 muestra cómo funciona la rasterización.

![](raterizacionprocess.png 'Browser rasterization process')

### El problema de la actualización del diseño

Cada renderizado supone un coste para el rendimiento del navegador. Dado que el DOM puede constar de muchos nodos, consultar y actualizar uno o varios nodos puede resultar extremadamente costoso. He aquí un ejemplo sencillo de una lista de elementos `<li>` en el DOM: