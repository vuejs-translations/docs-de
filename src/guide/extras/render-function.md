---
outline: deep
---

# Renderfunktionen & JSX {#render-functions-jsx}

Vue empfiehlt in den allermeisten Fällen die Verwendung von Templates zur Erstellung von Anwendungen. Es gibt jedoch Situationen, in denen wir die volle programmatische Kraft von JavaScript benötigen. Hier können wir die **render-Funktion** verwenden.

> Wenn Sie mit dem Konzept von virtuellem DOM und Rendering-Funktionen nicht vertraut sind, sollten Sie zuerst das Kapitel [Rendering-Mechanismus](/guide/extras/rendering-mechanism.html) lesen.

## Grundlegende Verwendung {#basic-usage}

### Vnodes erstellen {#creating-vnodes}

Vue bietet eine `h()` Funktion zum Erstellen von vnodes:

```js
import { h } from 'vue'

const vnode = h(
  'div', // type
  { id: 'foo', class: 'bar' }, // props
  [
    /* children */
  ]
)
```

h()` ist die Abkürzung für **hyperscript** - das bedeutet „JavaScript, das HTML (Hypertext Markup Language) erzeugt“. Dieser Name wurde von Konventionen übernommen, die von vielen virtuellen DOM-Implementierungen geteilt werden. Ein aussagekräftigerer Name wäre `createVnode()`, aber ein kürzerer Name ist hilfreich, wenn Sie diese Funktion in einer Rendering-Funktion viele Male aufrufen müssen.

Die Funktion `h()` ist so konzipiert, dass sie sehr flexibel ist:

```js
// all arguments except the type are optional
h('div')
h('div', { id: 'foo' })

// both attributes and properties can be used in props
// Vue automatically picks the right way to assign it
h('div', { class: 'bar', innerHTML: 'hello' })

// props modifiers such as .prop and .attr can be added
// with '.' and `^' prefixes respectively
h('div', { '.name': 'some-name', '^width': '100' })

// class and style have the same object / array
// value support that they have in templates
h('div', { class: [foo, { bar }], style: { color: 'red' } })

// event listeners should be passed as onXxx
h('div', { onClick: () => {} })

// children can be a string
h('div', { id: 'foo' }, 'hello')

// props can be omitted when there are no props
h('div', 'hello')
h('div', [h('span', 'hello')])

// children array can contain mixed vnodes and strings
h('div', ['hello', h('span', 'hello')])
```

The resulting vnode has the following shape:

```js
const vnode = h('div', { id: 'foo' }, [])

vnode.type // 'div'
vnode.props // { id: 'foo' }
vnode.children // []
vnode.key // null
```

:::warning Note
Die vollständige „VNode“-Schnittstelle enthält viele weitere interne Eigenschaften, aber es wird dringend empfohlen, sich nicht auf andere als die hier aufgeführten Eigenschaften zu verlassen. Dadurch wird ein unbeabsichtigter Bruch vermieden, falls die internen Eigenschaften geändert werden.
:::

### Deklaration von Renderfunktionen {#declaring-render-functions}

<div class="composition-api">

Bei der Verwendung von Vorlagen mit der Kompositions-API wird der Rückgabewert des „setup()“-Hooks verwendet, um Daten für die Vorlage bereitzustellen. Bei der Verwendung von Render-Funktionen können wir jedoch stattdessen direkt die Render-Funktion zurückgeben:

```js
import { ref, h } from 'vue'

export default {
  props: {
    /* ... */
  },
  setup(props) {
    const count = ref(1)

    // return the render function
    return () => h('div', props.msg + count.value)
  }
}
```

Die Render-Funktion wird innerhalb von `setup()` deklariert, so dass sie natürlich Zugriff auf die Requisiten und jeden reaktiven Zustand hat, der im selben Bereich deklariert ist.

Zusätzlich zur Rückgabe eines einzelnen Vnode können Sie auch Strings oder Arrays zurückgeben:

```js
export default {
  setup() {
    return () => 'hello world!'
  }
}
```

```js
import { h } from 'vue'

