---
outline: deep
---

# Vue mit TypeScript {#using-vue-with-typescript}

Ein Typsystem wie TypeScript kann viele häufige Fehler durch statische Analyse zur Buildzeit erkennen. Dies reduziert die Wahrscheinlichkeit von Laufzeitfehlern in der Produktion und ermöglicht es uns auch, Code in großen Anwendungen sicherer zu refaktorisieren. TypeScript verbessert auch die Entwicklerergonomie durch typbasierte Autovervollständigung in IDEs.

Vue selbst ist in TypeScript geschrieben und bietet erstklassige TypeScript-Unterstützung. Alle offiziellen Vue-Pakete werden mit gebündelten Typdeklarationen geliefert, die sofort funktionieren sollten.

## Projekt Einrichtung {#project-setup}

[`create-vue`](https://github.com/vuejs/create-vue), das offizielle Scaffolding Tool zum Erstellen von Projekten, bietet die Möglichkeit, ein mit [Vite](https://vitejs.dev/) betriebenes, TypeScript-fähiges Vue-Projekt zu erstellen.

### Überblick {#overview}

In einem Vite-basierten Setup sind der Entwicklungsserver und der Bundler nur für Transpilation zuständig und führen keine Typprüfung durch. Dadurch bleibt der Vite-Entwicklungsserver auch bei Verwendung von TypeScript extrem schnell.

- Während der Entwicklung empfehlen wir, sich auf eine gute [IDE](#ide-support) zu verlassen, um sofortiges Feedback zu Typfehlern zu erhalten.

- Wenn SFCs verwendet werden, verwenden Sie das [`vue-tsc`](https://github.com/johnsoncodehk/volar/tree/master/vue-language-tools/vue-tsc)-Dienstprogramm für die Typprüfung der Befehlszeile und die Generierung von Typdeklarationen. `vue-tsc` ist ein Wrapper um `tsc`, der eigenen Befehlszeilenschnittstelle von TypeScript. Es funktioniert weitgehend genauso wie `tsc`, außer dass es zusätzlich zu TypeScript-Dateien auch Vue SFCs unterstützt. Sie können `vue-tsc` im Beobachtungsmodus parallel zum Vite-Entwicklungsserver ausführen oder ein Vite-Plugin wie [vite-plugin-checker](https://vite-plugin-checker.netlify.app/) verwenden, das die Prüfungen in einem separaten Worker-Thread ausführt.

- Vue CLI bietet auch TypeScript-Unterstützung, wird jedoch nicht mehr empfohlen. [Siehe unten](#note-on-vue-cli-and-ts-loader).

### IDE-Unterstützung {#ide-support}

- [Visual Studio Code](https://code.visualstudio.com/) (VSCode) wird aufgrund seiner hervorragenden Out-of-the-Box-Unterstützung für TypeScript dringend empfohlen.

  - [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) ist die offizielle VSCode-Erweiterung, die TypeScript-Unterstützung innerhalb von Vue SFCs bietet, sowie viele andere großartige Funktionen.

    :::tip
    Volar ersetzt [Vetur](https://marketplace.visualstudio.com/items?itemName=octref.vetur), unsere vorherige offizielle VSCode-Erweiterung für Vue 2. Wenn Sie Vetur derzeit installiert haben, stellen Sie sicher, dass Sie es in Vue 3-Projekten deaktivieren.
    :::
    
  - [TypeScript Vue Plugin](https://marketplace.visualstudio.com/items?itemName=Vue.vscode-typescript-vue-plugin) ist ebenfalls erforderlich, um Typunterstützung für `*.vue`-Importe in TS-Dateien zu erhalten.

- [WebStorm](https://www.jetbrains.com/webstorm/) bietet auch Out-of-the-Box-Unterstützung für sowohl TypeScript als auch Vue. Andere JetBrains IDEs unterstützen sie ebenfalls, entweder Out-of-the-Box oder über [ein kostenloses Plugin](https://plugins.jetbrains.com/plugin/9442-vue-js).

### Konfigurieren von `tsconfig.json` {#configuring-tsconfig-json}

Projekte, die mit `create-vue` erstellt wurden, enthalten eine vorkonfigurierte `tsconfig.json`-Datei. Diese Basiskonfiguration ist im Paket [`@vue/tsconfig`](https://github.com/vuejs/tsconfig) abstrahiert. Innerhalb des Projekts verwenden wir [Projektverweise](https://www.typescriptlang.org/docs/handbook/project-references.html), um korrekte Typen für Code zu gewährleisten, der in verschiedenen Umgebungen ausgeführt wird (z.B. sollten App-Code und Test-Code unterschiedliche globale Variablen haben).

Wenn Sie `tsconfig.json` manuell konfigurieren, sind einige bemerkenswerte Optionen:

- [`compilerOptions.isolatedModules`](https://www.typescriptlang.org/tsconfig#isolatedModules) ist auf `true` gesetzt, da Vite [esbuild](https://esbuild.github.io/) für die Transpilierung von TypeScript verwendet und den Einschränkungen der Transpilierung einzelner Dateien unterliegt.

- Wenn Sie die Options API verwenden, müssen Sie [`compilerOptions.strict`](https://www.typescriptlang.org/tsconfig#strict) auf `true` setzen (oder zumindest [`compilerOptions.noImplicitThis`](https://www.typescriptlang.org/tsconfig#noImplicitThis) aktivieren, das Teil des `strict`-Flags ist), um die Typprüfung von `this` in Komponentenoptionen zu nutzen. Andernfalls wird `this` als `any` behandelt.

- Wenn Sie Resolver-Aliase in Ihrem Build-Tool konfiguriert haben, z.B. den `@/*`-Alias, der standardmäßig in einem `create-vue`-Projekt konfiguriert ist, müssen Sie ihn auch für TypeScript über [`compilerOptions.paths`](https://www.typescriptlang.org/tsconfig#paths) konfigurieren.

Siehe auch:

- [Official TypeScript compiler options docs](https://www.typescriptlang.org/docs/handbook/compiler-options.html)
- [esbuild TypeScript compilation caveats](https://esbuild.github.io/content-types/#typescript-caveats)

### Volar Übernahmemodus {#volar-takeover-mode}

> Dieser Abschnitt gilt nur für VSCode + Volar.

Um Vue SFCs und TypeScript zusammenarbeiten zu lassen, erstellt Volar eine separate TS-Sprachdienst-Instanz, die mit Vue-spezifischer Unterstützung gepatcht ist, und verwendet sie in Vue SFCs. Gleichzeitig werden einfache TS-Dateien weiterhin vom integrierten TS-Sprachdienst von VSCode behandelt, weshalb wir das [TypeScript Vue Plugin](https://marketplace.visualstudio.com/items?itemName=Vue.vscode-typescript-vue-plugin) benötigen, um Vue SFC-Importe in TS-Dateien zu unterstützen. Diese Standardkonfiguration funktioniert, aber für jedes Projekt führen wir zwei TS-Sprachdienst-Instanzen aus: eine von Volar, eine vom integrierten Dienst von VSCode. Dies ist etwas ineffizient und kann zu Leistungsproblemen in großen Projekten führen.

Volar bietet eine Funktion namens "Takeover Mode", um die Leistung zu verbessern. Im Takeover Mode bietet Volar Unterstützung für sowohl Vue- als auch TS-Dateien mithilfe einer einzigen TS-Sprachdienst-Instanz.

Um den Takeover Mode zu aktivieren, müssen Sie den integrierten TS-Sprachdienst von VSCode **nur in Ihrem Projekt-Workspace** deaktivieren, indem Sie diese Schritte befolgen:

1. Öffnen Sie die Befehlspalette. Drücken Sie `Ctrl + Shift + P` (macOS: `Cmd + Shift + P`).
2. Geben Sie `built` ein und wählen Sie "Extensions: Show Built-in Extensions".
3. Geben Sie `typescript` in das Suchfeld für Erweiterungen ein (entfernen Sie nicht das `@builtin`-Präfix).
4. Klicken Sie auf das kleine Zahnradsymbol von "TypeScript and JavaScript Language Features" und wählen Sie "Disable (Workspace)".
5. Laden Sie den Workspace neu. Der Takeover Mode wird aktiviert, wenn Sie eine Vue- oder TS-Datei öffnen.

<img src="./images/takeover-mode.png" width="590" height="426" style="margin:0px auto;border-radius:8px">

### Hinweis zu Vue CLI und `ts-loader` {#note-on-vue-cli-and-ts-loader}

In Webpack-basierten Setups wie Vue CLI ist es üblich, die Typprüfung als Teil der Modultransformationspipeline durchzuführen, beispielsweise mit `ts-loader`. Dies ist jedoch keine saubere Lösung, da das Typsystem Kenntnisse des gesamten Modulgraphen benötigt, um Typprüfungen durchzuführen. Der Transformationsschritt eines einzelnen Moduls ist einfach nicht der richtige Ort für diese Aufgabe. Dies führt zu folgenden Problemen:

- `ts-loader` kann nur Post-Transform-Code typenprüfen. Dies stimmt nicht mit den Fehlern überein, die wir in IDEs oder von `vue-tsc` sehen, die direkt auf den Quellcode zurückverweisen.

- Typprüfung kann langsam sein. Wenn sie im selben Thread / Prozess mit Codetransformationen durchgeführt wird, beeinträchtigt sie die Build-Geschwindigkeit der gesamten Anwendung erheblich.

- Wir haben bereits eine Typprüfung direkt in unserer IDE in einem separaten Prozess ausgeführt, sodass die Kosten für eine verlangsamte Entwicklungserfahrung einfach kein guter Kompromiss sind.

Wenn Sie derzeit Vue 3 + TypeScript über Vue CLI verwenden, empfehlen wir dringend, zu Vite zu migrieren. Wir arbeiten auch an CLI-Optionen, um nur die Transpilierung von TS zu aktivieren, sodass Sie zu `vue-tsc` für die Typprüfung wechseln können.

## Allgemeine Verwendungshinweise {#general-usage-notes}

### `defineComponent()` {#definecomponent}

Um TypeScript die korrekte Inferenz von Typen innerhalb von Komponentenoptionen zu ermöglichen, müssen wir Komponenten mit [`defineComponent()`](/api/general.html#definecomponent) definieren:

```ts
import { defineComponent } from 'vue'

export default defineComponent({
  // Typprüfung aktiviert
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

`defineComponent()` unterstützt auch die Inferenz der an `setup()` übergebenen Props bei Verwendung von Composition API ohne `<script setup>`:

```ts
import { defineComponent } from 'vue'

export default defineComponent({
  // Typprüfung aktiviert
  props: {
    message: String
  },
  setup(props) {
    props.message // type: string | undefined
  }
})
```

Siehe auch:

- [Note on webpack Treeshaking](/api/general.html#note-on-webpack-treeshaking)
- [type tests for `defineComponent`](https://github.com/vuejs/core/blob/main/test-dts/defineComponent.test-d.tsx)

:::tip
`defineComponent()` ermöglicht auch die Typinferenz für in plain JavaScript definierte Komponenten.
:::

### Verwendung in Single-File-Komponenten {#usage-in-single-file-components}

Um TypeScript in SFCs zu verwenden, fügen Sie das `lang="ts"`-Attribut zu `<script>`-Tags hinzu. Wenn `lang="ts"` vorhanden ist, genießen auch alle Template-Ausdrücke eine strengere Typprüfung.

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
  <!-- Typprüfung und Autovervollständigung aktiviert -->
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
  <!-- Typprüfung und Autovervollständigung aktiviert -->
  {{ count.toFixed(2) }}
</template>
```

### TypeScript in Templates {#typescript-in-templates}

Die `<template>`-Sektion unterstützt auch TypeScript in Bindungsausdrücken, wenn `<script lang="ts">` oder `<script setup lang="ts">` verwendet wird. Dies ist nützlich in Fällen, in denen Sie eine Typumwandlung in Template-Ausdrücken durchführen müssen.

Hier ist ein konstruiertes Beispiel:

```vue
<script setup lang="ts">
let x: string | number = 1
</script>

<template>
  <!-- Fehler, weil x ein String sein könnte. -->
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
Wenn Sie Vue CLI oder ein Webpack-basiertes Setup verwenden, erfordert TypeScript in Template-Ausdrücken `vue-loader@^16.8.0`.
:::

## API-spezifische Rezepte {#api-specific-recipes}

- [TS mit Composition API](./composition-api)
- [TS mit Options API](./options-api)
