---
outline: deep
---

# Composition API FAQ {#composition-api-faq}

:::tip
Diese FAQ setzt Erfahrung mit Vue voraus - insbesondere Erfahrung mit Vue 2, wobei primär die Options API verwendet wird.
:::

## Was ist die Composition API? {#what-is-composition-api}

Die Composition API ist eine Gruppe von APIs, die es uns ermöglicht, Vue-Komponenten mit importierten Funktionen zu erstellen, anstatt Optionen zu deklarieren. Es ist ein Oberbegriff, der die folgenden APIs umfasst:

- [Reactivity API](/api/reactivity-core.html), z.B. `ref()` und `reactive()`, die es uns ermöglichen, reaktive Zustände, berechnete Zustände und Watcher direkt zu erzeugen.

- [Lifecycle Hooks](/api/composition-api-lifecycle.html), z.B. `onMounted()` und `onUnmounted()`, die es uns ermöglichen, programmatisch in den Lebenszyklus der Komponente einzugreifen.

- [Dependency Injection](/api/composition-api-dependency-injection.html), d.h. `provide()` und `inject()`, die es uns ermöglichen, das Dependency Injection System von Vue zu nutzen, während wir Reactivity APIs verwenden.

Composition API ist ein eingebautes Feature von Vue 3 und [Vue 2.7](https://blog.vuejs.org/posts/vue-2-7-naruto.html). Für ältere Vue 2 Versionen, verwenden Sie die offiziell gepflegte [`@vue/composition-api`](https://github.com/vuejs/composition-api) Plugin. In Vue 3 wird es auch hauptsächlich zusammen mit dem [`<script setup>`](/api/sfc-script-setup.html) Syntax in Einzeldateikomponenten. Hier ist ein einfaches Beispiel für eine Komponente, die die Composition API verwendet:

```vue
<script setup>
import { ref, onMounted } from 'vue'

// reaktiver Zustand
const count = ref(0)

// Funktionen, die den Zustand verändern und Aktualisierungen auslösen
function increment() {
  count.value++
}

// Lebenszyklus-Haken
onMounted(() => {
  console.log(`The initial count is ${count.value}.`)
})
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```

Trotz eines API-Stils, der auf Funktionskomposition basiert, ist **Composition API KEINE funktionale Programmierung**. Composition API basiert auf dem veränderbaren, feinkörnigen Reaktivitätsparadigma von Vue, während funktionale Programmierung die Unveränderlichkeit betont.

Wenn Sie daran interessiert sind, zu lernen, wie man Vue mit Composition API verwendet, können Sie die site-weite API-Präferenz auf Composition API setzen, indem Sie den Toggle oben in der linken Seitenleiste benutzen und dann die Anleitung von Anfang an durchgehen.

## Warum Kompositions-API? {#why-composition-api}

### Bessere Wiederverwendung von Logik {#better-logic-reuse}

Der Hauptvorteil der Composition API ist, dass sie eine saubere, effiziente Wiederverwendung von Logik in Form von [Composable functions](/guide/reusability/composables.html) ermöglicht. Sie löst [alle Nachteile von Mixins](/guide/reusability/composables.html#vs-mixins), den primären Mechanismus zur Wiederverwendung von Logik für die Options-API.

Die Fähigkeit der Composition API zur Wiederverwendung von Logik hat zu beeindruckenden Community-Projekten geführt, wie z.B. [VueUse](https://vueuse.org/), eine ständig wachsende Sammlung von Composition-Utilities. Es dient auch als sauberer Mechanismus für die einfache Integration von zustandsbehafteten Diensten oder Bibliotheken von Drittanbietern in das Reaktivitätssystem von Vue, zum Beispiel [immutable data](/guide/extras/reactivity-in-depth.html#immutable-data), [state machines](/guide/extras/reactivity-in-depth.html#state-machines) und [RxJS](https://vueuse.org/rxjs/readme.html#vueuse-rxjs).

### Flexiblere Code-Organisation {#more-flexible-code-organization}

Viele Benutzer lieben es, dass wir mit der Options-API standardmäßig organisierten Code schreiben: Alles hat seinen Platz, je nachdem, unter welche Option es fällt. Allerdings stößt die Options-API an ihre Grenzen, wenn die Logik einer einzelnen Komponente eine bestimmte Komplexitätsschwelle überschreitet. Diese Einschränkung ist besonders ausgeprägt bei Komponenten, die mit mehreren **logischen Belangen** umgehen müssen, was wir aus erster Hand in vielen produktiven Vue 2 Anwendungen erlebt haben.

Nehmen Sie die Ordner-Explorer-Komponente aus der GUI von Vue CLI als Beispiel: Diese Komponente ist für die folgenden logischen Belange zuständig:

- Verfolgung des aktuellen Ordnerstatus und Anzeige seines Inhalts
- Handhabung der Ordnernavigation (Öffnen, Schließen, Aktualisieren...)
- Handhabung der Erstellung neuer Ordner
- Umschalten auf die Anzeige nur der Lieblingsordner
- Anzeigen versteckter Ordner umschalten
- Behandlung von Änderungen des aktuellen Arbeitsverzeichnisses

Die [ursprüngliche Version](https://github.com/vuejs/vue-cli/blob/a09407dd5b9f18ace7501ddb603b95e31d6d93c0/packages/@vue/cli-ui/src/components/folder/FolderExplorer.vue#L198-L404) der Komponente wurde in Options API geschrieben. Wenn wir jeder Codezeile eine Farbe geben, die auf dem logischen Anliegen basiert, mit dem sie sich befasst, sieht es so aus:

<img alt="folder component before" src="./images/options-api.png" width="129" height="500" style="margin: 1.2em auto">

Beachten Sie, dass Code, der sich mit demselben logischen Problem befasst, unter verschiedenen Optionen aufgeteilt werden muss, die sich in verschiedenen Teilen der Datei befinden. In einer Komponente, die mehrere hundert Zeilen lang ist, erfordert das Verstehen und Navigieren in einem einzelnen logischen Bereich ein ständiges Auf- und Abwärtsblättern in der Datei, was die Sache viel schwieriger macht, als sie sein sollte. Wenn wir ein logisches Anliegen in ein wiederverwendbares Dienstprogramm extrahieren wollen, ist es außerdem ziemlich mühsam, die richtigen Codeteile aus verschiedenen Teilen der Datei zu finden und zu extrahieren.

Hier ist dieselbe Komponente vor und nach der [Umstrukturierung zur Composition API](https://gist.github.com/yyx990803/8854f8f6a97631576c14b63c8acd8f2e):

![Ordnerkomponente nach](./images/composition-api-after.png)

Beachten Sie, wie der Code, der sich auf dasselbe logische Anliegen bezieht, jetzt gruppiert werden kann: Wir müssen nicht mehr zwischen verschiedenen Optionsblöcken hin- und herspringen, während wir an einem bestimmten logischen Anliegen arbeiten. Außerdem können wir jetzt eine Gruppe von Code mit minimalem Aufwand in eine externe Datei verschieben, da wir den Code nicht mehr umherschieben müssen, um ihn zu extrahieren. Diese geringere Reibung beim Refactoring ist der Schlüssel für die langfristige Wartbarkeit großer Codebasen.

### Bessere Typeninferenz {#better-type-inference}

In den letzten Jahren haben immer mehr Frontend-Entwickler [TypeScript] (https://www.typescriptlang.org/) angenommen, da es uns hilft, robusteren Code zu schreiben, Änderungen mit mehr Vertrauen vorzunehmen und eine großartige Entwicklungserfahrung mit IDE-Unterstützung bietet. Die Options-API, die ursprünglich im Jahr 2013 entwickelt wurde, wurde jedoch ohne Typinferenz entworfen. Wir mussten einige [absurd komplexe Typentests](https://github.com/vuejs/core/blob/44b95276f5c086e1d88fa3c686a5f39eb5bb7821/packages/runtime-core/src/componentPublicInstance.ts#L132-L165) implementieren, damit die Typinferenz mit der Options-API funktioniert. Trotz all dieser Bemühungen kann die Typinferenz für die Options-API immer noch bei Mixins und Dependency Injection versagen.

Dies führte dazu, dass viele Entwickler, die Vue mit TS verwenden wollten, sich auf die Klassen-API von „Vue-Class-Component“ stützten. Eine klassenbasierte API verlässt sich jedoch stark auf ES-Dekoratoren, ein Sprachfeature, das nur ein Vorschlag der Stufe 2 war, als Vue 3 im Jahr 2019 entwickelt wurde. Wir hielten es für zu riskant, eine offizielle API auf einen instabilen Vorschlag zu stützen. Seitdem wurde der Vorschlag für Dekoratoren noch einmal komplett überarbeitet und erreichte schließlich 2022 die Stufe 3. Darüber hinaus leidet die klassenbasierte API unter logischer Wiederverwendung und organisatorischen Einschränkungen, ähnlich wie die Options-API.

Im Vergleich dazu verwendet die Composition API hauptsächlich einfache Variablen und Funktionen, die von Natur aus typfreundlich sind. In Composition API geschriebener Code kann eine vollständige Typinferenz genießen, ohne dass manuelle Typhinweise erforderlich sind. Die meiste Zeit sieht Composition-API-Code in TypeScript und einfachem JavaScript weitgehend identisch aus. Dadurch können auch Benutzer von einfachem JavaScript von der partiellen Typinferenz profitieren.

### Kleineres Produktionsbündel und weniger Gemeinkosten {#smaller-production-bundle-and-less-overhead}

Mit Composition API und `<script setup>` geschriebener Code ist auch effizienter und minification-freundlicher als das Äquivalent von Options API. Das liegt daran, dass die Vorlage in einer `<script setup>`-Komponente als eine Funktion kompiliert wird, die im selben Bereich wie der `<script setup>`-Code eingefügt ist. Im Gegensatz zum Eigenschaftszugriff von `this` kann der kompilierte Template-Code direkt auf Variablen zugreifen, die innerhalb von `<script setup>` deklariert sind, ohne einen Instanz-Proxy dazwischen. Dies führt auch zu einer besseren Minifizierung, da alle Variablennamen sicher gekürzt werden können.

## Beziehung zur Options-API {#relationship-with-options-api}

### Abwägungen {#trade-offs}

Einige Benutzer, die von der Options-API auf die Composition-API umgestiegen sind, fanden ihren Code weniger gut organisiert und kamen zu dem Schluss, dass die Composition-API in Bezug auf die Codeorganisation „schlechter“ ist. Wir empfehlen Nutzern mit einer solchen Meinung, dieses Problem aus einer anderen Perspektive zu betrachten.

Es stimmt, dass die Composition API nicht mehr die „Leitplanken“ bietet, die Sie anleiten, Ihren Code in die entsprechenden Buckets einzuordnen. Im Gegenzug können Sie den Komponentencode so verfassen, wie Sie auch normales JavaScript schreiben würden. Das bedeutet, **dass Sie auf Ihren Composition-API-Code alle bewährten Verfahren zur Codeorganisation anwenden können und sollten, wie Sie es beim Schreiben von normalem JavaScript tun würden**. Wenn Sie gut organisiertes JavaScript schreiben können, sollten Sie auch in der Lage sein, gut organisierten Composition-API-Code zu schreiben.

Die Options-API ermöglicht es Ihnen, beim Schreiben von Komponentencode „weniger nachzudenken“, weshalb viele Benutzer sie lieben. Durch die Verringerung des mentalen Aufwands werden Sie jedoch auch in das vorgeschriebene Code-Organisationsmuster ohne Ausweichmöglichkeit eingesperrt, was das Refactoring oder die Verbesserung der Codequalität in größeren Projekten erschweren kann. In dieser Hinsicht bietet die Composition API eine bessere langfristige Skalierbarkeit.

### Deckt die Composition API alle Anwendungsfälle ab? {#does-composition-api-cover-all-use-cases}

Ja, was die zustandsabhängige Logik betrifft. Bei der Verwendung der Kompositions-API gibt es nur ein paar Optionen, die möglicherweise noch benötigt werden: `props`, `emits`, `name`, und `inheritAttrs`. Wenn `<script setup>` verwendet wird, dann ist `inheritAttrs` normalerweise die einzige Option, die einen separaten normalen `<script>` Block erfordert.

Wenn Sie beabsichtigen, ausschließlich die Composition API (zusammen mit den oben genannten Optionen) zu verwenden, können Sie einige Kilobyte von Ihrem Produktions-Bundle durch ein [compile-time flag](https://github.com/vuejs/core/tree/main/packages/vue#bundler-build-feature-flags) einsparen, das den Options API bezogenen Code aus Vue entfernt. Beachten Sie, dass dies auch Vue-Komponenten in Ihren Abhängigkeiten betrifft.

### Kann ich beide APIs zusammen verwenden? {#can-i-use-both-apis-together}

Ja, Sie können die Composition API über die Option [`setup()`](/api/composition-api-setup.html) in einer Options-API-Komponente verwenden.

Wir empfehlen dies jedoch nur, wenn Sie eine bestehende Options-API-Codebasis haben, die mit neuen Funktionen / externen Bibliotheken, die mit der Composition API geschrieben wurden, integriert werden muss.

### Wird Options API veraltet sein? {#will-options-api-be-deprecated}

Nein, wir haben keine Pläne, dies zu tun. Options API ist ein integraler Bestandteil von Vue und der Grund, warum viele Entwickler es lieben. Wir sind uns auch bewusst, dass viele der Vorteile der Composition API nur in größeren Projekten zum Tragen kommen, und die Options API bleibt eine solide Wahl für viele Szenarien mit geringer bis mittlerer Komplexität.

## Beziehung zur Klassen-API {#relationship-with-class-api}

Wir empfehlen nicht mehr, Class API mit Vue 3 zu verwenden, da Composition API eine großartige TypeScript-Integration mit zusätzlichen Vorteilen bei der Wiederverwendung von Logik und Codeorganisation bietet.

## Vergleich mit React Hooks {#comparison-with-react-hooks}

Die Composition API bietet die gleichen logischen Kompositionsmöglichkeiten wie React Hooks, jedoch mit einigen wichtigen Unterschieden.

React Hooks werden jedes Mal, wenn eine Komponente aktualisiert wird, wiederholt aufgerufen. Dies führt zu einer Reihe von Vorbehalten, die selbst erfahrene React-Entwickler verwirren können. Es führt auch zu Problemen bei der Leistungsoptimierung, die die Entwicklungserfahrung stark beeinträchtigen können. Hier sind einige Beispiele:

- Hooks sind aufrufreihenfolgeabhängig und können nicht konditional sein.

- In einer React-Komponente deklarierte Variablen können von einer Hook-Closure erfasst werden und veralten, wenn der Entwickler es versäumt, das korrekte Abhängigkeits-Array zu übergeben. Dies führt dazu, dass React-Entwickler sich auf ESLint-Regeln verlassen, um sicherzustellen, dass korrekte Abhängigkeiten übergeben werden. Die Regeln sind jedoch oft nicht intelligent genug und überkompensieren die Korrektheit, was zu unnötigen Ungültigkeitserklärungen und Kopfschmerzen führt, wenn Edge Cases auftauchen.

- Aufwändige Berechnungen erfordern die Verwendung von `useMemo`, was wiederum die manuelle Übergabe des korrekten Abhängigkeits-Arrays erfordert.

- Ereignishandler, die an untergeordnete Komponenten übergeben werden, führen standardmäßig zu unnötigen Aktualisierungen der untergeordneten Komponenten und erfordern als Optimierung ein explizites `useCallback`. Dies ist fast immer erforderlich und erfordert wiederum ein korrektes Abhängigkeits-Array. Wird dies vernachlässigt, führt dies dazu, dass Anwendungen standardmäßig übermäßig gerendert werden, was zu Leistungsproblemen führen kann, ohne dass man es merkt.

- Das Stale-Closure-Problem in Verbindung mit Concurrent-Features macht es schwierig, Aussagen darüber zu treffen, wann ein Teil des Hooks-Codes ausgeführt wird, und macht die Arbeit mit veränderlichen Zuständen, die über mehrere Rendervorgänge hinweg bestehen bleiben sollten (über `useRef`), umständlich.

Im Vergleich dazu ist die Vue Composition API:

- Ruft `setup()` oder `<script setup>` Code nur einmal auf. Dadurch lässt sich der Code besser mit den Intuitionen der idiomatischen JavaScript-Nutzung in Einklang bringen, da es keine veralteten Abschlüsse gibt, über die man sich Gedanken machen muss. Kompositions-API-Aufrufe sind auch nicht empfindlich gegenüber der Aufrufreihenfolge und können an Bedingungen geknüpft sein.

- Vue's Laufzeit-Reaktivitätssystem sammelt automatisch reaktive Abhängigkeiten, die in berechneten Eigenschaften und Watchern verwendet werden, so dass es nicht notwendig ist, Abhängigkeiten manuell zu deklarieren.

- Es ist nicht notwendig, Callback-Funktionen manuell zwischenzuspeichern, um unnötige Aktualisierungen der untergeordneten Komponenten zu vermeiden. Im Allgemeinen stellt das feinkörnige Reaktivitätssystem von Vue sicher, dass Child-Komponenten nur dann aktualisiert werden, wenn sie es müssen. Manuelle Child-Update-Optimierungen sind für Vue-Entwickler selten ein Thema.

Wir erkennen die Kreativität von React Hooks an, und es ist eine wichtige Inspirationsquelle für Composition API. Allerdings gibt es die oben erwähnten Probleme in ihrem Design und wir haben festgestellt, dass Vue's Reaktivitätsmodell zufällig einen Weg bietet, diese zu umgehen.
