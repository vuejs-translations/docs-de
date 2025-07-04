# Zusammensetzbar {#composables}

<script setup>
import { useMouse } from './mouse'
const { x, y } = useMouse()
</script>

:::tip
Dieser Abschnitt setzt grundlegende Kenntnisse der Composition API voraus. Wenn Sie Vue nur mit der Options-API gelernt haben, können Sie die API-Präferenz auf Composition-API setzen (mit dem Umschalter oben in der linken Seitenleiste) und die Kapitel [Reactivity Fundamentals](/guide/essentials/reactivity-fundamentals.html) und [Lifecycle Hooks](/guide/essentials/lifecycle.html) erneut lesen.
:::

## Was ist ein „Composable“? {#what-is-a-composable}

Im Kontext von Vue-Anwendungen ist eine „Composable“ eine Funktion, die die Composition API von Vue nutzt, um **zustandsbezogene Logik** zu kapseln und wiederzuverwenden.

Bei der Erstellung von Frontend-Anwendungen müssen wir oft Logik für allgemeine Aufgaben wiederverwenden. Zum Beispiel müssen wir vielleicht Datumsangaben an vielen Stellen formatieren, also extrahieren wir eine wiederverwendbare Funktion dafür. Diese Formatierungsfunktion kapselt **zustandslose Logik**: Sie nimmt eine Eingabe entgegen und gibt sofort die erwartete Ausgabe zurück. Es gibt viele Bibliotheken für die Wiederverwendung zustandsloser Logik - zum Beispiel [lodash](https://lodash.com/) und [date-fns](https://date-fns.org/), von denen Sie vielleicht schon gehört haben.

Im Gegensatz dazu geht es bei der zustandsabhängigen Logik um die Verwaltung von Zuständen, die sich im Laufe der Zeit ändern. Ein einfaches Beispiel wäre die Verfolgung der aktuellen Position der Maus auf einer Seite. In realen Szenarien kann es sich auch um eine komplexere Logik handeln, wie z. B. Berührungsgesten oder den Verbindungsstatus zu einer Datenbank.

## Beispiel für einen Maus-Tracker {#mouse-tracker-example}

Wenn wir die Mausverfolgungsfunktionalität mit Hilfe der Composition API direkt in einer Komponente implementieren würden, sähe es folgendermaßen aus:

```vue
<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const x = ref(0)
const y = ref(0)

function update(event) {
  x.value = event.pageX
  y.value = event.pageY
}

onMounted(() => window.addEventListener('mousemove', update))
onUnmounted(() => window.removeEventListener('mousemove', update))
</script>

<template>Mouse position is at: {{ x }}, {{ y }}</template>
```

Was aber, wenn wir dieselbe Logik in mehreren Komponenten wiederverwenden wollen? Wir können die Logik in eine externe Datei extrahieren, und zwar als zusammensetzbare Funktion:

```js
// mouse.js
import { ref, onMounted, onUnmounted } from 'vue'

// Konventionell beginnen die Namen zusammensetzbarer Funktionen mit „use“
export function useMouse() {
  // gekapselter und von der zusammensetzbaren Datenbank verwalteter Zustand
  const x = ref(0)
  const y = ref(0)

  // kann ein Composable seinen verwalteten Zustand im Laufe der Zeit aktualisieren
  function update(event) {
    x.value = event.pageX
    y.value = event.pageY
  }

  // kann eine zusammensetzbare Komponente auch in die eigene Komponente
  // lifecycle to setup and teardown side effects.
  onMounted(() => window.addEventListener('mousemove', update))
  onUnmounted(() => window.removeEventListener('mousemove', update))

  // den verwalteten Zustand als Rückgabewert offenlegen
  return { x, y }
}
```

Und so kann es in Komponenten verwendet werden:

```vue
<script setup>
import { useMouse } from './mouse.js'

const { x, y } = useMouse()
</script>

<template>Mouse position is at: {{ x }}, {{ y }}</template>
```

<div class="demo">
  Mouse position is at: {{ x }}, {{ y }}
</div>

[Versuchen Sie es auf dem Spielplatz](https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmltcG9ydCB7IHVzZU1vdXNlIH0gZnJvbSAnLi9tb3VzZS5qcydcblxuY29uc3QgeyB4LCB5IH0gPSB1c2VNb3VzZSgpXG48L3NjcmlwdD5cblxuPHRlbXBsYXRlPlxuICBNb3VzZSBwb3NpdGlvbiBpcyBhdDoge3sgeCB9fSwge3sgeSB9fVxuPC90ZW1wbGF0ZT4iLCJpbXBvcnQtbWFwLmpzb24iOiJ7XG4gIFwiaW1wb3J0c1wiOiB7XG4gICAgXCJ2dWVcIjogXCJodHRwczovL3NmYy52dWVqcy5vcmcvdnVlLnJ1bnRpbWUuZXNtLWJyb3dzZXIuanNcIlxuICB9XG59IiwibW91c2UuanMiOiJpbXBvcnQgeyByZWYsIG9uTW91bnRlZCwgb25Vbm1vdW50ZWQgfSBmcm9tICd2dWUnXG5cbmV4cG9ydCBmdW5jdGlvbiB1c2VNb3VzZSgpIHtcbiAgY29uc3QgeCA9IHJlZigwKVxuICBjb25zdCB5ID0gcmVmKDApXG5cbiAgZnVuY3Rpb24gdXBkYXRlKGV2ZW50KSB7XG4gICAgeC52YWx1ZSA9IGV2ZW50LnBhZ2VYXG4gICAgeS52YWx1ZSA9IGV2ZW50LnBhZ2VZXG4gIH1cblxuICBvbk1vdW50ZWQoKCkgPT4gd2luZG93LmFkZEV2ZW50TGlzdGVuZXIoJ21vdXNlbW92ZScsIHVwZGF0ZSkpXG4gIG9uVW5tb3VudGVkKCgpID0+IHdpbmRvdy5yZW1vdmVFdmVudExpc3RlbmVyKCdtb3VzZW1vdmUnLCB1cGRhdGUpKVxuXG4gIHJldHVybiB7IHgsIHkgfVxufSJ9)

Wie wir sehen können, bleibt die Kernlogik identisch - wir mussten sie nur in eine externe Funktion verschieben und den Status zurückgeben, der ausgesetzt werden sollte. Genau wie innerhalb einer Komponente können Sie die gesamte Palette der [Composition-API-Funktionen](/api/#composition-api) in Composables verwenden. Die gleiche `useMouse()` Funktionalität kann nun in jeder Komponente verwendet werden.

Das Tolle an Composables ist jedoch, dass man sie auch verschachteln kann: Eine Composable-Funktion kann eine oder mehrere andere Composable-Funktionen aufrufen. So können wir komplexe Logik aus kleinen, isolierten Einheiten zusammenstellen, ähnlich wie wir eine ganze Anwendung aus Komponenten zusammenstellen. Aus diesem Grund haben wir beschlossen, die Sammlung von APIs, die dieses Muster ermöglichen, Composition API zu nennen.

Zum Beispiel können wir die Logik des Hinzufügens und Entfernens eines DOM-Ereignis-Listeners in ein eigenes Composable extrahieren:

```js
// event.js
import { onMounted, onUnmounted } from 'vue'

export function useEventListener(target, event, callback) {
  // if you want, you can also make this
  // support selector strings as target
  onMounted(() => target.addEventListener(event, callback))
  onUnmounted(() => target.removeEventListener(event, callback))
}
```

Und nun kann unser `useMouse()` composable zu vereinfacht werden:

```js{3,9-12}
// mouse.js
import { ref } from 'vue'
import { useEventListener } from './event'

export function useMouse() {
  const x = ref(0)
  const y = ref(0)

  useEventListener(window, 'mousemove', (event) => {
    x.value = event.pageX
    y.value = event.pageY
  })

  return { x, y }
}
```

:::tip
Jede Komponenteninstanz, die `useMouse()` aufruft, erstellt ihre eigenen Kopien des `x`- und `y`-Zustands, damit sie sich nicht gegenseitig stören. Wenn Sie einen gemeinsamen Zustand zwischen Komponenten verwalten wollen, lesen Sie das Kapitel [State Management](/guide/scaling-up/state-management.html).
:::

## Beispiel für einen asynchronen Zustand {#async-state-example}

Das Composable `useMouse()` nimmt keine Argumente entgegen, also schauen wir uns ein anderes Beispiel an, das davon Gebrauch macht. Beim asynchronen Abrufen von Daten müssen wir oft verschiedene Zustände behandeln: Laden, Erfolg und Fehler:

```vue
<script setup>
import { ref } from 'vue'

const data = ref(null)
const error = ref(null)

fetch('...')
  .then((res) => res.json())
  .then((json) => (data.value = json))
  .catch((err) => (error.value = err))
</script>

<template>
  <div v-if="error">Oops! Error encountered: {{ error.message }}</div>
  <div v-else-if="data">
    Data loaded:
    <pre>{{ data }}</pre>
  </div>
  <div v-else>Loading...</div>
</template>
```

Es wäre mühsam, dieses Muster in jeder Komponente, die Daten abrufen muss, zu wiederholen. Extrahieren wir es in eine zusammensetzbare:

```js
// fetch.js
import { ref } from 'vue'

export function useFetch(url) {
  const data = ref(null)
  const error = ref(null)

  fetch(url)
    .then((res) => res.json())
    .then((json) => (data.value = json))
    .catch((err) => (error.value = err))

  return { data, error }
}
```

Jetzt können wir in unserer Komponente einfach tun:

```vue
<script setup>
import { useFetch } from './fetch.js'

const { data, error } = useFetch('...')
</script>
```

`useFetch()` nimmt eine statische URL-Zeichenfolge als Eingabe - es führt also den Abruf nur einmal durch und ist dann fertig. Was aber, wenn wir wollen, dass es immer dann erneut abgerufen wird, wenn sich die URL ändert? Das können wir erreichen, indem wir auch refs als Argument akzeptieren:

```js
// fetch.js
import { ref, isRef, unref, watchEffect } from 'vue'

export function useFetch(url) {
  const data = ref(null)
  const error = ref(null)

  function doFetch() {
    // reset state before fetching..
    data.value = null
    error.value = null
    // unref() unwraps potential refs
    fetch(unref(url))
      .then((res) => res.json())
      .then((json) => (data.value = json))
      .catch((err) => (error.value = err))
  }

  if (isRef(url)) {
    // setup reactive re-fetch if input URL is a ref
    watchEffect(doFetch)
  } else {
    // otherwise, just fetch once
    // and avoid the overhead of a watcher
    doFetch()
  }

  return { data, error }
}
```

Diese Version von `useFetch()` akzeptiert nun sowohl statische URL-Strings als auch Refs von URL-Strings. Wenn sie erkennt, dass die URL eine dynamische Referenz ist, indem sie [`isRef()`](/api/reactivity-utilities.html#isref), es wird ein reaktiver Effekt mit Hilfe von [`watchEffect()`](/api/reactivity-core.html#watcheffect). Der Effekt wird sofort ausgeführt und verfolgt auch die URL-Referenz als Abhängigkeit. Sobald sich die URL-Referenz ändert, werden die Daten zurückgesetzt und erneut abgerufen.

Hier ist [die aktualisierte Version von `useFetch()`](https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmltcG9ydCB7IHJlZiwgY29tcHV0ZWQgfSBmcm9tICd2dWUnXG5pbXBvcnQgeyB1c2VGZXRjaCB9IGZyb20gJy4vdXNlRmV0Y2guanMnXG5cbmNvbnN0IGJhc2VVcmwgPSAnaHR0cHM6Ly9qc29ucGxhY2Vob2xkZXIudHlwaWNvZGUuY29tL3RvZG9zLydcbmNvbnN0IGlkID0gcmVmKCcxJylcbmNvbnN0IHVybCA9IGNvbXB1dGVkKCgpID0+IGJhc2VVcmwgKyBpZC52YWx1ZSlcblxuY29uc3QgeyBkYXRhLCBlcnJvciwgcmV0cnkgfSA9IHVzZUZldGNoKHVybClcbjwvc2NyaXB0PlxuXG48dGVtcGxhdGU+XG4gIExvYWQgcG9zdCBpZDpcbiAgPGJ1dHRvbiB2LWZvcj1cImkgaW4gNVwiIEBjbGljaz1cImlkID0gaVwiPnt7IGkgfX08L2J1dHRvbj5cblxuXHQ8ZGl2IHYtaWY9XCJlcnJvclwiPlxuICAgIDxwPk9vcHMhIEVycm9yIGVuY291bnRlcmVkOiB7eyBlcnJvci5tZXNzYWdlIH19PC9wPlxuICAgIDxidXR0b24gQGNsaWNrPVwicmV0cnlcIj5SZXRyeTwvYnV0dG9uPlxuICA8L2Rpdj5cbiAgPGRpdiB2LWVsc2UtaWY9XCJkYXRhXCI+RGF0YSBsb2FkZWQ6IDxwcmU+e3sgZGF0YSB9fTwvcHJlPjwvZGl2PlxuICA8ZGl2IHYtZWxzZT5Mb2FkaW5nLi4uPC9kaXY+XG48L3RlbXBsYXRlPiIsImltcG9ydC1tYXAuanNvbiI6IntcbiAgXCJpbXBvcnRzXCI6IHtcbiAgICBcInZ1ZVwiOiBcImh0dHBzOi8vc2ZjLnZ1ZWpzLm9yZy92dWUucnVudGltZS5lc20tYnJvd3Nlci5qc1wiXG4gIH1cbn0iLCJ1c2VGZXRjaC5qcyI6ImltcG9ydCB7IHJlZiwgaXNSZWYsIHVucmVmLCB3YXRjaEVmZmVjdCB9IGZyb20gJ3Z1ZSdcblxuZXhwb3J0IGZ1bmN0aW9uIHVzZUZldGNoKHVybCkge1xuICBjb25zdCBkYXRhID0gcmVmKG51bGwpXG4gIGNvbnN0IGVycm9yID0gcmVmKG51bGwpXG5cbiAgYXN5bmMgZnVuY3Rpb24gZG9GZXRjaCgpIHtcbiAgICAvLyByZXNldCBzdGF0ZSBiZWZvcmUgZmV0Y2hpbmcuLlxuICAgIGRhdGEudmFsdWUgPSBudWxsXG4gICAgZXJyb3IudmFsdWUgPSBudWxsXG4gICAgXG4gICAgLy8gcmVzb2x2ZSB0aGUgdXJsIHZhbHVlIHN5bmNocm9ub3VzbHkgc28gaXQncyB0cmFja2VkIGFzIGFcbiAgICAvLyBkZXBlbmRlbmN5IGJ5IHdhdGNoRWZmZWN0KClcbiAgICBjb25zdCB1cmxWYWx1ZSA9IHVucmVmKHVybClcbiAgICBcbiAgICB0cnkge1xuICAgICAgLy8gYXJ0aWZpY2lhbCBkZWxheSAvIHJhbmRvbSBlcnJvclxuICBcdCAgYXdhaXQgdGltZW91dCgpXG4gIFx0ICAvLyB1bnJlZigpIHdpbGwgcmV0dXJuIHRoZSByZWYgdmFsdWUgaWYgaXQncyBhIHJlZlxuXHQgICAgLy8gb3RoZXJ3aXNlIHRoZSB2YWx1ZSB3aWxsIGJlIHJldHVybmVkIGFzLWlzXG4gICAgXHRjb25zdCByZXMgPSBhd2FpdCBmZXRjaCh1cmxWYWx1ZSlcblx0ICAgIGRhdGEudmFsdWUgPSBhd2FpdCByZXMuanNvbigpXG4gICAgfSBjYXRjaCAoZSkge1xuICAgICAgZXJyb3IudmFsdWUgPSBlXG4gICAgfVxuICB9XG5cbiAgaWYgKGlzUmVmKHVybCkpIHtcbiAgICAvLyBzZXR1cCByZWFjdGl2ZSByZS1mZXRjaCBpZiBpbnB1dCBVUkwgaXMgYSByZWZcbiAgICB3YXRjaEVmZmVjdChkb0ZldGNoKVxuICB9IGVsc2Uge1xuICAgIC8vIG90aGVyd2lzZSwganVzdCBmZXRjaCBvbmNlXG4gICAgZG9GZXRjaCgpXG4gIH1cblxuICByZXR1cm4geyBkYXRhLCBlcnJvciwgcmV0cnk6IGRvRmV0Y2ggfVxufVxuXG4vLyBhcnRpZmljaWFsIGRlbGF5XG5mdW5jdGlvbiB0aW1lb3V0KCkge1xuICByZXR1cm4gbmV3IFByb21pc2UoKHJlc29sdmUsIHJlamVjdCkgPT4ge1xuICAgIHNldFRpbWVvdXQoKCkgPT4ge1xuICAgICAgaWYgKE1hdGgucmFuZG9tKCkgPiAwLjMpIHtcbiAgICAgICAgcmVzb2x2ZSgpXG4gICAgICB9IGVsc2Uge1xuICAgICAgICByZWplY3QobmV3IEVycm9yKCdSYW5kb20gRXJyb3InKSlcbiAgICAgIH1cbiAgICB9LCAzMDApXG4gIH0pXG59In0=), with an artificial delay and randomized error for demo purposes.

## Konventionen und bewährte Praktiken {#conventions-and-best-practices}

### Namensgebung {#naming}

Es ist eine Konvention, zusammensetzbare Funktionen mit camelCase-Namen zu benennen, die mit „use“ beginnen.

### Eingabe-Argumente {#input-arguments}

Ein Composable kann ref-Argumente akzeptieren, auch wenn es nicht auf sie angewiesen ist, um reaktiv zu sein. Wenn Sie ein Composable schreiben, das von anderen Entwicklern verwendet werden kann, ist es eine gute Idee, den Fall zu behandeln, dass die Eingabeargumente refs statt Rohwerte sind. Die [`unref()`](/api/reactivity-utilities.html#unref) Utility-Funktion wird für diesen Zweck sehr nützlich sein:

```js
import { unref } from 'vue'

function useFeature(maybeRef) {
  // wenn maybeRef tatsächlich eine Referenz ist, wird ihr Wert zurückgegeben.
  // andernfalls wird maybeRef so zurückgegeben, wie es ist
  const value = unref(maybeRef)
}
```

Wenn Ihr Composable reaktive Effekte erzeugt, wenn die Eingabe ein ref ist, stellen Sie sicher, dass Sie den ref entweder explizit mit `watch()` beobachten oder `unref()` innerhalb eines `watchEffect()` aufrufen, damit er richtig verfolgt wird.

### Rückgabewerte {#return-values}

Sie haben wahrscheinlich bemerkt, dass wir in Composables ausschließlich `ref()` anstelle von `reactive()` verwendet haben. Die empfohlene Konvention ist, dass Composables immer ein einfaches, nicht reaktives Objekt zurückgeben, das mehrere Refs enthält. Dadurch kann es in Komponenten destrukturiert werden, während die Reaktivität erhalten bleibt:

```js
// x und y sind Referenzwerte
const { x, y } = useMouse()
```

Die Rückgabe eines reaktiven Objekts aus einem zusammensetzbaren Objekt führt dazu, dass solche Destrukturen die Verbindung zwischen Reaktivität und dem Zustand innerhalb des zusammensetzbaren Objekts verlieren, während die Referenzobjekte diese Verbindung beibehalten.

Wenn Sie es vorziehen, den von Composables zurückgegebenen Zustand als Objekteigenschaften zu verwenden, können Sie das zurückgegebene Objekt mit `reactive()` umhüllen, so dass die Refs ausgepackt werden. Zum Beispiel:

```js
const mouse = reactive(useMouse())
// mouse.x is linked to original ref
console.log(mouse.x)
```

```vue-html
Mouse position is at: {{ mouse.x }}, {{ mouse.y }}
```

### Nebenwirkungen {#side-effects}

Es ist in Ordnung, Seiteneffekte (z. B. das Hinzufügen von DOM-Ereignis-Listenern oder das Abrufen von Daten) in Composables durchzuführen, aber beachten Sie die folgenden Regeln:

- Wenn Sie an einer Anwendung arbeiten, die [Server-Side Rendering](/guide/scaling-up/ssr.html) (SSR) verwendet, stellen Sie sicher, dass Sie DOM-spezifische Seiteneffekte in Post-Mount-Lifecycle-Hooks ausführen, z. B. `onMounted()`. Diese Hooks werden nur im Browser aufgerufen, so dass Sie sicher sein können, dass der darin enthaltene Code Zugriff auf das DOM hat.

- Denken Sie daran, Seiteneffekte in `onUnmounted()` zu bereinigen. Wenn zum Beispiel ein Composable einen DOM-Ereignis-Listener einrichtet, sollte es diesen Listener in `onUnmounted()` entfernen, wie wir im Beispiel `useMouse()` gesehen haben. Es kann eine gute Idee sein, ein Composable zu verwenden, das dies automatisch für Sie tut, wie das Beispiel `useEventListener()`.

### Verwendungsbeschränkungen {#usage-restrictions}

Composables sollten nur **synchron** in `<script setup>` oder dem `setup()`-Hook aufgerufen werden. In einigen Fällen können Sie sie auch in Lebenszyklus-Hooks wie `onMounted()` aufrufen.

Dies sind die Kontexte, in denen Vue in der Lage ist, die aktuelle aktive Komponenteninstanz zu bestimmen. Der Zugriff auf eine aktive Komponenteninstanz ist notwendig, damit:

1. Lebenszyklus-Haken können für sie registriert werden.

2. Berechnete Eigenschaften und Beobachter können mit ihr verknüpft werden, so dass sie entsorgt werden können, wenn die Instanz ausgehängt wird, um Speicherlecks zu vermeiden.

:::tip
`<script setup>` ist die einzige Stelle, an der Sie Composables **nach** der Verwendung von `await` aufrufen können. Der Compiler stellt nach der asynchronen Operation automatisch den aktiven Instanzkontext für Sie wieder her.
:::

## Extrahieren von Composables für die Codeorganisation {#extracting-composables-for-code-organization}

Composables können nicht nur für die Wiederverwendung, sondern auch für die Codeorganisation extrahiert werden. Wenn die Komplexität Ihrer Komponenten zunimmt, kann es passieren, dass Sie Komponenten haben, die zu groß sind, um darin zu navigieren und zu denken. Mit der Composition API haben Sie die volle Flexibilität, Ihren Komponentencode nach logischen Gesichtspunkten in kleinere Funktionen aufzuteilen:

```vue
<script setup>
import { useFeatureA } from './featureA.js'
import { useFeatureB } from './featureB.js'
import { useFeatureC } from './featureC.js'

const { foo, bar } = useFeatureA()
const { baz } = useFeatureB(foo)
const { qux } = useFeatureC(baz)
</script>
```

In gewisser Weise können Sie sich diese extrahierten Composables als komponentenspezifische Dienste vorstellen, die miteinander kommunizieren können.

## Verwendung von Composables in der Options-API {#using-composables-in-options-api}

Wenn Sie die Options-API verwenden, müssen Composables innerhalb von `setup()` aufgerufen werden, und die zurückgegebenen Bindungen müssen von `setup()` zurückgegeben werden, damit sie für `this` und die Vorlage sichtbar sind:

```js
import { useMouse } from './mouse.js'
import { useFetch } from './fetch.js'

export default {
  setup() {
    const { x, y } = useMouse()
    const { data, error } = useFetch('...')
    return { x, y, data, error }
  },
  mounted() {
    // setup() exposed properties can be accessed on `this`
    console.log(this.x)
  }
  // ...other options
}
```

## Vergleiche mit anderen Techniken {#comparisons-with-other-techniques}

### vs. Mixins {#vs-mixins}

Benutzer, die von Vue 2 kommen, sind vielleicht mit der Option [mixins](/api/options-composition.html#mixins) vertraut, die uns ebenfalls erlaubt, Komponentenlogik in wiederverwendbare Einheiten zu extrahieren. Es gibt drei Hauptnachteile von Mixins:

1. **Unklare Quelle der Eigenschaften**: Bei der Verwendung vieler Mixins wird unklar, welche Instanzeigenschaft von welchem Mixin injiziert wird, was es schwierig macht, die Implementierung nachzuvollziehen und das Verhalten der Komponente zu verstehen. Dies ist auch der Grund, warum wir die Verwendung des refs + destructure-Musters für Composables empfehlen: Es macht die Eigenschaftsquelle in konsumierenden Komponenten klar.

2. **Namensraumkollisionen**: Mehrere Mixins von verschiedenen Autoren können potenziell dieselben Eigenschaftsschlüssel registrieren, was zu Namensraumkollisionen führt. Bei Composables können Sie die destrukturierten Variablen umbenennen, wenn es widersprüchliche Schlüssel von verschiedenen Composables gibt.

3. **Implizite Mixin-übergreifende Kommunikation**: Mehrere Mixins, die miteinander interagieren müssen, sind auf gemeinsame Eigenschaftsschlüssel angewiesen, wodurch sie implizit gekoppelt sind. Mit Composables können Werte, die von einem Composable zurückgegeben werden, als Argumente an ein anderes übergeben werden, genau wie normale Funktionen.

Aus den oben genannten Gründen empfehlen wir die Verwendung von Mixins in Vue 3 nicht mehr. Das Feature wird nur aus Gründen der Migration und Vertrautheit beibehalten.

### vs. Komponenten ohne Rendering {#vs-renderless-components}

Im Kapitel über Komponentenslots haben wir das Muster [Renderless Component](/guide/components/slots.html#renderless-components) besprochen, das auf skalierten Slots basiert. Wir haben sogar die gleiche Mausverfolgungsdemo mit Hilfe von Renderless-Komponenten implementiert.

Der Hauptvorteil von Composables gegenüber Komponenten ohne Rendering besteht darin, dass Composables keinen zusätzlichen Overhead durch Komponenteninstanzen verursachen. Wenn sie in einer gesamten Anwendung verwendet werden, kann die Menge der zusätzlichen Komponenteninstanzen, die durch das Renderless-Component-Pattern erzeugt werden, zu einem spürbaren Leistungs-Overhead werden.

Die Empfehlung lautet, Composables zu verwenden, wenn reine Logik wiederverwendet wird, und Komponenten zu verwenden, wenn sowohl Logik als auch visuelles Layout wiederverwendet werden.

### vs. React-Haken {#vs-react-hooks}

Wenn Sie Erfahrung mit React haben, werden Sie feststellen, dass dies den benutzerdefinierten React Hooks sehr ähnlich sieht. Die Composition API wurde zum Teil von React Hooks inspiriert, und Vue Composables sind in der Tat ähnlich wie React Hooks, was die logischen Kompositionsfähigkeiten angeht. Allerdings basieren Vue Composables auf dem feinkörnigen Reaktivitätssystem von Vue, das sich grundlegend vom Ausführungsmodell der React Hooks unterscheidet. Dies wird im Detail in der [Composition API FAQ](/guide/extras/composition-api-faq#comparison-with-react-hooks) diskutiert.

## Weitere Lektüre {#further-reading}

- [Reactivity In Depth](/guide/extras/reactivity-in-depth.html): für ein grundlegendes Verständnis der Funktionsweise von Vue's Reaktivitätssystem.
- [State Management](/guide/scaling-up/state-management.html): für Muster der Verwaltung von Zuständen, die von mehreren Komponenten gemeinsam genutzt werden.
- [Testing Composables](/guide/scaling-up/testing.html#testing-composables): Tipps zum Unit-Test von Composables.
- [VueUse](https://vueuse.org/): eine ständig wachsende Sammlung von Vue Composables. Der Quellcode ist auch eine großartige Lernressource.
