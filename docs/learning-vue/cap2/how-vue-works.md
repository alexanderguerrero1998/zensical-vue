# How Vue Works: The Basics

En el capítulo anterior, aprendiste las **herramientas esenciales** para construir una aplicación **Vue** y creaste tu **primera aplicación Vue**, preparándote para el siguiente paso: aprender **cómo funciona Vue** escribiendo código Vue.

Este capítulo te introduce a los conceptos de **Virtual Document Object Model (Virtual DOM)** y los fundamentos de escribir un **componente Vue** con **Vue Options API**. También explora **directivas Vue adicionales** y el **mecanismo de reactividad de Vue**. Al final del capítulo, entenderás **cómo funciona Vue** y podrás **escribir y registrar un componente Vue** para usarlo en tu aplicación.

## DOM virtual bajo el capó

Vue no trabaja directamente con el Modelo de Objetos del Documento (DOM).En su lugar, implementa su propio DOM Virtual para optimizar el rendimiento de la aplicación en tiempo de ejecución.

Para comprender a fondo cómo funciona el DOM Virtual, comenzamos con el concepto de DOM.

El DOM representa el contenido del documento HTML (o XML) en la web, en forma de una estructura de datos en memoria con forma de árbol. Actúa como una interfaz de programación que conecta la página web con el código de programación (como JavaScript). Las etiquetas, como `<div>` o `<section>`, en el documento HTML se representan como nodos y objetos programáticos.

![](domtree.png 'Example of a DOM tree' )

Después de que el navegador analiza el documento HTML, el DOM estará disponible para interacción inmediatamente. Ante cualquier cambio de diseño, el navegador pinta y repinta el DOM constantemente en segundo plano. A este proceso de analizar y pintar el DOM lo llamamos rasterización o el pipeline de píxel a pantalla. La figura 2-2 muestra cómo funciona la rasterización.

![](raterizacionprocess.png 'Browser rasterization process')

### El problema de la actualización del diseño

Cada renderizado supone un coste para el rendimiento del navegador. Dado que el DOM puede constar de muchos nodos, consultar y actualizar uno o varios nodos puede resultar extremadamente costoso. He aquí un ejemplo sencillo de una lista de elementos `<li>` en el DOM:

```html linenums="1"
ul class="list" id="todo-list">
    <li class="list-item">To do item 1</li>
    <li class="list-item">To do item 2</li>
    <!--so on…-->
</ul>
```

**Agregar/quitar** un elemento `<li>` o **modificar su contenido** requiere **consultar el DOM** para ese elemento usando **`document.getElementById`** (o **`document.getElementsByClassName`**). Entonces necesitas **llevar acabo las actualizaciones deseadas** usando las **APIs apropiadas del DOM**.

Por ejemplo, si quieres **agregar un nuevo ítem** al ejemplo anterior, necesitas hacer los siguientes pasos:

1. **Consultar el elemento de la lista** por el valor de su atributo `id` —"todolist".

2. **Agregar el nuevo elemento `li`** usando **`document.createElement()`**.

3. **Establecer el `textContent`** y los **atributos relevantes** para que coincidan con el estándar de los otros elementos usando **`setAttribute()`**.

4. **Agregar ese elemento a la lista que lo contiene** encontrada en el paso 1 como su hijo usando **`appendChild()`**.

```js linenums="1"
const list = document.getElementById('todo-list');
const newItem = document.createElement('li');
newItem.setAttribute('class', 'list-item');
newItem.textContent = 'To do item 3';
list.appendChild(newItem);
```

Similarmente, supongamos que quieres **cambiar el contenido de texto** del **2º elemento `li`** a **"buy groceries"**. En ese caso, realizas el **paso 1** para obtener el **elemento de la lista que lo contiene**, luego buscar el **elemento de destino** usando **`getElementsByClassName()`**, y finalmente cambias su **`textContent`** al **nuevo contenido**.

```js linenums="1"
const secondItem = list.getElementsByClassName('list-item')[1];
secondItem.textContent = 'Buy groceries
```