export default {
  setup() {
    // use an array to return multiple root nodes
    return () => [
      h('div'),
      h('div'),
      h('div')
    ]
  }
}
```

:::tip
Vergewissern Sie sich, dass Sie eine Funktion zurückgeben, anstatt direkt Werte zurückzugeben! Die Funktion `setup()` wird nur einmal pro Komponente aufgerufen, während die zurückgegebene Render-Funktion mehrfach aufgerufen wird.
:::

</div>
<div class="options-api">

Wir können Render-Funktionen mit der Option `render` deklarieren:

```js
import { h } from 'vue'

export default {
  data() {
    return {
      msg: 'hello'
    }
  },
  render() {
    return h('div', this.msg)
  }
}
```

Die Funktion `render()` hat über `this` Zugriff auf die Komponenteninstanz.

Zusätzlich zur Rückgabe eines einzelnen Vnodes können Sie auch Strings oder Arrays zurückgeben:

```js
export default {
  render() {
    return 'hello world!'
  }
}
```

```js
import { h } from 'vue'

export default {
  render() {
    // use an array to return multiple root nodes
    return [
      h('div'),
      h('div'),
      h('div')
    ]
  }
}
```

</div>

Wenn eine Rendering-Funktionskomponente keinen Instanzstatus benötigt, kann sie der Kürze halber auch direkt als Funktion deklariert werden:

```js
function Hello() {
  return 'hello world!'
}
```

Das ist richtig, dies ist eine gültige Vue Komponente! Siehe [Functional Components](#functional-components) für weitere Details zu dieser Syntax.

### Vnodes müssen einmalig sein {#vnodes-must-be-unique}

Alle vnodes im Komponentenbaum müssen eindeutig sein. Das bedeutet, dass die folgende Renderfunktion ungültig ist:

```js
function render() {
  const p = h('p', 'hi')
  return h('div', [
    // Yikes - duplicate vnodes!
    p,
    p
  ])
}
```

Wenn Sie dasselbe Element bzw. dieselbe Komponente wirklich mehrmals duplizieren wollen, können Sie dies mit einer Fabrikfunktion tun. Zum Beispiel ist die folgende Render-Funktion eine absolut gültige Methode, um 20 identische Absätze zu rendern:

```js
function render() {
  return h(
    'div',
    Array.from({ length: 20 }).map(() => {
      return h('p', 'hi')
    })
  )
}
```

## JSX / TSX {#jsx-tsx}

[JSX](https://facebook.github.io/jsx/) ist eine XML-ähnliche Erweiterung von JavaScript, die es uns ermöglicht, Code wie diesen zu schreiben:

```jsx
const vnode = <div>hello</div>
```

Verwenden Sie innerhalb von JSX-Ausdrücken geschweifte Klammern, um dynamische Werte einzubetten:

```jsx
const vnode = <div id={dynamicId}>hello, {userName}</div>
```

Sowohl `create-vue` als auch Vue CLI haben Optionen für das Scaffolding von Projekten mit vorkonfigurierter JSX Unterstützung. Wenn Sie JSX manuell konfigurieren, schauen Sie bitte in die Dokumentation von [`@vue/babel-plugin-jsx`](https://github.com/vuejs/jsx-next) für Details.

Obwohl JSX zuerst von React eingeführt wurde, hat es keine definierte Laufzeitsemantik und kann in verschiedene Ausgaben kompiliert werden. Wenn Sie bereits mit JSX gearbeitet haben, beachten Sie bitte, dass **Vue JSX transform sich von Reacts JSX transform unterscheidet**, so dass Sie Reacts JSX transform nicht in Vue-Anwendungen verwenden können. Einige bemerkenswerte Unterschiede zu React JSX sind:

- Sie können HTML-Attribute wie `class` und `for` als Requisiten verwenden - Sie müssen nicht `className` oder `htmlFor` verwenden.
- Übergabe von Kindern an Komponenten (d.h. Slots) [funktioniert anders](#passing-slots).

Vue's Typdefinition bietet auch Typinferenz für die Verwendung von TSX. Wenn man TSX benutzt, muss man `„jsx“ angeben: „preserve"` in `tsconfig.json`, so dass TypeScript die JSX-Syntax intakt lässt, damit Vue JSX transform verarbeiten kann.

## Rezepte für Renderfunktionen {#render-function-recipes}

