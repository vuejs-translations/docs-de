# Benutzerdefinierte Direktiven {#custom-directives}

<script setup>
const vFocus = {
  mounted: el => {
    el.focus()
  }
}
</script>

## Einführung {#introduction}

Zusätzlich zu den Standard-Direktiven, die im Kern ausgeliefert werden (wie `v-model` oder `v-show`), erlaubt es Vue auch, eigene Direktiven zu registrieren.

Wir haben zwei Formen der Wiederverwendung von Code in Vue eingeführt: [components](/guide/essentials/component-basics.html) und [composables](./composables). Komponenten sind die Hauptbausteine, während Composables sich auf die Wiederverwendung von zustandsbehafteter Logik konzentrieren. Benutzerdefinierte Direktiven hingegen sind hauptsächlich für die Wiederverwendung von Logik gedacht, die einen Low-Level-DOM-Zugriff auf einfache Elemente beinhaltet.

Eine benutzerdefinierte Richtlinie ist als Objekt definiert, das Lebenszyklus-Hooks enthält, die denen einer Komponente ähneln. Die Hooks erhalten das Element, an das die Direktive gebunden ist. Hier ist ein Beispiel für eine Direktive, die eine Eingabe fokussiert, wenn das Element in das DOM von Vue eingefügt wird:

<div class="composition-api">

```vue
<script setup>
// enables v-focus in templates
const vFocus = {
  mounted: (el) => el.focus()
}
</script>

<template>
  <input v-focus />
</template>
```

</div>

<div class="options-api">

```js
const focus = {
  mounted: (el) => el.focus()
}

export default {
  directives: {
    // enables v-focus in template
    focus
  }
}
```

```vue-html
<input v-focus />
```

</div>

<div class="demo">
  <input v-focus placeholder="This should be focused" />
</div>

Angenommen, Sie haben nicht an anderer Stelle auf der Seite geklickt, sollte die obige Eingabe automatisch fokussiert werden. Diese Direktive ist nützlicher als das `autofocus`-Attribut, weil sie nicht nur beim Laden der Seite funktioniert - sie funktioniert auch, wenn das Element dynamisch von Vue eingefügt wird.

<div class="composition-api">

In `<script setup>` kann jede camelCase-Variable, die mit dem Präfix `v` beginnt, als eigene Direktive verwendet werden. Im obigen Beispiel kann „vFocus“ in der Vorlage als „v-focus“ verwendet werden.

Wenn `<script setup>` nicht verwendet wird, können benutzerdefinierte Anweisungen mit der Option `directives` registriert werden:

```js
export default {
  setup() {
    /*...*/
  },
  directives: {
    // enables v-focus in template
    focus: {
      /* ... */
    }
  }
}
```

</div>

<div class="options-api">

Ähnlich wie Komponenten müssen benutzerdefinierte Direktiven registriert werden, damit sie in Vorlagen verwendet werden können. Im obigen Beispiel verwenden wir die lokale Registrierung über die Option `Direktiven`.

</div>

Es ist auch üblich, benutzerdefinierte Anweisungen global auf App-Ebene zu registrieren:

```js
const app = createApp({})

// make v-focus usable in all components
app.directive('focus', {
  /* ... */
})
```

:::tip
Benutzerdefinierte Direktiven sollten nur verwendet werden, wenn die gewünschte Funktionalität nur durch direkte DOM-Manipulation erreicht werden kann. Bevorzugen Sie nach Möglichkeit deklaratives Template-Design mit integrierten Direktiven wie „v-bind“, da diese effizienter und serverfreundlicher sind.
:::

## Direktive Hooks {#directive-hooks}

Ein Direktivendefinitionsobjekt kann mehrere Hook-Funktionen bereitstellen (alle optional):

```js
const myDirective = {
  // called before bound element's attributes
  // or event listeners are applied
  created(el, binding, vnode, prevVnode) {
    // see below for details on arguments
  },
  // called right before the element is inserted into the DOM.
  beforeMount(el, binding, vnode, prevVnode) {},
  // called when the bound element's parent component
  // and all its children are mounted.
  mounted(el, binding, vnode, prevVnode) {},
  // called before the parent component is updated
  beforeUpdate(el, binding, vnode, prevVnode) {},
  // called after the parent component and
  // all of its children have updated
  updated(el, binding, vnode, prevVnode) {},
  // called before the parent component is unmounted
  beforeUnmount(el, binding, vnode, prevVnode) {},
  // called when the parent component is unmounted
  unmounted(el, binding, vnode, prevVnode) {}
}
```

### Hook Arguments {#hook-arguments}

Directive hooks are passed these arguments:

- `el`: the element the directive is bound to. This can be used to directly manipulate the DOM.

- `binding`: an object containing the following properties.

  - `value`: The value passed to the directive. For example in `v-my-directive="1 + 1"`, the value would be `2`.
  - `oldValue`: The previous value, only available in `beforeUpdate` and `updated`. It is available whether or not the value has changed.
  - `arg`: The argument passed to the directive, if any. For example in `v-my-directive:foo`, the arg would be `"foo"`.
  - `modifiers`: An object containing modifiers, if any. For example in `v-my-directive.foo.bar`, the modifiers object would be `{ foo: true, bar: true }`.
  - `instance`: The instance of the component where the directive is used.
  - `dir`: the directive definition object.

- `vnode`: the underlying VNode representing the bound element.
- `prevNode`: the VNode representing the bound element from the previous render. Only available in the `beforeUpdate` and `updated` hooks.

As an example, consider the following directive usage:

```vue-html
<div v-example:foo.bar="baz">
```

The `binding` argument would be an object in the shape of:

```js
{
  arg: 'foo',
  modifiers: { bar: true },
  value: /* value of `baz` */,
  oldValue: /* value of `baz` from previous update */
}
```

Similar to built-in directives, custom directive arguments can be dynamic. For example:

```vue-html
<div v-example:[arg]="value"></div>
```

Here the directive argument will be reactively updated based on `arg` property in our component state.

:::tip Note
Apart from `el`, you should treat these arguments as read-only and never modify them. If you need to share information across hooks, it is recommended to do so through element's [dataset](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset).
:::

## Function Shorthand {#function-shorthand}

It's common for a custom directive to have the same behavior for `mounted` and `updated`, with no need for the other hooks. In such cases we can define the directive as a function:

```vue-html
<div v-color="color"></div>
```

```js
app.directive('color', (el, binding) => {
  // this will be called for both `mounted` and `updated`
  el.style.color = binding.value
})
```

## Object Literals {#object-literals}

If your directive needs multiple values, you can also pass in a JavaScript object literal. Remember, directives can take any valid JavaScript expression.

```vue-html
<div v-demo="{ color: 'white', text: 'hello!' }"></div>
```

```js
app.directive('demo', (el, binding) => {
  console.log(binding.value.color) // => "white"
  console.log(binding.value.text) // => "hello!"
})
```

## Usage on Components {#usage-on-components}

When used on components, custom directives will always apply to a component's root node, similar to [Fallthrough Attributes](/guide/components/attrs.html).

```vue-html
<MyComponent v-demo="test" />
```

```vue-html
<!-- template of MyComponent -->

<div> <!-- v-demo directive will be applied here -->
  <span>My component content</span>
</div>
```

Note that components can potentially have more than one root node. When applied to a multi-root component, a directive will be ignored and a warning will be thrown. Unlike attributes, directives can't be passed to a different element with `v-bind="$attrs"`. In general, it is **not** recommended to use custom directives on components.
