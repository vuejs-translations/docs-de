# Reaktivität Transformieren {#reactivity-transform}

:::warning Experimentelles Merkmal
Reactivity Transform ist derzeit ein Experimentelles Merkmal. Es ist standardmäßig deaktiviert und erfordert [explizites Opt-in](#explicit-opt-in). Es kann sich auch noch ändern, bevor es finalisiert wird. Um auf dem Laufenden zu bleiben, behalten Sie den [Vorschlag und die Diskussion auf GitHub](https://github.com/vuejs/rfcs/discussions/369) im Auge.
:::

:::tip Composition-API-specific
Reactivity Transform ist eine Composition-API-spezifische Funktion und erfordert einen Build-Schritt.
:::

## Refs vs. Reaktive Variablen {#refs-vs-reactive-variables}

Seit der Einführung der Composition API ist eine der wichtigsten ungelösten Fragen die Verwendung von Refs im Vergleich zu reaktiven Objekten. Bei der Destrukturierung reaktiver Objekte verliert man leicht die Reaktivität, während es bei der Verwendung von refs umständlich sein kann, überall `.value` zu verwenden. Außerdem ist `.value` leicht zu übersehen, wenn man kein Typsystem verwendet.

[Vue Reactivity Transform](https://github.com/vuejs/core/tree/main/packages/reactivity-transform) ist eine Kompilierzeit-Transformation, die es uns erlaubt, Code wie diesen zu schreiben:

```vue
<script setup>
let count = $ref(0)

console.log(count)

function increment() {
  count++
}
</script>

<template>
  <button @click="increment">{{ count }}</button>
</template>
```

Die Methode „$ref()“ ist ein **Compile-Time-Macro**: Es ist keine tatsächliche Methode, die zur Laufzeit aufgerufen wird. Stattdessen verwendet der Vue-Compiler sie als Hinweis, um die resultierende Variable `count` als **reaktive Variable** zu behandeln.

Auf reaktive Variablen kann wie auf normale Variablen zugegriffen werden und sie können neu zugewiesen werden, aber diese Operationen werden in refs mit `.value` kompiliert. Zum Beispiel wird der `<script>` Teil der obigen Komponente kompiliert in:

```js{5,8}
import { ref } from 'vue'

let count = ref(0)

console.log(count.value)

function increment() {
  count.value++
}
```

Jede Reaktivitäts-API, die Refs zurückgibt, hat ein Makroäquivalent mit dem Präfix „$“. Diese APIs umfassen:

- [`ref`](/api/reactivity-core.html#ref) -> `$ref`
- [`computed`](/api/reactivity-core.html#computed) -> `$computed`
- [`shallowRef`](/api/reactivity-advanced.html#shallowref) -> `$shallowRef`
- [`customRef`](/api/reactivity-advanced.html#customref) -> `$customRef`
- [`toRef`](/api/reactivity-utilities.html#toref) -> `$toRef`

Diese Makros sind global verfügbar und müssen nicht importiert werden, wenn Reactivity Transform aktiviert ist, aber Sie können sie optional von `vue/macros` importieren, wenn Sie expliziter sein wollen:

```js
import { $ref } from 'vue/macros'

let count = $ref(0)
```

## Umstrukturierung mit `$()` {#destructuring-with}

Es ist üblich, dass eine Kompositionsfunktion ein Objekt mit Refs zurückgibt und die Destrukturierung verwendet, um diese Refs abzurufen. Zu diesem Zweck bietet reactivity transform das Makro **`$()`**:

```js
import { useMouse } from '@vueuse/core'

const { x, y } = $(useMouse())

console.log(x, y)
```

Kompilierte Ausgabe:

```js
import { toRef } from 'vue'
import { useMouse } from '@vueuse/core'

const __temp = useMouse(),
  x = toRef(__temp, 'x'),
  y = toRef(__temp, 'y')

console.log(x.value, y.value)
```

Wenn `x` bereits ein ref ist, gibt `toRef(__temp, 'x')` ihn einfach so zurück, wie er ist, und es wird kein zusätzlicher ref erzeugt. Wenn ein destrukturierter Wert kein ref ist (z.B. eine Funktion), wird er trotzdem funktionieren - der Wert wird in einen ref eingepackt, so dass der Rest des Codes wie erwartet funktioniert.

`$()` destructure funktioniert sowohl bei reaktiven Objekten **als auch** bei einfachen Objekten, die Refs enthalten.

## Vorhandene Refs in reaktive Variablen umwandeln mit `$()` {#convert-existing-refs-to-reactive-variables-with}

In einigen Fällen können wir umhüllte Funktionen haben, die auch Refs zurückgeben. Der Vue-Compiler wird jedoch nicht in der Lage sein, im Voraus zu wissen, dass eine Funktion eine Referenz zurückgeben wird. In solchen Fällen kann das `$()` Makro auch verwendet werden, um vorhandene refs in reaktive Variablen zu konvertieren:

```js
function myCreateRef() {
  return ref(0)
}

let count = $(myCreateRef())
```

## Reaktive Requisiten Destrukturierung {#reactive-props-destructure}

Es gibt zwei Probleme mit der derzeitigen Verwendung von „DefineProps()“ in „<Skript Setup>“:

1. Ähnlich wie bei `.value` muss man immer auf props als `props.x` zugreifen, um die Reaktivität zu erhalten. Das bedeutet, dass man `defineProps` nicht destrukturieren kann, weil die resultierenden destrukturierten Variablen nicht reaktiv sind und nicht aktualisiert werden.

2. Bei Verwendung der [type-only props declaration](/api/sfc-script-setup.html#typescript-only-features) gibt es keine einfache Möglichkeit, Standardwerte für die Requisiten zu deklarieren. Wir haben die `withDefaults()` API für genau diesen Zweck eingeführt, aber sie ist immer noch umständlich in der Anwendung.

Wir können diese Probleme angehen, indem wir eine Kompilierzeittransformation anwenden, wenn `defineProps` mit Destrukturierung verwendet wird, ähnlich dem, was wir zuvor mit `$()` gesehen haben:

```html
<script setup lang="ts">
  interface Props {
    msg: string
    count?: number
    foo?: string
  }

  const {
    msg,
    // Standardwert funktioniert einfach
    count = 1,
    // Lokales Aliasing funktioniert auch einfach
    // hier aliasieren wir `props.foo` zu `bar`
    foo: bar
  } = defineProps<Props>()

  watchEffect(() => {
    // protokolliert jede Änderung der Requisiten
    console.log(msg, count, bar)
  })
</script>
```

Die obigen Angaben werden in das folgende Äquivalent für die Laufzeitdeklaration kompiliert:

```js
export default {
  props: {
    msg: { type: String, required: true },
    count: { type: Number, default: 1 },
    foo: String
  },
  setup(props) {
    watchEffect(() => {
      console.log(props.msg, props.count, props.foo)
    })
  }
}
```

## Beibehaltung der Reaktivität über Funktionsgrenzen hinweg {#retaining-reactivity-across-function-boundaries}

Während reaktive Variablen uns davon entlasten, überall `.value` verwenden zu müssen, entsteht ein Problem des „Reaktivitätsverlustes“, wenn wir reaktive Variablen über Funktionsgrenzen hinweg übergeben. Dies kann in zwei Fällen geschehen:

### Übergabe an die Funktion als Argument {#passing-into-function-as-argument}

Bei einer Funktion, die ein ref als Argument erwartet, z.B.:

```ts
function trackChange(x: Ref<number>) {
  watch(x, (x) => {
    console.log('x changed!')
  })
}

let count = $ref(0)
trackChange(count) // funktioniert nicht!
```

Der obige Fall wird nicht wie erwartet funktionieren, weil er zu kompilieren:

```ts
let count = ref(0)
trackChange(count.value)
```

Hier wird `count.value` als Zahl übergeben, während `trackChange` eine tatsächliche Referenz erwartet. Dies kann behoben werden, indem `count` vor der Übergabe mit `$$()` umschlossen wird:

```diff
let count = $ref(0)
- trackChange(count)
+ trackChange($$(count))
```

Die obigen Angaben lassen sich wie folgt zusammenfassen:

```js
import { ref } from 'vue'

let count = ref(0)
trackChange(count)
```

Wie wir sehen können, ist `$$()` ein Makro, das als **escape-Hinweis** dient: reaktive Variablen innerhalb von `$$()` werden nicht mit `.value` versehen.

### Rückgabe innerhalb des Funktionsbereichs {#returning-inside-function-scope}

Die Reaktivität kann auch verloren gehen, wenn reaktive Variablen direkt in einem zurückgegebenen Ausdruck verwendet werden:

```ts
function useMouse() {
  let x = $ref(0)
  let y = $ref(0)

  // mousemove anhören...

  // funktioniert nicht!
  return {
    x,
    y
  }
}
```

Die obige Return-Anweisung kompiliert zu:

```ts
return {
  x: x.value,
  y: y.value
}
```

Um die Reaktivität beizubehalten, sollten wir die tatsächlichen Refs zurückgeben, nicht den aktuellen Wert zur Rückgabezeit.

Auch hier können wir `$$()` verwenden, um dies zu beheben. In diesem Fall kann `$$()` direkt auf das zurückgegebene Objekt angewendet werden - jeder Verweis auf reaktive Variablen innerhalb des `$$()`-Aufrufs behält den Verweis auf die zugrundeliegenden Refs bei:

```ts
function useMouse() {
  let x = $ref(0)
  let y = $ref(0)

  // mousemove anhören...

  // behoben
  return $$({
    x,
    y
  })
}
```

### Verwendung von „$()“ für destrukturierte Requisiten {#using-on-destructured-props}

$$$()` funktioniert auch bei destrukturierten Requisiten, da sie ebenfalls reaktive Variablen sind. Der Compiler konvertiert sie aus Effizienzgründen mit `toRef`:

```ts
const { count } = defineProps<{ count: number }>()

passAsRef($$(count))
```

compiles to:

```js
setup(props) {
  const __props_count = toRef(props, 'count')
  passAsRef(__props_count)
}
```

## TypeScript-Integration <sup class="vt-badge ts" /> {#typescript-integration}

Vue bietet Typisierungen für diese Makros (global verfügbar) und alle Typen werden wie erwartet funktionieren. Es gibt keine Inkompatibilitäten mit Standard-TypeScript-Semantik, so dass die Syntax mit allen bestehenden Tooling funktionieren wird.

Das bedeutet auch, dass die Makros in allen Dateien funktionieren können, in denen gültige JS / TS erlaubt sind - nicht nur innerhalb von Vue SFCs.

Da die Makros global verfügbar sind, müssen ihre Typen explizit referenziert werden (z.B. in einer „env.d.ts“ Datei):

```ts
/// <reference types="vue/macros-global" />
```

Wenn die Makros explizit aus `vue/macros` importiert werden, funktioniert der Typ ohne Deklaration der Globals.

## Explizites Opt-in {#explicit-opt-in}

Reactivity Transform ist derzeit standardmäßig deaktiviert und muss explizit aktiviert werden. Darüber hinaus benötigen alle der folgenden Setups `vue@^3.2.25`.

### Vite {#vite}

- Benötigt `@vitejs/plugin-vue@>=2.0.0`.
- Gilt für SFCs und js(x)/ts(x)-Dateien. Eine schnelle Verwendungsprüfung wird auf Dateien durchgeführt, bevor die Transformation angewendet wird, so dass es keine Leistungseinbußen für Dateien geben sollte, die die Makros nicht verwenden.
- Hinweis: `reactivityTransform` ist jetzt eine Option auf der Root-Ebene des Plugins und nicht mehr als `script.refSugar` verschachtelt, da sie nicht nur SFCs betrifft.

```js
// vite.config.js
export default {
  plugins: [
    vue({
      reactivityTransform: true
    })
  ]
}
```

### `vue-cli` {#vue-cli}

- Betrifft derzeit nur SFCs
- Benötigt `vue-loader@>=17.0.0`

```js
// vue.config.js
module.exports = {
  chainWebpack: (config) => {
    config.module
      .rule('vue')
      .use('vue-loader')
      .tap((options) => {
        return {
          ...options,
          reactivityTransform: true
        }
      })
  }
}
```

### Einfache `webpack` + `vue-loader` {#plain-webpack-vue-loader}

- Betrifft derzeit nur SFCs
- Benötigt `vue-loader@>=17.0.0`

```js
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: {
          reactivityTransform: true
        }
      }
    ]
  }
}
```