Im Folgenden finden Sie einige gängige Rezepte für die Implementierung von Template-Funktionen als entsprechende Renderfunktionen / JSX.

### `v-if` {#v-if}

Template:

```vue-html
<div>
  <div v-if="ok">yes</div>
  <span v-else>no</span>
</div>
```

Äquivalente Renderfunktion / JSX:

<div class="composition-api">

```js
h('div', [ok.value ? h('div', 'yes') : h('span', 'no')])
```

```jsx
<div>{ok.value ? <div>yes</div> : <span>no</span>}</div>
```

</div>
<div class="options-api">

```js
h('div', [this.ok ? h('div', 'yes') : h('span', 'no')])
```

```jsx
<div>{this.ok ? <div>yes</div> : <span>no</span>}</div>
```

</div>

### `v-for` {#v-for}

Template:

```vue-html
<ul>
  <li v-for="{ id, text } in items" :key="id">
    {{ text }}
  </li>
</ul>
```

Equivalent render function / JSX:

<div class="composition-api">

```js
h(
  'ul',
  // assuming `items` is a ref with array value
  items.value.map(({ id, text }) => {
    return h('li', { key: id }, text)
  })
)
```

```jsx
<ul>
  {items.value.map(({ id, text }) => {
    return <li key={id}>{text}</li>
  })}
</ul>
```

</div>
<div class="options-api">

```js
h(
  'ul',
  this.items.map(({ id, text }) => {
    return h('li', { key: id }, text)
  })
)
```

```jsx
<ul>
  {this.items.map(({ id, text }) => {
    return <li key={id}>{text}</li>
  })}
</ul>
```

</div>

### `v-on` {#v-on}

Requisiten, deren Namen mit „on“, gefolgt von einem Großbuchstaben, beginnen, werden als Ereignis-Listener behandelt. Zum Beispiel ist „onClick“ das Äquivalent von „@click“ in Vorlagen.

```js
h(
  'button',
  {
    onClick(event) {
      /* ... */
    }
  },
  'click me'
)
```

```jsx
<button
  onClick={(event) => {
    /* ... */
  }}
>
  click me
</button>
```

#### Ereignis-Modifikatoren {#event-modifiers}

Bei den Ereignismodifikatoren `.passive`, `.capture` und `.once` können sie nach dem Ereignisnamen unter Verwendung von camelCase aneinandergereiht werden.

Zum Beispiel:

```js
h('input', {
  onClickCapture() {
    /* listener in capture mode */
  },
  onKeyupOnce() {
    /* triggers only once */
  },
  onMouseoverOnceCapture() {
    /* once + capture */
  }
})
```

```jsx
<input
  onClickCapture={() => {}}
  onKeyupOnce={() => {}}
  onMouseoverOnceCapture={() => {}}
/>
```

