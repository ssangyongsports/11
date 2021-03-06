# Desarrollo del Frontend

Esta p谩gina describe c贸mo realizar cambios en la interfaz de usuario de Flarum. C贸mo a帽adir botones, marquesinas y texto parpadeante. 馃ぉ

[Recuerda](/extend/start.md#architecture), el frontend de Flarum es una **aplicaci贸n JavaScript de una sola p谩gina**. No hay Twig, Blade, o cualquier otro tipo de plantilla PHP para hablar. Las pocas plantillas que est谩n presentes en el backend s贸lo se utilizan para renderizar el contenido optimizado para el motor de b煤squeda. Todos los cambios en la interfaz de usuario deben hacerse a trav茅s de JavaScript.

Flarum tiene dos aplicaciones frontales separadas:

* `forum`, el lado p煤blico de su foro donde los usuarios crean discusiones y mensajes.
* `admin`, el lado privado de tu foro donde, como administrador de tu foro, configuras tu instalaci贸n de Flarum.

Comparten el mismo c贸digo fundacional, as铆 que una vez que sabes c贸mo extender uno, sabes c贸mo extender ambos.

:::tip Typings!

Along with new TypeScript support, we have a [`tsconfig` package](https://www.npmjs.com/package/flarum-tsconfig) available, which you should install as a dev dependency to gain access to our typings. Make sure you follow the instructions in the [package's README](https://github.com/flarum/flarum-tsconfig#readme) to configure typings support.

:::

## Transpilaci贸n y estructura de archivos

Esta parte de la gu铆a explicar谩 la configuraci贸n de archivos necesaria para las extensiones. Una vez m谩s, recomendamos encarecidamente utilizar el [generador de extensiones FoF](https://github.com/FriendsOfFlarum/extension-generator) no oficial para configurar la estructura de los archivos por usted. Dicho esto, usted debe leer esto para entender lo que est谩 pasando bajo la superficie.

Antes de que podamos escribir cualquier JavaScript, necesitamos configurar un **transpilador**. Esto nos permite usar [TypeScript](https://www.typescriptlang.org/) y su magia en el n煤cleo y las extensiones de Flarum.

Para hacer esta transpilaci贸n, tienes que trabajar en un entorno capaz. No, no se trata de un entorno dom茅stico o de oficina, 隆puedes trabajar en el ba帽o por lo que a m铆 respecta! Me refiero a las herramientas instaladas en tu sistema. Necesitar谩s:

* Node.js y npm ([Descarga](https://nodejs.org/en/download/))
* Webpack (`npm install -g webpack`)

Esto puede ser complicado porque cada sistema es diferente. Desde el sistema operativo que usas, hasta las versiones de los programas que tienes instalados, pasando por los permisos de acceso de los usuarios... 隆me dan escalofr铆os s贸lo de pensarlo! Si tienes problemas, ~~dale recuerdos~~ utiliza [Google](https://google.com) para ver si alguien se ha encontrado con el mismo error que t煤 y ha encontrado una soluci贸n. Si no, pide ayuda en la [Comunidad Flarum](https://discuss.flarum.org) o en el [chat de Discord](https://flarum.org/discord/).

Es hora de configurar nuestro peque帽o proyecto de transpilaci贸n de JavaScript. Crea una nueva carpeta en tu extensi贸n llamada `js`, y luego introduce un par de archivos nuevos. Una extensi贸n t铆pica tendr谩 la siguiente estructura de frontend:

```
js
鈹溾攢鈹? dist (compiled js is placed here)
鈹溾攢鈹? src
鈹?   鈹溾攢鈹? admin
鈹?   鈹斺攢鈹? forum
鈹溾攢鈹? admin.js
鈹溾攢鈹? forum.js
鈹溾攢鈹? package.json
鈹斺攢鈹? webpack.config.json
```

### package.json

```json
{
  "private": true,
  "name": "@acme/flarum-hello-world",
  "dependencies": {
    "flarum-webpack-config": "0.1.0-beta.10",
    "webpack": "^4.0.0",
    "webpack-cli": "^3.0.7"
  },
  "scripts": {
    "dev": "webpack --mode development --watch",
    "build": "webpack --mode production"
  }
}
```

Este es un [archivo de descripci贸n de paquetes](https://docs.npmjs.com/files/package.json) JS est谩ndar, utilizado por npm y Yarn (gestores de paquetes Javascript). Puedes usarlo para a帽adir comandos, dependencias js y metadatos del paquete. En realidad no estamos publicando un paquete npm: esto simplemente se utiliza para recoger las dependencias.

Por favor, ten en cuenta que no necesitamos incluir `flarum/core` o cualquier extensi贸n de flarum como dependencias: se empaquetar谩n autom谩ticamente cuando Flarum compile los frontales de todas las extensiones.

### webpack.config.js

```js
const config = require('flarum-webpack-config');

module.exports = config();
```

[Webpack](https://webpack.js.org/concepts/) es el sistema que realmente compila y agrupa todo el javascript (y sus dependencias) para nuestra extensi贸n. Para que funcione correctamente, nuestras extensiones deben utilizar el [official flarum webpack config](https://github.com/flarum/flarum-webpack-config) (mostrado en el ejemplo anterior).

### tsconfig.json

```json
{
  // Use Flarum's tsconfig as a starting point
  "extends": "flarum-tsconfig",
  // This will match all .ts, .tsx, .d.ts, .js, .jsx files in your `src` folder
  // and also tells your Typescript server to read core's global typings for
  // access to `dayjs` and `$` in the global namespace.
  "include": ["src/**/*", "../vendor/flarum/core/js/dist-typings/@types/**/*"],
  "compilerOptions": {
    // This will output typings to `dist-typings`
    "declarationDir": "./dist-typings",
    "baseUrl": ".",
    "paths": {
      "flarum/*": ["../vendor/flarum/core/js/dist-typings/*"]
    }
  }
}
```

This is a standard configuration file to enable support for Typescript with the options that Flarum needs.

Always ensure you're using the latest version of this file: https://github.com/flarum/flarum-tsconfig#readme.

A continuaci贸n repasaremos las herramientas disponibles para las extensiones.

To get the typings working, you'll need to run `composer update` in your extension's folder to download the latest copy of Flarum's core into a new `vendor` folder. Remember not to commit this folder if you're using a version control system such as Git.

You may also need to restart your IDE's TypeScript server. In Visual Studio Code, you can press F1, then type "Restart TypeScript Server" and hit ENTER. This might take a minute to complete.

### admin.js and forum.js

Estos archivos contienen la ra铆z de nuestro frontend JS real. Podr铆as poner toda tu extensi贸n aqu铆, pero eso no estar铆a bien organizado. Por esta raz贸n, recomendamos poner el c贸digo en `src`, y que estos archivos s贸lo exporten el contenido de `src`. Por ejemplo:

```js
// admin.js
export * from './src/admin';

// forum.js
export * from './src/forum';
```

### src

Si seguimos las recomendaciones para `admin.js` y `forum.js`, querremos tener 2 subcarpetas aqu铆: una para el c贸digo del frontend `admin`, y otra para el c贸digo del frontend `forum`. Si tienes componentes, modelos, utilidades u otro c贸digo que se comparte en ambos frontends, puedes crear una subcarpeta `common` y colocarla all铆.

La estructura para `admin` y `forum` es id茅ntica, as铆 que s贸lo la mostraremos para `forum` aqu铆:

```
src/forum/
鈹溾攢鈹? components/
|-- models/
鈹溾攢鈹? utils/
鈹斺攢鈹? index.js
```

`components`, `models`, y `utils` son directorios que contienen archivos donde puedes definir [componentes](#components), [modelos](data.md#frontend-models), y funciones de ayuda reutilizables. Tenga en cuenta que todo esto es simplemente una recomendaci贸n: no hay nada que le obligue a utilizar esta estructura de archivos en particular (o cualquier otra estructura de archivos).

El archivo m谩s importante aqu铆 es `index.js`: todo lo dem谩s es simplemente extraer clases y funciones en sus propios archivos. Repasemos una estructura t铆pica de archivos `index.js`:

```js
import {extend, override} from 'flarum/extend';

// Proporcionamos nuestro c贸digo de extensi贸n en forma de un "inicializador".
// Este es un callback que se ejecutar谩 despu茅s de que el n煤cleo haya arrancado.
app.initializers.add('our-extension', function(app) {
  // Su c贸digo de extensi贸n aqu铆
  console.log("EXTENSION NAME is working!");
});
```

We'll go over tools available for extensions below.

### Transpilaci贸n

:::tip Bibliotecas externas

Casi todas las extensiones de Flarum necesitar谩n importar *algo* de Flarum Core. Como la mayor铆a de las extensiones, el c贸digo fuente JS del n煤cleo est谩 dividido en las carpetas `admin`, `common` y `forum`. Sin embargo, todo se exporta bajo `flarum`. Para elaborar:

En algunos casos, una extensi贸n puede querer extender el c贸digo de otra extensi贸n de flarum. Esto s贸lo es posible para las extensiones que exportan expl铆citamente su contenido.

* `flarum/tags` y `flarum/flags` son actualmente las 煤nicas extensiones empaquetadas que permiten extender su JS. Puedes importar sus contenidos desde `flarum/{EXT_NAME}/PATH` (por ejemplo, `flarum/tags/components/TagHero`).
* The process for extending each community extension is different; you should consult documentation for each individual extension.

### Transpilation

Bien, es hora de encender el transpilador. Ejecuta los siguientes comandos en el directorio `js`:

```bash
npm install
npm run dev
```

Esto compilar谩 su c贸digo JavaScript listo para el navegador en el archivo `js/dist/forum.js`, y se mantendr谩 atento a los cambios en los archivos fuente. 隆Genial!

Cuando hayas terminado de desarrollar tu extensi贸n (o antes de un nuevo lanzamiento), querr谩s ejecutar `npm run build` en lugar de `npm run dev`: esto construye la extensi贸n en modo de producci贸n, lo que hace que el c贸digo fuente sea m谩s peque帽o y r谩pido.

## Registro de activos

### JavaScript

Para que el JavaScript de tu extensi贸n se cargue en el frontend, necesitamos decirle a Flarum d贸nde encontrarlo. Podemos hacer esto usando el m茅todo `js` del extensor `Frontend`. A帽谩delo al archivo `extend.php` de tu extensi贸n:

```php
<?php

use Flarum\Extend;

return [
    (new Extend\Frontend('forum'))
        ->js(__DIR__.'/js/dist/forum.js')
];
```

Flarum har谩 que cualquier cosa que haga `export` desde `forum.js` est茅 disponible en el objeto global `flarum.extensions['acme-hello-world']`. Por lo tanto, puede elegir exponer su propia API p煤blica para que otras extensiones interact煤en con ella.

:::tip External Libraries

S贸lo se permite un archivo JavaScript principal por extensi贸n. Si necesitas incluir alguna librer铆a JavaScript externa, inst谩lala con NPM e `import` para que se compile en tu archivo JavaScript, o consulta [Rutas y Contenido](/extend/routes.md) para saber c贸mo a帽adir etiquetas `<script>` adicionales al documento del frontend.

:::

### CSS

Tambi茅n puedes a帽adir activos CSS y [LESS](http://lesscss.org/features/) al frontend utilizando el m茅todo `css` del extensor `Frontend`:

```php
    (new Extend\Frontend('forum'))
        ->js(__DIR__.'/js/dist/forum.js')
        ->css(__DIR__.'/less/forum.less')
```

:::tip

Debes desarrollar las extensiones con el modo de depuraci贸n **activado** en `config.php`. Esto asegurar谩 que Flarum recompile los activos de forma autom谩tica, por lo que no tendr谩s que limpiar manualmente la cach茅 cada vez que hagas un cambio en el JavaScript de tu extensi贸n.

:::

## Cambiando la UI Parte 1

La interfaz de Flarum est谩 construida con un framework de JavaScript llamado [Mithril.js](https://mithril.js.org/). Si est谩s familiarizado con [React](https://reactjs.org), lo entender谩s enseguida. Pero si no est谩s familiarizado con ning煤n framework de JavaScript, te sugerimos que pases por un [tutorial](https://mithril.js.org/simple-application.html) para entender los fundamentos antes de continuar.

El quid de la cuesti贸n es que Flarum genera elementos virtuales del DOM que son una representaci贸n de JavaScript del HTML. Mithril toma estos elementos virtuales del DOM y los convierte en HTML real de la manera m谩s eficiente posible. (隆Por eso Flarum es tan r谩pido!)

Debido a que la interfaz est谩 construida con JavaScript, es realmente f谩cil engancharse y hacer cambios. Todo lo que tienes que hacer es encontrar el extensor adecuado para la parte de la interfaz que quieres cambiar, y luego a帽adir tu propio DOM virtual a la mezcla.

La mayor铆a de las partes mutables de la interfaz son en realidad *listas de elementos*. Por ejemplo:

* The controls that appear on each post (Reply, Like, Edit, Delete)
* El proceso para extender cada extensi贸n comunitaria es diferente; debe consultar la documentaci贸n de cada extensi贸n individual.
* Los elementos de la cabecera (B煤squeda, Notificaciones, Men煤 de usuario)

Cada elemento de estas listas recibe un **nombre** para que puedas a帽adir, eliminar y reorganizar los elementos f谩cilmente. Simplemente encuentre el componente apropiado para la parte de la interfaz que desea cambiar, y monkey-patch sus m茅todos para modificar el contenido de la lista de elementos. Por ejemplo, para a帽adir un enlace a Google en la cabecera:

```jsx
import { extend } from 'flarum/extend';
import HeaderPrimary from 'flarum/components/HeaderPrimary';

extend(HeaderPrimary.prototype, 'items', function(items) {
  items.add('google', <a href="https://google.com">Google</a>);
});
```

No est谩 mal. Sin duda, nuestros usuarios har谩n cola para agradecernos un acceso tan r谩pido y c贸modo a Google.

En el ejemplo anterior, utilizamos la utilidad `extend` (explicada m谩s adelante) para a帽adir HTML a la salida de `HeaderPrimary.prototype.items()`. 驴C贸mo funciona esto realmente? Bueno, primero tenemos que entender lo que es HeaderPrimary.

## Componentes

La interfaz de Flarum se compone de muchos **componentes** anidados. Los componentes son un poco como los elementos de HTML, ya que encapsulan el contenido y el comportamiento. Por ejemplo, mira este 谩rbol simplificado de los componentes que conforman una p谩gina de discusi贸n:

```
DiscussionPage
鈹溾攢鈹? DiscussionList (the side pane)
鈹?   鈹溾攢鈹? DiscussionListItem
鈹?   鈹斺攢鈹? DiscussionListItem
鈹溾攢鈹? DiscussionHero (the title)
鈹溾攢鈹? PostStream
鈹?   鈹溾攢鈹? Post
鈹?   鈹斺攢鈹? Post
鈹溾攢鈹? SplitDropdown (the reply button)
鈹斺攢鈹? PostStreamScrubber
```

Deber铆as familiarizarte con la [API de componentes de Mithril](https://mithril.js.org/components.html) y el [sistema de redraw](https://mithril.js.org/autoredraw.html). Flarum envuelve los componentes en la clase `flarum/Component`, que extiende la [clase componentes](https://mithril.js.org/components.html#classes) de Mithril. Proporciona las siguientes ventajas:

* Los controles que aparecen en cada entrada (Responder, Me gusta, Editar, Borrar)
* El m茅todo est谩tico `initAttrs` muta `this.attrs` antes de establecerlos, y te permite establecer valores por defecto o modificarlos de alguna manera antes de usarlos en tu clase. Ten en cuenta que esto no afecta al `vnode.attrs` inicial.
* El m茅todo `$` devuelve un objeto jQuery para el elemento DOM ra铆z del componente. Opcionalmente se puede pasar un selector para obtener los hijos del DOM.
* el m茅todo est谩tico `component` puede ser utilizado como una alternativa a JSX y al hyperscript `m`. Los siguientes son equivalentes:
  * `m(CustomComponentClass, attrs, children)`
  * `CustomComponentClass.component(attrs, children)`
  * `<CustomComponentClass {...attrs}>{children}</CustomComponentClass>`

Sin embargo, las clases de componentes que extienden `Component` deben llamar a `super` cuando utilizan los m茅todos `oninit`, `oncreate` y `onbeforeupdate`.

Volvamos al ejemplo original de "a帽adir un enlace a Google en la cabecera" para demostrarlo.

Todas las dem谩s propiedades de los componentes Mithril, incluidos los [m茅todos del ciclo de vida](https://mithril.js.org/lifecycle-methods.html) (con los que deber铆a familiarizarse), se conservan. Teniendo esto en cuenta, una clase de componente personalizada podr铆a tener este aspecto:

```jsx
import Component from 'flarum/Component';

class Counter extends Component {
  oninit(vnode) {
    super.oninit(vnode);

    this.count = 0;
  }

  view() {
    return (
      <div>
        Count: {this.count}
        <button onclick={e => this.count++}>
          {this.attrs.buttonLabel}
        </button>
      </div>
    );
  }

  oncreate(vnode) {
    super.oncreate(vnode);

    // En realidad no estamos haciendo nada aqu铆, pero este ser铆a
    // un buen lugar para adjuntar manejadores de eventos, inicializar librer铆as
    // como sortable, o hacer otras modificaciones en el DOM.
    $element = this.$();
    $button = this.$('button');
  }
}

m.mount(document.body, <MyComponent buttonLabel="Increment" />);
```

## Cambiando la UI Parte 2

Ahora que tenemos una mejor comprensi贸n del sistema de componentes, vamos a profundizar un poco m谩s en c贸mo funciona la ampliaci贸n de la interfaz de usuario.

### ItemList

Como se ha indicado anteriormente, la mayor铆a de las partes f谩cilmente extensibles de la interfaz de usuario le permiten extender m茅todos llamados `items` o algo similar (por ejemplo, `controlItems`, `accountItems`, `toolbarItems`, etc. Los nombres exactos dependen del componente que est茅s extendiendo) para a帽adir, eliminar o reemplazar elementos. Bajo la superficie, estos m茅todos devuelven una instancia de `utils/ItemList`, que es esencialmente un objeto ordenado. La documentaci贸n detallada de sus m茅todos est谩 disponible en [nuestra documentaci贸n de la API](https://api.docs.flarum.org/js/master/class/src/common/utils/itemlist.ts~itemlist). Cuando se llama al m茅todo `toArray` de ItemList, los elementos se devuelven en orden ascendente de prioridad (0 si no se proporciona), y luego por clave alfab茅ticamente cuando las prioridades son iguales.

### Utilidades de Flarum

Casi todas las extensiones del frontend utilizan [monkey patching](https://en.wikipedia.org/wiki/Monkey_patch) para a帽adir, modificar o eliminar comportamientos. Por ejemplo:

```jsx
// Esto a帽ade un atributo al global `app`.
app.googleUrl = "https://google.com";

// Esto reemplaza la salida de la p谩gina de discusi贸n con "Hello World"
import DiscussionPage from 'flarum/components/DiscussionPage';

DiscussionPage.prototype.view = function() {
  return <p>Hello World</p>;
}
```

convertir谩 las p谩ginas de discusi贸n de Flarum en anuncios de "Hola Mundo". 隆Qu茅 creativo!

En la mayor铆a de los casos, no queremos reemplazar completamente los m茅todos que estamos modificando. Por esta raz贸n, Flarum incluye las utilidades `extend` y `override`. `extend` nos permite a帽adir c贸digo para que se ejecute despu茅s de que un m茅todo se haya completado. La funci贸n `override` nos permite reemplazar un m茅todo por uno nuevo, manteniendo el m茅todo anterior disponible como callback. Ambas son funciones que toman 3 argumentos:

1. El prototipo de una clase (o alg煤n otro objeto extensible)
2. El nombre de cadena de un m茅todo de esa clase
3. Un callback que realiza la modificaci贸n.
   1. En el caso de `extend`, la llamada de retorno recibe la salida del m茅todo original, as铆 como cualquier argumento pasado al m茅todo original.
   2. Para `override`, el callback recibe un callable (que puede ser usado para llamar al m茅todo original), as铆 como cualquier argumento pasado al m茅todo original.

:::tip Overriding multiple methods

With `extend` and `override`, you can also pass an array of multiple methods that you want to patch. This will apply the same modifications to all of the methods you provide:

```jsx
extend(IndexPage.prototype, ['oncreate', 'onupdate'], () => { /* your logic */ });
```

:::

Ten en cuenta que si intentas cambiar la salida de un m茅todo con `override`, debes devolver la nueva salida. Si est谩s cambiando la salida con `extend`, simplemente debes modificar la salida original (que se recibe como primer argumento). Ten en cuenta que `extend` s贸lo puede mutar la salida si 茅sta es mutable (por ejemplo, un objeto o un array, y no un n煤mero/cadena).

Let's now revisit the original "adding a link to Google to the header" example to demonstrate.

```jsx
import { extend, override } from 'flarum/extend';
import HeaderPrimary from 'flarum/components/HeaderPrimary';
import ItemList from 'flarum/utils/ItemList';
import CustomComponentClass from './components/CustomComponentClass';

// Aqu铆, a帽adimos un elemento a la lista de elementos devuelta. Estamos utilizando un componente personalizado
// como se ha comentado anteriormente. Tambi茅n hemos especificado una prioridad como tercer argumento,
// que se utilizar谩 para ordenar estos elementos. Ten en cuenta que no necesitamos devolver nada.
extend(HeaderPrimary.prototype, 'items', function(items) {
  items.add(
    'google',
    <CustomComponentClass>
      <a href="https://google.com">Google</a>
    </CustomComponentClass>,
    5
  );
});

// Aqu铆, utilizamos condicionalmente la salida original de un m茅todo,
// o creamos nuestro propio ItemList, y luego a帽adimos un elemento a 茅l.
// Ten en cuenta que DEBEMOS devolver nuestra salida personalizada.
override(HeaderPrimary.prototype, 'items', function(original) {
  let items;

  if (someArbitraryCondition) {
    items = original();
  } else {
    items = new ItemList();
  }

  items.add('google', <a href="https://google.com">Google</a>);

  return items;
});
```

Dado que todos los componentes y utilidades de Flarum est谩n representados por clases, `extend`, `override`, y el t铆pico JS significa que podemos enganchar o reemplazar cualquier m茅todo en cualquier parte de Flarum. Algunos usos potenciales "avanzados" incluyen:

* Extender o anular `view` para cambiar (o redefinir completamente) la estructura html de los componentes de Flarum. Esto abre a Flarum a una tematizaci贸n ilimitada
* Engancharse a los m茅todos de los componentes Mithril para a帽adir escuchas de eventos JS, o redefinir la l贸gica del negocio.

### Flarum Utils

Flarum define (y proporciona) bastantes funciones de ayuda y utilidades, que puede querer utilizar en sus extensiones. A few particularly useful ones:

- `flarum/common/utils/Stream` provides [Mithril Streams](https://mithril.js.org/stream.html), and is useful in [forms](forms.md).
- Engancharse a los m茅todos de los componentes Mithril para a帽adir escuchas de eventos JS, o redefinir la l贸gica del negocio.
- `flarum/common/utils/extractText` extracts text as a string from Mithril component vnode instances (or translation vnodes).
- `flarum/common/utils/throttleDebounce` provides the [throttle-debounce](https://www.npmjs.com/package/throttle-debounce) library
- `flarum/common/helpers/avatar` displays a user's avatar
- `flarum/common/helpers/highlight` highlights text in strings: great for search results!
- `flarum/common/helpers/icon` displays an icon, usually used for FontAwesome.
- `flarum/common/helpers/username` shows a user's display name, or "deleted" text if the user has been deleted.

And there's a bunch more! La mejor manera de conocerlas es a trav茅s de [el c贸digo fuente](https://github.com/flarum/core/tree/master/js) o [nuestra documentaci贸n de la API de javascript](https://api.docs.flarum.org/js/).