**Consultar y actualizar el DOM** a **pequeña escala** generalmente **no impacta enormemente el rendimiento**.  Sin embargo, estas acciones **pueden ralentizar la página** si se realizan de manera **repetitiva** (en pocos segundos) y en una **página web más compleja**. El **impacto en el rendimiento es significativo** cuando hay **actualizaciones menores consecutivas**. Muchos frameworks, como **Angular 1.x**, **no reconocen ni abordan** este problema de rendimiento a medida que **crece la base de código**. El **Virtual DOM** está diseñado para **solucionar el problema de actualización del layout**.

## What Is Virtual DOM?

El __DOM virtual__ es una copia virtual en memoria del __DOM real__ del
navegador, pero es más ligero y cuenta con funcionalidades adicionales

**Imita la estructura del DOM real**, con una **estructura de datos diferente** (usualmente un **Objeto**) (ver Figura 2-3). [styde](https://styde.net/que-es-el-virtual-dom-en-react/)

![](virtualdom.png '')

De tras de escenas, el **Virtual DOM** aún usa la **API del DOM** para **construir y renderizar** elementos actualizados en el navegador. Por lo tanto, **sigue provocando el proceso de repintado** del navegador, pero **de forma más eficiente**.

En resumen, el **Virtual DOM** es un **patrón abstracto** que busca **liberar al DOM** de todas las acciones que pueden llevar a **ineficiencias de rendimiento**, como **manipular atributos**, **manejar eventos** y **actualizar manualmente elementos del DOM**.

## How Virtual DOM Works in Vue

El **Virtual DOM** se sitúa **entre el DOM real** y el **código de la aplicación Vue**. A continuación se muestra un **ejemplo** de cómo se ve un **nodo en el Virtual DOM**:

```js linenums="1"
const node = {
tag: 'div',
attributes: [{ id: 'list-container', class: 'list-container' }],
children: [ /* an array of nodes */]
}
```

Llamemos a este nodo **VNode**. El **VNode** es un **nodo virtual** que reside dentro del **Virtual DOM** y representa el **elemento del DOM real** en el DOM real. 

A través de **interacciones con la UI**, el usuario le dice a **Vue** en qué **estado desea** que esté el elemento; **Vue** entonces activa el **Virtual DOM** para actualizar el **objeto (nodo)** a la forma deseada, mientras **registra esos cambios**. 

Finalmente, se comunica con el **DOM real** y realiza **actualizaciones precisas** en los nodos modificados. 

Dado que el **Virtual DOM** es un **árbol de objetos JavaScript personalizados**, actualizar un componente equivale a actualizar un **objeto JavaScript personalizado**. Este proceso **no toma mucho tiempo**. 

Como **no llamamos a ninguna API del DOM**, esta acción de actualización **no provoca un repintado del DOM**. Una vez que el **Virtual DOM** termina de actualizarse, se **sincroniza en lote** con el **DOM real**, llevando los cambios a reflejarse en el navegador. 

La **Figura 2-4** ilustra cómo funcionan las actualizaciones del **Virtual DOM** al **DOM real** cuando se agrega un nuevo ítem de lista y se cambia el texto del ítem de lista.


![](actualdom.png 'Actualización del DOM virtual al DOM real: se añade un nuevo elemento y se actualiza el texto de un elemento existente en la lista.')

Dado que el **Virtual DOM** es un **árbol de objetos**, podemos **rastrear fácilmente** las **actualizaciones específicas** que necesitan sincronizarse con el **DOM real** al modificar el Virtual DOM. 

En lugar de **consultar y actualizar directamente** en el **DOM real**, ahora podemos **programar y llamar** las APIs actualizadas con una **única función de renderizado** en **un ciclo de actualización** para mantener la **eficiencia del rendimiento**. 

Ahora que entendemos cómo funciona el **Virtual DOM**, exploraremos la **instancia de Vue** y la **Vue Options API**.

## The Vue App Instance and Options API

Toda **aplicación Vue** comienza con una **única instancia de componente Vue** como **raíz de la aplicación**. Cualquier **otro componente Vue** creado en la misma aplicación necesita **anidarse dentro** de este **componente raíz**.

!!! note

    Puedes encontrar el **ejemplo de código de inicialización** en **`main.ts`** de nuestro proyecto Vue. **Vite genera automáticamente** el código como parte de su **proceso de scaffolding**. También encontrarás el **ejemplo de código de este capítulo** dentro de este archivo.

En Vue 2, Vue expone una clase Vue (o función JavaScript) para que puedas crear una instancia de componente Vue a partir de un conjunto de opciones de configuración, utilizando la siguiente sintaxis:

```js linenums="1"
const App = {
//component's options
}
const app = new Vue(App)
```

**Vue** recibe un **componente**, o más precisamente, la **configuración del componente**. La **configuración de un componente** es un **Objeto** que contiene todas las **opciones de configuración iniciales** del componente. Llamamos a la estructura de este argumento **Options API**, que es **otro de los APIs centrales de Vue**. 

A partir de **Vue 3**, ya **no puedes llamar directamente** a `new Vue()`. En su lugar, creas la **instancia de la aplicación** usando el método **`createApp()`** del paquete `vue`. Este cambio en la funcionalidad **mejora el aislamiento** de cada instancia Vue creada tanto en **dependencias** como en **componentes compartidos** (si los hay) y la **legibilidad del código**.

```js linenums="1"
import { createApp } from 'vue'
const App = {
//component's options
}
const app = createApp(App)
```

`createApp()` también acepta un objeto con las configuraciones del componente. En función de estas configuraciones, Vue crea una instancia de componente Vue como aplicación raíz `app`. A continuación, debes montar el componente raíz `app` en el elemento HTML deseado utilizando el método `app.mount()`, de la siguiente forma:

```js linenums="1"
app.mount('#app')
```

`#app` es el selector de id único para el elemento raíz de la aplicación. El motor de Vue busca ese elemento usando este id, monta la instancia de la aplicación en él y luego renderiza la aplicación en el navegador. El siguiente paso es proporcionar las configuraciones para que Vue cree una instancia de componente según la API de opciones (Options API).

!!! note

    A partir de este momento, escribimos código de acuerdo con los estándares de la API de Vue 3.

## Exploring the Options API

La API de opciones (Options API) es la API central de Vue para inicializar un componente Vue. Contiene las configuraciones del componente estructuradas en formato de objeto. Nosotros dividimos sus propiedades esenciales en cuatro categorías principales:

`State handling` 

:   Incluye `data()`, que devuelve el estado de datos local del componente, `computed`, `methods` y `watch` para habilitar la observación en datos locales específicos, y `props` para los datos entrantes. [vueschool](https://vueschool.io/articles/vuejs-tutorials/options-api-vs-composition-api/)

`Rendering`  

:   Incluye `template` para la plantilla HTML de la vista, y `render()` como la lógica de renderizado del componente. [dev](https://dev.to/sucodelarangela/vue3-options-api-vs-composition-api-en-1fbo)

`Lifecycle hooks` 

:   Tales como, `beforeCreate()`, `created()`, `mounted()`, etc., que sirven para gestionar diferentes etapas del ciclo de vida de un componente. [dev](https://dev.to/sucodelarangela/vue3-components-lifecycle-en-10e)

`Others`

:   Incluye `provide()` e `inject()` para manejar las diferentes personalizaciones y la comunicación entre componentes., y `components`, una colección de plantillas de componentes anidados que se pueden usar dentro del componente. [vuejs](https://vuejs.org/guide/components/provide-inject)

El siguiente listado es un ejemplo de la estructura de nuestro componente raíz `App` basada en la Options API.

```js linenums="1"
import { createApp } from 'vue'
const App = {
template: "This is the app's entrance",
}
const app = createApp(App)
app.mount('#app')
```

En el código anterior, una plantilla HTML muestra texto normal. También podemos definir un estado de datos local usando la función `data()`, que veremos con más detalle en «Creación de estado local con propiedades de datos».

También puedes reescribir el código anterior para usar la función `render()`:

```js linenums="1"
import { createApp } from 'vue'

const App = {
    render() {
    return "This is the app's entrance"
  }
}
const app = createApp(App)
app.mount('#app')
```

Ambos códigos generarán el mismo resultado 

![](exampleouput.png)

Si abres la pestaña "Elements" (Elementos) en las herramientas de desarrollador del navegador, verás que el DOM real ahora contiene un `div` con `id="app"` y un contenido de texto "This is the app’s entrance" 

![](treedom.png)

También puedes crear un componente nuevo, `Description`, que renderice un texto estático y lo pase a los componentes del `App`. Luego puedes usarlo como un componente anidado en la plantilla.

!!! example "Insertar de componente en el App"

    === "Sin SFC (Options API + template en string)"
    
        ```js linenums="1"
        import { createApp } from 'vue'

        // Crea el componente
        const Description = {
        template: "This is the app's entrance" // (1)!
        };

        const App = {
        // Registra el componente
        components: { Description },

        //Lo encuentra en el template
        template: '<Description />'
        }
        const app = createApp(App)
        app.mount('#app')
        ```

        1. Esto es un `template en string`

    === "Con SFC (Composition API + script setup)"

        ```js linenums="1"

        // Logica del componente, se define: varibles, funciones e imports
        <script setup> 
            import Description from './components/Description.vue'; 
        </script>
        
        //Estructura HTML
        <template>      
            <Description/>
        </template>

        // Estilos del componente (CSS) 
        <style scoped></style> 
        ```

       
La salida sigue siendo la misma que en la figura 2‑6. 

Observa que aquí debes declarar bien `template` o bien la función `render()` (véase “La función render y JSX”) para el componente. Sin embargo, no necesitas estas propiedades si estás escribiendo el componente con el estándar Single File Component (SFC). Trataremos este estándar de componentes en el capítulo 3. A continuación, veamos la sintaxis de la propiedad `template`.

## The Template Syntax

En la Options API, `template` acepta una sola cadena de texto que contiene código HTML válido y representa el componente de la interfaz de usuario. El motor de Vue parsea este valor y lo compila en código JavaScript optimizado, luego genera los elementos del DOM correspondientes. [vuejs](https://vuejs.org/guide/essentials/template-syntax)

El siguiente código muestra nuestro componente raíz `App`, cuyo diseño es un solo `div` que muestra el texto “This is the app’s entrance”:

```js linenums="1"
import { createApp } from 'vue'
const App = {
template: "<div>This is the app's entrance</div>",
}
const app = createApp(App)
app.mount('#app')
```

Para código de plantilla HTML de varios niveles, podemos usar caracteres de acento grave (plantillas literales de JavaScript), denotados por el símbolo `` ` ``, y así mantener la legibilidad. Podemos reescribir la plantilla de `App` en el ejemplo anterior para incluir otros elementos `h1` y `h2`, como en el siguiente caso:

```js linenums="1"
import { createApp } from 'vue'
const App = {
template: `
<h1>This is the app's entrance</h1>
<h2>We are exploring template syntax</h2>
`,
}
```
El motor Vue renderizará en el DOM con dos encabezados

![](headers.png)

La sintaxis de la propiedad `template` es esencial para crear el vinculo entre un elemento DOM específico y los datos locales del componente, utilizando directivas y una sintaxis dedicada. A continuación exploraremos cómo definir los datos que queremos mostrar en la interfaz de usuario.

## Creating Local State with Data Properties

La mayoría de los componentes mantienen su estado local (o datos locales) o reciben datos desde una fuente externa. En Vue, nosotros almacenamos el estado local del componente usando la propiedad `data()` de la Options API. 

`data()` es una función anónima que devuelve un objeto que representa el estado de los datos locales del componente. A ese objeto devuelto lo llamamos **objeto de datos**. Cuando inicializa la instancia del componente, el motor de Vue añade cada propiedad de este objeto de datos a su sistema de reactividad para rastrear sus cambios y disparar la re‑renderización de la plantilla de la interfaz cuando sea necesario. En resumen, el objeto de datos es el estado reactivo de un componente. 

Para inyectar una propiedad de `data` en la plantilla usamos la sintaxis de “mustache”, denotada por las llaves dobles `{{}}`. Dentro de la plantilla HTML, envolvemos la propiedad de datos con las llaves allí donde necesitamos inyectar su valor. 

```js linenums="1"
import { createApp } from 'vue'
type Data = {
    title: string;
}
const App = {
        template: `<div>{{ title }}</div>`, data(): Data {
        return {
            title: 'My first Vue component'
        }
    }
}
const app = createApp(App)
app.mount('#app')

```