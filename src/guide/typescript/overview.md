---
outline: deep
---

# Vue mit TypeScript verwenden {#using-vue-with-typescript}

Ein Typsystem wie TypeScript kann viele häufige Fehler durch statische Analyse vor Build-Zeit erkennen. Dadurch verringert sich das Risiko von Laufzeitfehlern in der Produktion, und gleichzeitig können wir Code in großen Anwendungen sicherer und mit mehr Vertrauen refaktorisieren. TypeScript verbessert außerdem die Entwicklerfreundlichkeit durch typspezifische Autovervollständigung in IDEs und Code-Editoren.

Vue ist selbst in TypeScript geschrieben und bietet erstklassige Unterstützung für TypeScript. Alle offiziellen Vue-Pakete enthalten gebündelte Typdeklarationen, die sofort einsatzbereit sein sollten.

## Projekt - Setup {#project-setup}

[`create-vue`](https://github.com/vuejs/create-vue), das offizielle Projekt-Scaffolding-Tool, [Vite](https://vitejs.dev/)-powered, bietet die Möglichkeit ein Vue-Projekt mit Vite und TypeScript-Unterstützung zu erstellen.

### Übersicht {#overview}

Bei einem Vite-basierten Setup führen der Entwicklungsserver und der Bundler nur die Transpilation durch und übernehmen keine Typüberprüfung. Dies stellt sicher, dass der Vite-Entwicklungsserver auch bei der Verwendung von TypeScript extrem schnell bleibt.

- Während der Entwicklung empfehlen wir, auf eine gute [IDE-Konfiguration](#ide-support) zu setzen, um sofortiges Feedback bei Typfehlern zu erhalten.

- Wenn du SFCs verwendest, nutze das [`vue-tsc`](https://github.com/vuejs/language-tools/tree/master/packages/tsc) Tool für die Typüberprüfung und die Generierung von Typdeklarationen über die Kommandozeile. `vue-tsc` ist ein Wrapper um `tsc`, die eigene Kommandozeilen-Schnittstelle von TypeScript. Es funktioniert größtenteils genauso wie `tsc` unterstützt jedoch zusätzlich Vue SFCs neben TypeScript-Dateien. Du kannst `vue-tsc` im Watch-Modus parallel zum Vite-Entwicklungsserver ausführen oder ein Vite-Plugin wie [vite-plugin-checker](https://vite-plugin-checker.netlify.app/) verwenden, das die Überprüfungen in einem separaten Worker-Thread ausführt.

- Vue CLI bietet ebenfalls TypeScript-Unterstützung, wird jedoch nicht mehr empfohlen. Siehe die [Hinweis](#note-on-vue-cli-and-ts-loader).

### IDE-Unterstützung {#ide-support}

- [Visual Studio Code](https://code.visualstudio.com/) (VS Code) wird aufgrund seiner hervorragenden, sofort verfügbaren Unterstützung für TypeScript stark empfohlen.

  - [Vue - Official](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (früher Volar) ist die offizielle VS Code-Erweiterung, die TypeScript-Unterstützung innerhalb von Vue SFCs bietet, zusammen mit vielen anderen großartigen Funktionen.

    :::tip
    Die offizielle Vue-Erweiterung ersetzt [Vetur](https://marketplace.visualstudio.com/items?itemName=octref.vetur), unsere frühere offizielle VS Code-Erweiterung für Vue 2. Wenn du Vetur derzeit installiert hast, stelle sicher, dass du es in Vue 3-Projekten deaktivierst.
    :::

- [WebStorm](https://www.jetbrains.com/webstorm/) bietet ebenfalls Unterstützung für sowohl TypeScript als auch Vue. Auch andere JetBrains-IDEs unterstützen diese Technologien, entweder direkt oder über [ein kostenloses plugin](https://plugins.jetbrains.com/plugin/9442-vue-js). Ab der Version 02.2023 bieten WebStorm und das Vue-Plugin integrierte Unterstützung für den Vue Language Server. Du kannst den Vue-Dienst so einstellen, dass er die Volar-Integration für alle TypeScript-Versionen verwendet, unter Einstellungen > Sprachen & Frameworks > TypeScript > Vue. Standardmäßig wird Volar für TypeScript-Versionen 5.0 und höher verwendet.

### Konfigurieren von `tsconfig.json` {#configuring-tsconfig-json}

Projekte, die mit `create-vue` erstellt werden, enthalten eine vorab konfigurierte `tsconfig.json`. Die Basis-Konfiguration wird im [`@vue/tsconfig`](https://github.com/vuejs/tsconfig) Paket abstrahiert. Innerhalb des Projekts verwenden wir [Projekt-Referenzen](https://www.typescriptlang.org/docs/handbook/project-references.html), um die richtigen Typen für Code zu gewährleisten, der in verschiedenen Umgebungen läuft (z. B. sollte der Anwendungscode und der Testcode unterschiedliche globale Variablen haben).

Beim manuellen Konfigurieren von `tsconfig.json` gibt es einige bemerkenswerte Optionen:

- [`compilerOptions.isolatedModules`](https://www.typescriptlang.org/tsconfig#isolatedModules) ist auf `true` gesetzt, weil Vite [esbuild](https://esbuild.github.io/) zum Transpilieren von TypeScript verwendet und es bestimmte Einschränkungen bei der Transpilation von Einzeldateien gibt. [`compilerOptions.verbatimModuleSyntax`](https://www.typescriptlang.org/tsconfig#verbatimModuleSyntax) ist [ein superset von `isolatedModules`](https://github.com/microsoft/TypeScript/issues/53601) und eine ebenfalls gute Wahl – es wird auch im [`@vue/tsconfig`](https://github.com/vuejs/tsconfig) verwendet.

- Wenn du die Options-API verwendest, musst du [`compilerOptions.strict`](https://www.typescriptlang.org/tsconfig#strict) auf `true` setzen (oder zumindest [`compilerOptions.noImplicitThis`](https://www.typescriptlang.org/tsconfig#noImplicitThis), aktivieren, was Teil der `strict` Option ist), um die Typüberprüfung von `this` in den Komponentenoptionen zu nutzen. Andernfalls wird `this` als `any` behandelt.

- Wenn du Resolver-Aliasnamen in deinem Build-Tool konfiguriert hast, zum Beispiel der Standard-Alias `@/*` der in einem `create-vue` Projekt konfiguriert ist, musst du dies auch in TypeScript über [`compilerOptions.paths`](https://www.typescriptlang.org/tsconfig#paths) konfigurieren.

- Wenn du TSX mit Vue verwenden möchtest, setze [`compilerOptions.jsx`](https://www.typescriptlang.org/tsconfig#jsx) auf `"preserve"`, und [`compilerOptions.jsxImportSource`](https://www.typescriptlang.org/tsconfig#jsxImportSource) auf `"vue"`.

Siehe auch:

- [Official TypeScript compiler options docs](https://www.typescriptlang.org/docs/handbook/compiler-options.html)
- [esbuild TypeScript compilation caveats](https://esbuild.github.io/content-types/#typescript-caveats)

### Hinweis zu Vue CLI und `ts-loader` {#note-on-vue-cli-and-ts-loader}

In webpack-basierten Setups wie Vue CLI ist es üblich, die Typüberprüfung als Teil der Modul-Transformations-Pipeline durchzuführen, beispielsweise mit `ts-loader`. Dies ist jedoch keine saubere Lösung, da das Typsystem Wissen über das gesamte Modulnetzwerk benötigt, um Typüberprüfungen durchzuführen. Die Transformationsstufe eines einzelnen Moduls ist einfach nicht der richtige Ort für diese Aufgabe. Es führt zu den folgenden Problemen:

- `ts-loader` kann nur den nach der Transformation erzeugten Code typisieren. Dies stimmt nicht mit den Fehlern überein, die wir in IDEs oder von `vue-tsc`, sehen, die direkt auf den Quellcode zurückgeführt werden.

- Die Typüberprüfung kann langsam sein. Wenn sie im selben Thread/Prozess wie die Code-Transformationen durchgeführt wird, hat dies erhebliche Auswirkungen auf die Build-Geschwindigkeit der gesamten Anwendung

- Wir haben bereits eine Typüberprüfung, die in unserer IDE in einem separaten Prozess läuft, daher ist der Leistungsabfall bei der Entwicklererfahrung einfach nicht gerechtfertigt.

Wenn du derzeit Vue 3 + TypeScript über Vue CLI verwendest, empfehlen wir dringend, auf Vite umzusteigen. Wir arbeiten auch an CLI-Optionen, um eine Transpilations-Only-TS-Unterstützung zu ermöglichen, damit du zu `vue-tsc` für die Typüberprüfung wechseln kannst.

## Allgemeine Hinweise zur Nutzung {#general-usage-notes}

### `defineComponent()` {#definecomponent}

Um TypeScript zu ermöglichen, die Typen korrekt innerhalb der Komponentenoptionen abzuleiten, müssen wir die Komponenten mit [`defineComponent()`](/api/general#definecomponent) definieren:

```ts
import { defineComponent } from 'vue'

export default defineComponent({
  // Typableitung aktiviert
  props: {
    name: String,
    msg: { type: String, required: true }
  },
  data() {
    return {
      count: 1
    }
  },
  mounted() {
    this.name // type: string | undefined
    this.msg // type: string
    this.count // type: number
  }
})
```

`defineComponent()` unterstützt auch die Ableitung der Props, die an `setup()` übergeben werden, wenn die Composition API ohne `<script setup>` verwendet wird:

```ts
import { defineComponent } from 'vue'

export default defineComponent({
  // Typableitung aktiviert
  props: {
    message: String
  },
  setup(props) {
    props.message // type: string | undefined
  }
})
```

Siehe auch:

- [Note on webpack Treeshaking](/api/general#note-on-webpack-treeshaking)
- [type tests for `defineComponent`](https://github.com/vuejs/core/blob/main/packages-private/dts-test/defineComponent.test-d.tsx)

:::tip
`defineComponent()` ermöglicht auch die Typableitung für Komponenten, die in einfachem JavaScript definiert sind.
:::

### Verwendung in Single-File Components {#usage-in-single-file-components}

Um TypeScript in SFCs zu verwenden, füge das `lang="ts"`-Attribut zu den `<script>`-Tags hinzu. Wenn `lang="ts"` vorhanden ist, profitieren auch alle Template-Ausdrücke von strikterer Typüberprüfung.

```vue
<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  data() {
    return {
      count: 1
    }
  }
})
</script>

<template>
  <!-- Typüberprüfung und Autovervollständigung aktiviert -->
  {{ count.toFixed(2) }}
</template>
```

`lang="ts"` kann auch mit `<script setup>` verwendet werden:

```vue
<script setup lang="ts">
// TypeScript aktiviert
import { ref } from 'vue'

const count = ref(1)
</script>

<template>
  <!-- Typüberprüfung und Autovervollständigung aktiviert -->
  {{ count.toFixed(2) }}
</template>
```

### TypeScript in Templates {#typescript-in-templates}

Das `<template>` unterstützt auch TypeScript in Binding-Ausdrücken, wenn `<script lang="ts">` oder `<script setup lang="ts">` verwendet wird. Dies ist besonders nützlich, wenn in Template-Ausdrücken eine Typumwandlung erforderlich ist.

Hier ein konstruiertes Beispiel:

```vue
<script setup lang="ts">
let x: string | number = 1
</script>

<template>
  <!-- Fehler, weil x auch ein string sein könnte -->
  {{ x.toFixed(2) }}
</template>
```

Dies kann mit einer Inline-Typumwandlung umgangen werden:

```vue{6}
<script setup lang="ts">
let x: string | number = 1
</script>

<template>
  {{ (x as number).toFixed(2) }}
</template>
```

:::tip
Wenn du Vue CLI oder eine webpack-basierte Umgebung verwendest, erfordert TypeScript in Template-Ausdrücken `vue-loader@^16.8.0`.
:::

### Verwendung mit TSX {#usage-with-tsx}

Vue unterstützt auch das Erstellen von Komponenten mit JSX / TSX. Weitere Details findest du in der [Render Function & JSX](/guide/extras/render-function.html#jsx-tsx) Anleitung.

## Generic Components {#generic-components}

Generische Komponenten werden in zwei Fällen unterstützt:

- In SFCs: [`<script setup>` mit dem `generic` -Attribut](/api/sfc-script-setup.html#generics)
- Die Render-Funktion / JSX-Komponenten: Die Funktionssignatur von [`defineComponent()`](/api/general.html#function-signature)

## API-Spezifische Rezepte {#api-specific-recipes}

- [TS mit Composition API](./composition-api)
- [TS mit Options API](./options-api)
