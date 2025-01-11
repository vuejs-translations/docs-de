---
outline: deep
---

<script setup>
import SpreadSheet from './demos/SpreadSheet.vue'
</script>

# Reaktivität in der Tiefe {#reactivity-in-depth}

Eines der markantesten Merkmale von Vue ist das unaufdringliche Reaktivitätssystem. Der Zustand einer Komponente besteht aus reaktiven JavaScript-Objekten. Wenn Sie diese ändern, wird die Ansicht aktualisiert. Es macht die Zustandsverwaltung einfach und intuitiv, aber es ist auch wichtig zu verstehen, wie es funktioniert, um einige häufige Fehler zu vermeiden. In diesem Abschnitt werden wir uns mit einigen Details des Reaktivitätssystems von Vue beschäftigen.

## Was ist Reaktivität? {#what-is-reactivity}

Dieser Begriff taucht in der Programmierung heutzutage recht häufig auf, aber was ist damit gemeint? Reaktivität ist ein Programmierparadigma, mit dem wir uns auf deklarative Weise an Änderungen anpassen können. Das kanonische Beispiel, das die Leute gewöhnlich zeigen, weil es ein großartiges Beispiel ist, ist eine Excel-Tabelle:

<SpreadSheet />

In diesem Fall ist die Zelle A2 durch die Formel `= A0 + A1` definiert (Sie können auf A2 klicken, um die Formel anzuzeigen oder zu bearbeiten), so dass das Rechenblatt 3 anzeigt. Das ist keine Überraschung. Aber wenn Sie A0 oder A1 aktualisieren, werden Sie feststellen, dass A2 automatisch mit aktualisiert wird.

JavaScript funktioniert normalerweise nicht so. Wenn wir etwas Vergleichbares in JavaScript schreiben würden:

```js
let A0 = 1
let A1 = 2
let A2 = A0 + A1

console.log(A2) // 3

A0 = 2
console.log(A2) // Still 3
```

Wenn wir `A0` mutieren, ändert sich `A2` nicht automatisch.

Wie würden wir das in JavaScript machen? Um den Code zur Aktualisierung von „A2“ erneut auszuführen, müssen wir ihn zunächst in eine Funktion verpacken:

```js
let A2

function update() {
  A2 = A0 + A1
}
```

Dann müssen wir ein paar Begriffe definieren:

- Die Funktion `update()` erzeugt einen **Seiteneffekt**, oder kurz **Effekt**, weil sie den Zustand des Programms verändert.

- `A0` und `A1` werden als **Abhängigkeiten** des Effekts betrachtet, da ihre Werte zur Ausführung des Effekts verwendet werden. Der Effekt wird als **Abonnent** für seine Abhängigkeiten bezeichnet.

Was wir brauchen, ist eine magische Funktion, die `update()` (die **Auswirkung**) aufrufen kann, wenn sich `A0` oder `A1` (die **Abhängigkeiten**) ändern:

```js
whenDepsChange(update)
```

This `whenDepsChange()` Funktion hat folgende Aufgaben:

1. Verfolgen, wann eine Variable gelesen wird. Wenn z.B. der Ausdruck `A0 + A1` ausgewertet wird, werden sowohl `A0` als auch `A1` gelesen.

2. Wenn eine Variable gelesen wird, während ein Effekt gerade läuft, mache diesen Effekt zu einem Abonnenten dieser Variable. Da z.B. `A0` und `A1` gelesen werden, wenn `update()` ausgeführt wird, wird `update()` nach dem ersten Aufruf zu einem Abonnenten von `A0` und `A1`.

3. Erkennen, wenn eine Variable verändert wird. Wenn z.B. `A0` ein neuer Wert zugewiesen wird, werden alle seine Abonnenten benachrichtigt, damit sie erneut ausgeführt werden.

## Wie Reaktivität in Vue funktioniert {#how-reactivity-works-in-vue}

Wir können das Lesen und Schreiben von lokalen Variablen wie im Beispiel nicht wirklich verfolgen. Es gibt einfach keinen Mechanismus, um das in Vanilla JavaScript zu tun. Was wir jedoch **können**, ist das Abfangen des Lesens und Schreibens von **Objekteigenschaften**.

Es gibt zwei Möglichkeiten, den Zugriff auf Eigenschaften in JavaScript abzufangen: [getter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get) / [setters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/set) und [Proxies](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy). Vue 2 verwendet aufgrund von Einschränkungen der Browserunterstützung ausschließlich Getter / Setter. In Vue 3 werden Proxies für reaktive Objekte und Getter / Setter für Refs verwendet. Hier ist einige Pseudo-Code, der zeigt, wie sie funktionieren:

```js{4,9,17,22}
function reactive(obj) {
  return new Proxy(obj, {
    get(target, key) {
      track(target, key)
      return target[key]
    },
    set(target, key, value) {
      target[key] = value
      trigger(target, key)
    }
  })
}

function ref(value) {
  const refObject = {
    get value() {
      track(refObject, 'value')
      return value
    },
    set value(newValue) {
      value = newValue
      trigger(refObject, 'value')
    }
  }
  return refObject
}
```

:::tip
Die Codeschnipsel hier und weiter unten sollen die Kernkonzepte in möglichst einfacher Form erklären, daher werden viele Details weggelassen und Randfälle ignoriert.
:::

Dies erklärt einige [Einschränkungen von reaktiven Objekten](/guide/essentials/reactivity-fundamentals.html#limitations-of-reactive), die wir im Abschnitt über die Grundlagen diskutiert haben:

- Wenn Sie die Eigenschaft eines reaktiven Objekts einer lokalen Variablen zuweisen oder destrukturieren, wird die Reaktivität „abgekoppelt“, da der Zugriff auf die lokale Variable nicht mehr die Get/Set-Proxy-Fallen auslöst.

- Der von `reactive()` zurückgegebene Proxy verhält sich zwar genauso wie das Original, hat aber eine andere Identität, wenn wir ihn mit dem Operator `===` mit dem Original vergleichen.

Innerhalb von `track()` wird geprüft, ob es einen laufenden Effekt gibt. Wenn es einen gibt, suchen wir nach den Abonnenteneffekten (gespeichert in einem Set) für die Eigenschaft, die verfolgt wird, und fügen den Effekt dem Set hinzu:

```js
// Dies wird unmittelbar vor der Ausführung eines Effekts gesetzt
// ausgeführt werden soll. Wir werden uns später damit befassen.
let activeEffect

function track(target, key) {
  if (activeEffect) {
    const effects = getSubscribersForProperty(target, key)
    effects.add(activeEffect)
  }
}
```

Effekt-Abonnements werden in einer globalen `WeakMap<Ziel, Map<Schlüssel, Set<Effekt>>>` Datenstruktur gespeichert. Wenn für eine Eigenschaft (die zum ersten Mal verfolgt wird) kein Set mit abonnierten Effekten gefunden wurde, wird es erstellt. Das ist es, was die Funktion `getSubscribersForProperty()` in Kurzform tut. Der Einfachheit halber werden wir die Details überspringen.

Innerhalb von `trigger()` suchen wir wieder nach den Abonnenteneffekten für die Eigenschaft. Aber dieses Mal rufen wir sie stattdessen auf:

```js
function trigger(target, key) {
  const effects = getSubscribersForProperty(target, key)
  effects.forEach((effect) => effect())
}
```

Kehren wir nun zur Funktion `whenDepsChange()` zurück:

```js
function whenDepsChange(update) {
  const effect = () => {
    activeEffect = effect
    update()
    activeEffect = null
  }
  effect()
}
```

Sie verpackt die rohe `update`-Funktion in einen Effekt, der sich selbst als den aktuell aktiven Effekt setzt, bevor er die eigentliche Aktualisierung durchführt. Dies ermöglicht `track()`-Aufrufe während der Aktualisierung, um den aktuellen aktiven Effekt zu finden.

An dieser Stelle haben wir einen Effekt erstellt, der automatisch seine Abhängigkeiten verfolgt und bei jeder Änderung einer Abhängigkeit erneut ausgeführt wird. Wir nennen dies einen **Reaktiven Effekt**.

Vue bietet eine API, die es erlaubt, reaktive Effekte zu erstellen: [`watchEffect()`](/api/reactivity-core.html#watcheffect). In der Tat haben Sie vielleicht bemerkt, dass es ziemlich ähnlich wie das magische `whenDepsChange()` im Beispiel funktioniert. Wir können nun das ursprüngliche Beispiel mit aktuellen Vue-APIs überarbeiten:

```js
import { ref, watchEffect } from 'vue'

const A0 = ref(0)
const A1 = ref(1)
const A2 = ref()

watchEffect(() => {
  // Spuren A0 und A1
  A2.value = A0.value + A1.value
})

// löst die Wirkung aus
A0.value = 2
```

Die Verwendung eines reaktiven Effekts zum Ändern einer Referenz ist nicht der interessanteste Anwendungsfall - die Verwendung einer berechneten Eigenschaft macht ihn deklarativer:

```js
import { ref, computed } from 'vue'

const A0 = ref(0)
const A1 = ref(1)
const A2 = computed(() => A0.value + A1.value)

A0.value = 2
```

Intern verwaltet `computed` seine Ungültigkeitserklärung und Neuberechnung mit Hilfe eines reaktiven Effekts.

Was ist also ein Beispiel für einen üblichen und nützlichen reaktiven Effekt? Nun, das Aktualisieren des DOM! Wir können ein einfaches „reaktives Rendering“ wie folgt implementieren:

```js
import { ref, watchEffect } from 'vue'

const count = ref(0)

watchEffect(() => {
  document.body.innerHTML = `count is: ${count.value}`
})

// aktualisiert das DOM
count.value++
```

In der Tat kommt dies der Art und Weise, wie eine Vue-Komponente den Zustand und das DOM synchron hält, ziemlich nahe - jede Komponenteninstanz erzeugt einen reaktiven Effekt, um das DOM zu rendern und zu aktualisieren. Natürlich verwenden Vue-Komponenten viel effizientere Wege, um das DOM zu aktualisieren als `innerHTML`. Dies wird in [Rendering-Mechanismus](./rendering-mechanism) diskutiert.

<div class="options-api">

Die `ref()`, `computed()` und `watchEffect()` APIs sind alle Teil der Composition API. Wenn Sie bisher nur die Options-API mit Vue verwendet haben, werden Sie feststellen, dass die Composition-API näher daran ist, wie das Reaktivitätssystem von Vue unter der Haube arbeitet. Tatsächlich ist in Vue 3 die Options API auf der Composition API implementiert. Jeder Eigenschaftszugriff auf die Komponenteninstanz (`this`) löst Getter / Setter für die Reaktivitätsverfolgung aus und Optionen wie `watch` und `computed` rufen intern ihre Composition API Äquivalente auf.

</div>

## Laufzeit vs. Kompilierzeit Reaktivität {#runtime-vs-compile-time-reactivity}

Das Reaktivitätssystem von Vue ist in erster Linie laufzeitbasiert: Das Tracking und die Auslösung werden alle durchgeführt, während der Code direkt im Browser läuft. Die Vorteile der Laufzeit Reaktivität sind, dass es ohne einen Build-Schritt arbeiten kann, und es gibt weniger Randfälle. Andererseits ist sie dadurch durch die Syntaxbeschränkungen von JavaScript eingeschränkt.

Wir sind bereits im vorherigen Beispiel auf eine Einschränkung gestoßen: JavaScript bietet uns keine Möglichkeit, das Lesen und Schreiben lokaler Variablen abzufangen, so dass wir immer auf reaktive Zustände als Objekteigenschaften zugreifen müssen, entweder mit reaktiven Objekten oder mit refs.

Wir haben mit der Funktion [Reactivity Transform](/guide/extras/reactivity-transform.html) experimentiert, um den Umfang des Codes zu verringern:

```js
let A0 = $ref(0)
let A1 = $ref(1)

// Spur auf Variable lesen
const A2 = $computed(() => A0 + A1)

// Auslösung beim Schreiben von Variablen
A0 = 2
```

Dieses Snippet lässt sich genau so kompilieren, wie wir es ohne die Transformation geschrieben hätten, indem wir automatisch `.value` nach Verweisen auf die Variablen anhängen. Mit Reactivity Transform wird das Reaktivitätssystem von Vue zu einem hybriden System.

## Debugging der Reaktivität {#reactivity-debugging}

Es ist großartig, dass Vue's Reaktivitätssystem automatisch Abhängigkeiten verfolgt, aber in einigen Fällen möchten wir vielleicht herausfinden, was genau verfolgt wird oder was eine Komponente zum erneuten Rendern veranlasst.

### Debugging-Haken für Komponenten {#component-debugging-hooks}

We can debug what dependencies are used during a component's render and which dependency is triggering an update using the <span class=„options-api“>`renderTracked`</span><span class="composition- api„>`onRenderTracked`</span> and <span class=“options-api„>`renderTriggered`</span><span class=“composition-api">`onRenderTriggered`</span> lifecycle hooks. Both hooks will receive a debugger event which contains information on the dependency in question. It is recommended to place a `debugger` statement in the callbacks to interactively inspect the dependency:

<div class="composition-api">

```vue
<script setup>
import { onRenderTracked, onRenderTriggered } from 'vue'

onRenderTracked((event) => {
  debugger
})

onRenderTriggered((event) => {
  debugger
})
</script>
```

</div>
<div class="options-api">

```js
export default {
  renderTracked(event) {
    debugger
  },
  renderTriggered(event) {
    debugger
  }
}
```

</div>

:::tip
Komponentendebug-Hooks funktionieren nur im Entwicklungsmodus.
:::

Die Debug-Ereignisobjekte haben den folgenden Typ:

<span id="debugger-event"></span>

```ts
type DebuggerEvent = {
  effect: ReactiveEffect
  target: object
  type:
    | TrackOpTypes /* 'get' | 'has' | 'iterate' */
    | TriggerOpTypes /* 'set' | 'add' | 'delete' | 'clear' */
  key: any
  newValue?: any
  oldValue?: any
  oldTarget?: Map<any, any> | Set<any>
}
```

### Computergestütztes Debugging {#computed-debugging}

<!-- TODO options API equivalent -->

Wir können berechnete Eigenschaften debuggen, indem wir `computed()` ein zweites Optionsobjekt mit `onTrack` und `onTrigger` Callbacks übergeben:

- `onTrack` wird aufgerufen, wenn eine reaktive Eigenschaft oder eine Referenz als Abhängigkeit verfolgt wird.
- `onTrigger` wird aufgerufen, wenn der Watcher-Callback durch die Mutation einer Abhängigkeit ausgelöst wird.

Beide Callbacks empfangen Debugger-Ereignisse im [gleichen Format](#debugger-event) wie Komponenten-Debug-Hooks:

```js
const plusOne = computed(() => count.value + 1, {
  onTrack(e) {
    // triggered when count.value is tracked as a dependency
    debugger
  },
  onTrigger(e) {
    // triggered when count.value is mutated
    debugger
  }
})

// access plusOne, should trigger onTrack
console.log(plusOne.value)

// mutate count.value, should trigger onTrigger
count.value++
```

:::tip
`onTrack` und `onTrigger` Die berechneten Optionen funktionieren nur im Entwicklungsmodus.
:::

### Watcher-Debugging {#watcher-debugging}

<!-- TODO options API equivalent -->

Ähnlich wie bei `computed()` unterstützen Watcher auch die Optionen `onTrack` und `onTrigger`:

```js
watch(source, callback, {
  onTrack(e) {
    debugger
  },
  onTrigger(e) {
    debugger
  }
})

watchEffect(callback, {
  onTrack(e) {
    debugger
  },
  onTrigger(e) {
    debugger
  }
})
```

:::tip
`onTrack` und `onTrigger` Watcher-Optionen funktionieren nur im Entwicklungsmodus.
:::

## Integration mit externen staatlichen Systemen {#integration-with-external-state-systems}

Das Reaktivitätssystem von Vue funktioniert durch tiefe Konvertierung von einfachen JavaScript-Objekten in reaktive Proxies. Die tiefe Konvertierung kann unnötig oder manchmal unerwünscht sein, wenn man mit externen Zustandsverwaltungssystemen integriert (z.B. wenn eine externe Lösung auch Proxies verwendet).

Die allgemeine Idee der Integration von Vue's Reaktivitätssystem mit einer externen Zustandsverwaltungslösung ist es, den externen Zustand in einer [`shallowRef`](/api/reactivity-advanced.html#shallowref) zu halten. Ein shallow ref ist nur reaktiv, wenn auf seine Eigenschaft `.value` zugegriffen wird - der innere Wert bleibt intakt. Wenn sich der externe Zustand ändert, ersetzen Sie den ref-Wert, um Aktualisierungen auszulösen.

### Unveränderliche Daten {#immutable-data}

Wenn Sie eine Rückgängig-/Wiederherstellungsfunktion implementieren, möchten Sie wahrscheinlich bei jeder Benutzereingabe einen Schnappschuss des Anwendungsstatus erstellen. Allerdings ist das veränderbare Reaktivitätssystem von Vue dafür nicht am besten geeignet, wenn der Zustandsbaum groß ist, da die Serialisierung des gesamten Zustandsobjekts bei jeder Aktualisierung sowohl in Bezug auf die CPU- als auch auf die Speicherkosten teuer sein kann.

[Unveränderliche Datenstrukturen](https://en.wikipedia.org/wiki/Persistent_data_structure) lösen dieses Problem, indem sie die Zustandsobjekte niemals verändern - stattdessen werden neue Objekte erstellt, die die gleichen, unveränderten Teile mit den alten teilen. Es gibt verschiedene Möglichkeiten, unveränderliche Daten in JavaScript zu verwenden, aber wir empfehlen die Verwendung von [Immer](https://immerjs.github.io/immer/) mit Vue, weil es die Verwendung unveränderlicher Daten unter Beibehaltung der ergonomischeren, veränderbaren Syntax ermöglicht.

Wir können Immer mit Vue über ein einfaches Composable integrieren:

```js
import produce from 'immer'
import { shallowRef } from 'vue'

export function useImmer(baseState) {
  const state = shallowRef(baseState)
  const update = (updater) => {
    state.value = produce(state.value, updater)
  }

  return [state, update]
}
```

[Versuchen Sie es auf dem Spielplatz](https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmltcG9ydCB7IHVzZUltbWVyIH0gZnJvbSAnLi9pbW1lci5qcydcbiAgXG5jb25zdCBbaXRlbXMsIHVwZGF0ZUl0ZW1zXSA9IHVzZUltbWVyKFtcbiAge1xuICAgICB0aXRsZTogXCJMZWFybiBWdWVcIixcbiAgICAgZG9uZTogdHJ1ZVxuICB9LFxuICB7XG4gICAgIHRpdGxlOiBcIlVzZSBWdWUgd2l0aCBJbW1lclwiLFxuICAgICBkb25lOiBmYWxzZVxuICB9XG5dKVxuXG5mdW5jdGlvbiB0b2dnbGVJdGVtKGluZGV4KSB7XG4gIHVwZGF0ZUl0ZW1zKGl0ZW1zID0+IHtcbiAgICBpdGVtc1tpbmRleF0uZG9uZSA9ICFpdGVtc1tpbmRleF0uZG9uZVxuICB9KVxufVxuPC9zY3JpcHQ+XG5cbjx0ZW1wbGF0ZT5cbiAgPHVsPlxuICAgIDxsaSB2LWZvcj1cIih7IHRpdGxlLCBkb25lIH0sIGluZGV4KSBpbiBpdGVtc1wiXG4gICAgICAgIDpjbGFzcz1cInsgZG9uZSB9XCJcbiAgICAgICAgQGNsaWNrPVwidG9nZ2xlSXRlbShpbmRleClcIj5cbiAgICAgICAge3sgdGl0bGUgfX1cbiAgICA8L2xpPlxuICA8L3VsPlxuPC90ZW1wbGF0ZT5cblxuPHN0eWxlPlxuLmRvbmUge1xuICB0ZXh0LWRlY29yYXRpb246IGxpbmUtdGhyb3VnaDtcbn1cbjwvc3R5bGU+IiwiaW1wb3J0LW1hcC5qc29uIjoie1xuICBcImltcG9ydHNcIjoge1xuICAgIFwidnVlXCI6IFwiaHR0cHM6Ly9zZmMudnVlanMub3JnL3Z1ZS5ydW50aW1lLmVzbS1icm93c2VyLmpzXCIsXG4gICAgXCJpbW1lclwiOiBcImh0dHBzOi8vdW5wa2cuY29tL2ltbWVyQDkuMC43L2Rpc3QvaW1tZXIuZXNtLmpzP21vZHVsZVwiXG4gIH1cbn0iLCJpbW1lci5qcyI6ImltcG9ydCBwcm9kdWNlIGZyb20gJ2ltbWVyJ1xuaW1wb3J0IHsgc2hhbGxvd1JlZiB9IGZyb20gJ3Z1ZSdcblxuZXhwb3J0IGZ1bmN0aW9uIHVzZUltbWVyKGJhc2VTdGF0ZSkge1xuICBjb25zdCBzdGF0ZSA9IHNoYWxsb3dSZWYoYmFzZVN0YXRlKVxuICBjb25zdCB1cGRhdGUgPSAodXBkYXRlcikgPT4ge1xuICAgIHN0YXRlLnZhbHVlID0gcHJvZHVjZShzdGF0ZS52YWx1ZSwgdXBkYXRlcilcbiAgfVxuXG4gIHJldHVybiBbc3RhdGUsIHVwZGF0ZV1cbn0ifQ==)

### Zustandsmaschinen {#state-machines}

Der [Zustandsautomat](https://en.wikipedia.org/wiki/Finite-state_machine) ist ein Modell zur Beschreibung aller möglichen Zustände, in denen sich eine Anwendung befinden kann, und aller Möglichkeiten, wie sie von einem Zustand in einen anderen übergehen kann. Während es für einfache Komponenten überflüssig sein mag, kann es helfen, komplexe Zustandsabläufe robuster und handhabbarer zu machen.

Eine der beliebtesten State-Machine-Implementierungen in JavaScript ist [XState](https://xstate.js.org/). Hier ist ein Composable, das sich damit integrieren lässt:

```js
import { createMachine, interpret } from 'xstate'
import { shallowRef } from 'vue'

export function useMachine(options) {
  const machine = createMachine(options)
  const state = shallowRef(machine.initialState)
  const service = interpret(machine)
    .onTransition((newState) => (state.value = newState))
    .start()
  const send = (event) => service.send(event)

  return [state, send]
}
```

[Versuchen Sie es auf dem Spielplatz](https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmltcG9ydCB7IHVzZU1hY2hpbmUgfSBmcm9tICcuL21hY2hpbmUuanMnXG4gIFxuY29uc3QgW3N0YXRlLCBzZW5kXSA9IHVzZU1hY2hpbmUoe1xuICBpZDogJ3RvZ2dsZScsXG4gIGluaXRpYWw6ICdpbmFjdGl2ZScsXG4gIHN0YXRlczoge1xuICAgIGluYWN0aXZlOiB7IG9uOiB7IFRPR0dMRTogJ2FjdGl2ZScgfSB9LFxuICAgIGFjdGl2ZTogeyBvbjogeyBUT0dHTEU6ICdpbmFjdGl2ZScgfSB9XG4gIH1cbn0pXG48L3NjcmlwdD5cblxuPHRlbXBsYXRlPlxuICA8YnV0dG9uIEBjbGljaz1cInNlbmQoJ1RPR0dMRScpXCI+XG4gICAge3sgc3RhdGUubWF0Y2hlcyhcImluYWN0aXZlXCIpID8gXCJPZmZcIiA6IFwiT25cIiB9fVxuICA8L2J1dHRvbj5cbjwvdGVtcGxhdGU+IiwiaW1wb3J0LW1hcC5qc29uIjoie1xuICBcImltcG9ydHNcIjoge1xuICAgIFwidnVlXCI6IFwiaHR0cHM6Ly9zZmMudnVlanMub3JnL3Z1ZS5ydW50aW1lLmVzbS1icm93c2VyLmpzXCIsXG4gICAgXCJ4c3RhdGVcIjogXCJodHRwczovL3VucGtnLmNvbS94c3RhdGVANC4yNy4wL2VzL2luZGV4LmpzP21vZHVsZVwiXG4gIH1cbn0iLCJtYWNoaW5lLmpzIjoiaW1wb3J0IHsgY3JlYXRlTWFjaGluZSwgaW50ZXJwcmV0IH0gZnJvbSAneHN0YXRlJ1xuaW1wb3J0IHsgc2hhbGxvd1JlZiB9IGZyb20gJ3Z1ZSdcblxuZXhwb3J0IGZ1bmN0aW9uIHVzZU1hY2hpbmUob3B0aW9ucykge1xuICBjb25zdCBtYWNoaW5lID0gY3JlYXRlTWFjaGluZShvcHRpb25zKVxuICBjb25zdCBzdGF0ZSA9IHNoYWxsb3dSZWYobWFjaGluZS5pbml0aWFsU3RhdGUpXG4gIGNvbnN0IHNlcnZpY2UgPSBpbnRlcnByZXQobWFjaGluZSlcbiAgICAub25UcmFuc2l0aW9uKChuZXdTdGF0ZSkgPT4gKHN0YXRlLnZhbHVlID0gbmV3U3RhdGUpKVxuICAgIC5zdGFydCgpXG4gIGNvbnN0IHNlbmQgPSAoZXZlbnQpID0+IHNlcnZpY2Uuc2VuZChldmVudClcblxuICByZXR1cm4gW3N0YXRlLCBzZW5kXVxufSJ9)

### RxJS {#rxjs}

[RxJS](https://rxjs.dev/) ist eine Bibliothek für die Arbeit mit asynchronen Ereignisströmen. Die [VueUse](https://vueuse.org/) Bibliothek stellt das [`@vueuse/rxjs`](https://vueuse.org/rxjs/readme.html) Add-on zur Verfügung, um RxJS Streams mit dem Reaktivitätssystem von Vue zu verbinden.
