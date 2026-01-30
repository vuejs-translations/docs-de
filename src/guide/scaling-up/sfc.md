# Einzelne Dateikomponenten {#single-file-components}

## Einführung {#introduction}

Vue Single-File Components (auch bekannt als `*.vue`-Dateien, abgekürzt als **SFC**) ist ein spezielles Dateiformat, mit dem wir die Vorlage, die Logik **und** das Styling einer Vue-Komponente in einer einzigen Datei kapseln können. Hier ist ein Beispiel für eine SFC:

<div class="options-api">

```vue
<script>
export default {
  data() {
    return {
      greeting: 'Hello World!'
    }
  }
}
</script>

<template>
  <p class="greeting">{{ greeting }}</p>
</template>

<style>
.greeting {
  color: red;
  font-weight: bold;
}
</style>
```

</div>

<div class="composition-api">

```vue
<script setup>
import { ref } from 'vue'
const greeting = ref('Hello World!')
</script>

<template>
  <p class="greeting">{{ greeting }}</p>
</template>

<style>
.greeting {
  color: red;
  font-weight: bold;
}
</style>
```

</div>

Wie wir sehen können, ist Vue SFC eine natürliche Erweiterung des klassischen Trios aus HTML, CSS und JavaScript. Die Blöcke `<template>`, `<script>` und `<style>` kapseln die Ansicht, Logik und das Styling einer Komponente und vereinen sie in derselben Datei. Die vollständige Syntax ist in der [SFC-Syntaxspezifikation](/api/sfc-spec) definiert.

## Warum SFC {#why-sfc}

SFCs erfordern zwar einen Build-Schritt, bieten dafür aber zahlreiche Vorteile:

- Autor modulare Komponenten unter Verwendung vertrauter HTML-, CSS- und JavaScript-Syntax
- [Kollokation von inhärent gekoppelten Anliegen](#what-about-separation-of-concerns)
- Vorkompilierte Vorlagen ohne Laufzeitkompilierungskosten
- [Komponentenbezogenes CSS](/api/sfc-css-features)
- [Ergonomischere Syntax bei der Arbeit mit der Composition API](/api/sfc-script-setup)
- Weitere Optimierungen zur Kompilierungszeit durch die gegenseitige Analyse von Vorlagen und Skripten
- [IDE-Unterstützung](/guide/scaling-up/tooling#ide-support) mit Autovervollständigung und Typprüfung für Template-Ausdrücke
- Sofort einsatzbereite Unterstützung für Hot-Module Replacement (HMR)

SFC ist ein charakteristisches Merkmal von Vue als Framework und wird für die Verwendung von Vue in den folgenden Szenarien empfohlen:

- Single-Page-Anwendungen (SPA)
- Statische Seitengenerierung (SSG)
- Jedes nicht triviale Frontend, bei dem ein Build-Schritt für eine bessere Entwicklungserfahrung (DX) gerechtfertigt ist.

Allerdings sind wir uns bewusst, dass es Szenarien gibt, in denen SFCs übertrieben wirken können. Aus diesem Grund kann Vue weiterhin ohne Build-Schritt über einfaches JavaScript verwendet werden. Wenn Sie lediglich weitgehend statisches HTML mit leichten Interaktionen verbessern möchten, können Sie sich auch [petite-vue](https://github.com/vuejs/petite-vue) ansehen, eine 6 kB große Teilmenge von Vue, die für progressive Verbesserungen optimiert ist.

## So funktioniert es {#how-it-works}

Vue SFC ist ein frameworkspezifisches Dateiformat und muss von [@vue/compiler-sfc](https://github.com/vuejs/core/tree/main/packages/compiler-sfc) in Standard-JavaScript und CSS vorkompiliert werden. Ein kompiliertes SFC ist ein Standard-JavaScript-Modul (ES) – das bedeutet, dass Sie mit einer geeigneten Build-Konfiguration ein SFC wie ein Modul importieren können:

```js
import MyComponent from './MyComponent.vue'

export default {
  components: {
    MyComponent
  }
}
```

`<style>`-Tags innerhalb von SFCs werden während der Entwicklung in der Regel als native `<style>`-Tags eingefügt, um Hot Updates zu unterstützen. Für die Produktion können sie extrahiert und in einer einzigen CSS-Datei zusammengeführt werden.

Sie können mit SFCs experimentieren und herausfinden, wie sie im [Vue SFC Playground](https://play.vuejs.org/) kompiliert werden.

In realen Projekten integrieren wir den SFC-Compiler üblicherweise in ein Build-Tool wie [Vite](https://vitejs.dev/) oder [Vue CLI](http://cli.vuejs.org/) (das auf [webpack](https://webpack.js.org/) basiert). Vue bietet offizielle Scaffolding-Tools, um Ihnen den Einstieg in SFCs zu erleichtern. Weitere Details finden Sie im Abschnitt [SFC-Tools](/guide/scaling-up/tooling).

## Wie steht es mit der Trennung der Belange? {#what-about-separation-of-concerns}

Manche Nutzer mit einem traditionellen Webentwicklungshintergrund könnten die Befürchtung haben, dass SFCs verschiedene Belange an einem Ort vermischen – etwas, das HTML/CSS/JS eigentlich trennen sollten!

To answer this question, it is important for us to agree that **separation of concerns is not equal to the separation of file types**. The ultimate goal of engineering principles is to improve the maintainability of codebases. Separation of concerns, when applied dogmatically as separation of file types, does not help us reach that goal in the context of increasingly complex frontend applications.

In modern UI development, we have found that instead of dividing the codebase into three huge layers that interweave with one another, it makes much more sense to divide them into loosely-coupled components and compose them. Inside a component, its template, logic, and styles are inherently coupled, and colocating them actually makes the component more cohesive and maintainable.

Note even if you don't like the idea of Single-File Components, you can still leverage its hot-reloading and pre-compilation features by separating your JavaScript and CSS into separate files using [Src Imports](/api/sfc-spec#src-imports).