Für andere Ereignis- und Tastenmodifikatoren kann die Hilfsfunktion [`withModifiers`](/api/render-function.html#withmodifiers) verwendet werden:

```js
import { withModifiers } from 'vue'

h('div', {
  onClick: withModifiers(() => {}, ['self'])
})
```

```jsx
<div onClick={withModifiers(() => {}, ['self'])} />
```

### Komponenten {#components}

Um einen Vnode für eine Komponente zu erstellen, sollte das erste Argument, das an `h()` übergeben wird, die Komponentendefinition sein. Das bedeutet, dass bei der Verwendung von Render-Funktionen keine Komponenten registriert werden müssen - Sie können einfach die importierten Komponenten direkt verwenden:

```js
import Foo from './Foo.vue'
import Bar from './Bar.jsx'

function render() {
  return h('div', [h(Foo), h(Bar)])
}
```

```jsx
function render() {
  return (
    <div>
      <Foo />
      <Bar />
    </div>
  )
}
```

Wie wir sehen können, kann „h“ mit Komponenten arbeiten, die aus einem beliebigen Dateiformat importiert wurden, solange es eine gültige Vue-Komponente ist.

Dynamische Komponenten sind mit Render-Funktionen sehr einfach:

```js
import Foo from './Foo.vue'
import Bar from './Bar.jsx'

function render() {
  return ok.value ? h(Foo) : h(Bar)
}
```

```jsx
function render() {
  return ok.value ? <Foo /> : <Bar />
}
```

Wenn eine Komponente namentlich registriert ist und nicht direkt importiert werden kann (z. B. wenn sie von einer Bibliothek global registriert wurde), kann sie programmatisch aufgelöst werden, indem die Hilfsfunktion [`resolveComponent()`](/api/render-function.html#resolvecomponent) verwendet wird.

### Rendering-Slots {#rendering-slots}

<div class="composition-api">

In Render-Funktionen kann auf Slots über den Kontext „setup()“ zugegriffen werden. Jeder Slot des Objekts „slots“ ist eine **Funktion, die ein Array von vnodes** zurückgibt:

```js
export default {
  props: ['message'],
  setup(props, { slots }) {
    return () => [
      // default slot:
      // <div><slot /></div>
      h('div', slots.default()),

      // named slot:
      // <div><slot name="footer" :text="message" /></div>
      h(
        'div',
        slots.footer({
          text: props.message
        })
      )
    ]
  }
}
```

JSX gleichwertig:

```jsx
// default
<div>{slots.default()}</div>

// named
<div>{slots.footer({ text: props.message })}</div>
```

</div>
<div class="options-api">

In Render-Funktionen kann auf Slots zugegriffen werden von [`this.$slots`](/api/component-instance.html#slots):

```js
export default {
  props: ['message'],
  render() {
    return [
      // <div><slot /></div>
      h('div', this.$slots.default()),

      // <div><slot name="footer" :text="message" /></div>
      h(
        'div',
        this.$slots.footer({
          text: this.message
        })
      )
    ]
  }
}
```

JSX gleichwertig:

```jsx
// <div><slot /></div>
<div>{this.$slots.default()}</div>

// <div><slot name="footer" :text="message" /></div>
<div>{this.$slots.footer({ text: this.message })}</div>
```

</div>

### Passierschächte {#passing-slots}

Die Übergabe von Kindern an Komponenten funktioniert ein wenig anders als die Übergabe von Kindern an Elemente. Anstelle eines Arrays müssen wir entweder eine Slot-Funktion oder ein Objekt von Slot-Funktionen übergeben. Slot-Funktionen können alles zurückgeben, was eine normale Render-Funktion zurückgeben kann - die immer zu Arrays von Vnodes normalisiert werden, wenn auf sie in der untergeordneten Komponente zugegriffen wird.

```js
// single default slot
h(MyComponent, () => 'hello')

// named slots
// notice the `null` is required to avoid
// the slots object being treated as props
h(MyComponent, null, {
  default: () => 'default slot',
  foo: () => h('div', 'foo'),
  bar: () => [h('span', 'one'), h('span', 'two')]
})
```

JSX equivalent:

```jsx
// default
<MyComponent>{() => 'hello'}</MyComponent>

// named
<MyComponent>{{
  default: () => 'default slot',
  foo: () => <div>foo</div>,
  bar: () => [<span>one</span>, <span>two</span>]
}}</MyComponent>
```

Die Übergabe von Slots als Funktionen ermöglicht es, dass sie von der untergeordneten Komponente nachlässig aufgerufen werden. Dies führt dazu, dass die Abhängigkeiten des Slots von der untergeordneten Komponente und nicht von der übergeordneten Komponente verfolgt werden, was zu genaueren und effizienteren Aktualisierungen führt.

### Eingebaute Komponenten {#built-in-components}

[Eingebaute Komponenten](/api/built-in-components.html) wie `<KeepAlive>`, `<Transition>`, `<TransitionGroup>`, `<Teleport>` und `<Suspense>` müssen für die Verwendung in Renderfunktionen importiert werden:

<div class="composition-api">

```js
import { h, KeepAlive, Teleport, Transition, TransitionGroup } from 'vue'

export default {
  setup () {
    return () => h(Transition, { mode: 'out-in' }, /* ... */)
  }
}
```

</div>
<div class="options-api">

```js
import { h, KeepAlive, Teleport, Transition, TransitionGroup } from 'vue'

export default {
  render () {
    return h(Transition, { mode: 'out-in' }, /* ... */)
  }
}
```

</div>

### `v-model` {#v-model}

Die Direktive „v-model“ wird während der Kompilierung der Vorlage zu den Requisiten „modelValue“ und „onUpdate:modelValue“ erweitert - wir müssen diese Requisiten selbst bereitstellen:

<div class="composition-api">

```js
export default {
  props: ['modelValue'],
  emits: ['update:modelValue'],
  setup(props, { emit }) {
    return () =>
      h(SomeComponent, {
        modelValue: props.modelValue,
        'onUpdate:modelValue': (value) => emit('update:modelValue', value)
      })
  }
}
```

</div>
<div class="options-api">

```js
export default {
  props: ['modelValue'],
  emits: ['update:modelValue'],
  render() {
    return h(SomeComponent, {
      modelValue: this.modelValue,
      'onUpdate:modelValue': (value) => this.$emit('update:modelValue', value)
    })
  }
}
```

</div>

### Benutzerdefinierte Direktiven {#custom-directives}

Benutzerdefinierte Direktiven können auf einen vnode angewendet werden, indem [`withDirectives`](/api/render-function.html#withdirectives):

```js
import { h, withDirectives } from 'vue'

// a custom directive
const pin = {
  mounted() { /* ... */ },
  updated() { /* ... */ }
}

// <div v-pin:top.animate="200"></div>
const vnode = withDirectives(h('div'), [
  [pin, 200, 'top', { animate: true }]
])
```

Wenn die Direktive namentlich registriert ist und nicht direkt importiert werden kann, kann sie mit der Hilfsfunktion [`resolveDirective`](/api/render-function.html#resolvedirective) aufgelöst werden.

## Funktionelle Komponenten {#functional-components}

Funktionale Komponenten sind eine alternative Form von Komponenten, die keinen eigenen Zustand haben. Sie verhalten sich wie reine Funktionen: props in, vnodes out. Sie werden gerendert, ohne eine Komponenteninstanz zu erzeugen (d.h. kein `this`), und ohne die üblichen Komponenten-Lebenszyklus-Haken.

Um eine funktionale Komponente zu erstellen, verwenden wir eine einfache Funktion, anstatt eines options-Objekts. Die Funktion ist quasi die „render“-Funktion für die Komponente.

<div class="composition-api">

Die Signatur einer funktionalen Komponente ist die gleiche wie die des `setup()`-Hooks:

```js
function MyComponent(props, { slots, emit, attrs }) {
  // ...
}
```

</div>
<div class="options-api">

Da es keine `this`-Referenz für eine funktionale Komponente gibt, wird Vue die `props` als erstes Argument übergeben:

```js
function MyComponent(props, context) {
  // ...
}
```

Das zweite Argument, `context`, enthält drei Eigenschaften: `attrs`, `emit`, und `slots`. Diese sind äquivalent zu den Instanz-Eigenschaften [`$attrs`](/api/component-instance.html#attrs), [`$emit`](/api/component-instance.html#emit) bzw. [`$slots`](/api/component-instance.html#slots).

</div>

Die meisten der üblichen Konfigurationsoptionen für Komponenten sind für funktionale Komponenten nicht verfügbar. Es ist jedoch möglich, [`props`](/api/options-state.html#props) und [`emits`](/api/options-state.html#emits) zu definieren, indem sie als Eigenschaften hinzugefügt werden:

```js
MyComponent.props = ['value']
MyComponent.emits = ['click']
```

Wenn die Option „props“ nicht angegeben wird, enthält das an die Funktion übergebene Objekt „props“ alle Attribute, genau wie „attrs“. Die Prop-Namen werden nicht in camelCase normalisiert, es sei denn, die Option `props` ist angegeben.

Für funktionale Komponenten mit expliziten `props`, funktioniert [attribute fallthrough](/guide/components/attrs.html) genauso wie bei normalen Komponenten. Für funktionale Komponenten, die ihre `props` nicht explizit angeben, werden jedoch nur die `class`, `style` und `onXxx` Event-Listener standardmäßig von den `attrs` geerbt. In beiden Fällen kann `inheritAttrs` auf `false` gesetzt werden, um die Vererbung von Attributen zu deaktivieren:

```js
MyComponent.inheritAttrs = false
```

Funktionale Komponenten können genau wie normale Komponenten registriert und konsumiert werden. Wenn Sie eine Funktion als erstes Argument an `h()` übergeben, wird sie als funktionale Komponente behandelt.
