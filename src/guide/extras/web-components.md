# Vue- und Webkomponenten {#vue-and-web-components}

[Web Components](https://developer.mozilla.org/en-US/docs/Web/Web_Components) ist ein Überbegriff für eine Reihe webnativer APIs, die es Entwicklern ermöglichen, wiederverwendbare benutzerdefinierte Elemente zu erstellen.

Wir betrachten Vue und Web Components in erster Linie als komplementäre Technologien. Vue bietet hervorragende Unterstützung sowohl für die Verwendung als auch für die Erstellung benutzerdefinierter Elemente. Ganz gleich, ob Sie benutzerdefinierte Elemente in eine bestehende Vue-Anwendung integrieren oder Vue zum Erstellen und Verteilen benutzerdefinierter Elemente verwenden, Sie befinden sich in guter Gesellschaft.

## Verwenden benutzerdefinierter Elemente in Vue {#using-custom-elements-in-vue}

Vue [erzielt in den Custom Elements Everywhere-Tests perfekte 100 %](https://custom-elements-everywhere.com/libraries/vue/results/results.html). Die Verwendung benutzerdefinierter Elemente innerhalb einer Vue-Anwendung funktioniert weitgehend genauso wie die Verwendung nativer HTML-Elemente, wobei einige Dinge zu beachten sind:

### Komponentenauflösung überspringen {#skipping-component-resolution}

Standardmäßig versucht Vue, ein nicht-natives HTML-Tag als registrierte Vue-Komponente aufzulösen, bevor es wieder als benutzerdefiniertes Element gerendert wird. Dies führt dazu, dass Vue während der Entwicklung die Warnung „Komponente konnte nicht aufgelöst werden“ ausgibt. Um Vue darüber zu informieren, dass bestimmte Elemente als benutzerdefinierte Elemente behandelt werden sollen und die Komponentenauflösung übersprungen werden soll, können wir die Option [`compilerOptions.isCustomElement`](/api/application.html#app-config-compileroptions) angeben.

Wenn Sie Vue mit einem Build-Setup verwenden, sollte die Option über Build-Konfigurationen übergeben werden, da es sich um eine Option zur Kompilierungszeit handelt.

#### Beispiel für eine In-Browser-Konfiguration {#example-in-browser-config}

```js
// Funktioniert nur, wenn die In-Browser-Kompilierung verwendet wird.
// Wenn Sie Build-Tools verwenden, sehen Sie sich die Konfigurationsbeispiele unten an.
app.config.compilerOptions.isCustomElement = (tag) => tag.includes('-')
```

#### Beispiel einer Vite-Konfiguration {#example-vite-config}

```js
// vite.config.js
import vue from '@vitejs/plugin-vue'

export default {
  plugins: [
    vue({
      template: {
        compilerOptions: {
          // treat all tags with a dash as custom elements
          isCustomElement: (tag) => tag.includes('-')
        }
      }
    })
  ]
}
```

#### Beispiel für eine Vue-CLI-Konfiguration {#example-vue-cli-config}

```js
// vue.config.js
module.exports = {
  chainWebpack: config => {
    config.module
      .rule('vue')
      .use('vue-loader')
      .tap(options => ({
        ...options,
        compilerOptions: {
          // treat any tag that starts with ion- as custom elements
          isCustomElement: tag => tag.startsWith('ion-')
        }
      }))
  }
}
```

### Übergeben von DOM-Eigenschaften {#passing-dom-properties}

Da DOM-Attribute nur Zeichenfolgen sein können, müssen wir komplexe Daten als DOM-Eigenschaften an benutzerdefinierte Elemente übergeben. Beim Festlegen von Requisiten für ein benutzerdefiniertes Element prüft Vue 3 automatisch das Vorhandensein der DOM-Eigenschaft mithilfe des „in“-Operators und legt den Wert lieber als DOM-Eigenschaft fest, wenn der Schlüssel vorhanden ist. Das bedeutet, dass Sie in den meisten Fällen nicht darüber nachdenken müssen, wenn das benutzerdefinierte Element den [empfohlenen Best Practices](https://web.dev/custom-elements-best-practices/) folgt.

Es kann jedoch in seltenen Fällen vorkommen, dass die Daten als DOM-Eigenschaft übergeben werden müssen, das benutzerdefinierte Element die Eigenschaft jedoch nicht ordnungsgemäß definiert/widerspiegelt (was dazu führt, dass die „In“-Prüfung fehlschlägt). In diesem Fall können Sie mit dem Modifikator „.prop“ erzwingen, dass eine „v-bind“-Bindung als DOM-Eigenschaft festgelegt wird:

```vue-html
<my-element :user.prop="{ name: 'jack' }"></my-element>

<!-- shorthand equivalent -->
<my-element .user="{ name: 'jack' }"></my-element>
```

## Erstellen benutzerdefinierter Elemente mit Vue {#building-custom-elements-with-vue}

Der Hauptvorteil benutzerdefinierter Elemente besteht darin, dass sie mit jedem Framework oder sogar ohne Framework verwendet werden können. Dies macht sie ideal für die Verteilung von Komponenten, bei denen der Endverbraucher möglicherweise nicht denselben Frontend-Stack verwendet, oder wenn Sie die Endanwendung von den Implementierungsdetails der verwendeten Komponenten isolieren möchten.

### defineCustomElement {#definecustomelement}

Vue unterstützt die Erstellung benutzerdefinierter Elemente mit genau denselben Vue-Komponenten-APIs über [`defineCustomElement`](/api/general.html#definecustomelement) Verfahren. Die Methode akzeptiert das gleiche Argument wie [`defineComponent`](/api/general.html#definecomponent), sondern gibt stattdessen einen benutzerdefinierten Elementkonstruktor zurück, der erweitert wird `HTMLElement`:

```vue-html
<my-vue-element></my-vue-element>
```

```js
import { defineCustomElement } from 'vue'

const MyVueElement = defineCustomElement({
  // normal Vue component options here
  props: {},
  emits: {},
  template: `...`,

  // defineCustomElement only: CSS to be injected into shadow root
  styles: [`/* inlined css */`]
})

// Register the custom element.
// After registration, all `<my-vue-element>` tags
// on the page will be upgraded.
customElements.define('my-vue-element', MyVueElement)

// You can also programmatically instantiate the element:
// (can only be done after registration)
document.body.appendChild(
  new MyVueElement({
    // initial props (optional)
  })
)
```

#### Lebenszyklus {#lifecycle}

– Ein benutzerdefiniertes Vue-Element stellt eine interne Vue-Komponenteninstanz in seinem Schattenstamm bereit, wenn der [`connectedCallback`](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements#using_the_lifecycle_callbacks) des Elements ist zum ersten Mal angerufen.

- Wenn der „disconnectedCallback“ des Elements aufgerufen wird, prüft Vue, ob das Element nach einem Mikrotask-Tick vom Dokument getrennt ist.

  - Wenn sich das Element noch im Dokument befindet, wird es verschoben und die Komponenteninstanz bleibt erhalten;

  – Wenn das Element vom Dokument getrennt wird, handelt es sich um eine Entfernung und die Komponenteninstanz wird ausgehängt.

#### Props {#props}

– Alle mit der Option „props“ deklarierten Requisiten werden im benutzerdefinierten Element als Eigenschaften definiert. Vue übernimmt gegebenenfalls automatisch die Spiegelung zwischen Attributen/Eigenschaften.

  - Attribute werden immer auf die entsprechenden Eigenschaften übertragen.

  – Eigenschaften mit primitiven Werten („string“, „boolean“ oder „number“) werden als Attribute wiedergegeben.

- Vue wandelt auch Requisiten, die mit den Typen „Boolean“ oder „Number“ deklariert wurden, automatisch in den gewünschten Typ um, wenn sie als Attribute festgelegt werden (bei denen es sich immer um Zeichenfolgen handelt). Nehmen wir zum Beispiel die folgende Props-Deklaration:

  ```js
  props: {
    selected: Boolean,
    index: Number
  }
  ```

  Und die Verwendung benutzerdefinierter Elemente:

  ```vue-html
  <my-element selected index="1"></my-element>
  ```

  In der Komponente wird „selected“ in „true“ (boolescher Wert) und „index“ in „1“ (Zahl) umgewandelt.

#### Events {#events}

Über `this.$emit` oder Setup `emit` ausgegebene Ereignisse werden als native [CustomEvents](https://developer.mozilla.org/en-US/docs/Web/Events/Creating_and_triggering_events#adding_custom_data_%E2%80%) versendet. 93_customevent) für das benutzerdefinierte Element. Zusätzliche Ereignisargumente (Nutzlast) werden als Array im CustomEvent-Objekt als dessen `Detail`-Eigenschaft angezeigt.

#### Slots {#slots}

Innerhalb der Komponente können Slots wie gewohnt mit dem Element „<slot/>“ gerendert werden. Beim Konsumieren des resultierenden Elements akzeptiert es jedoch nur [native Slots-Syntax](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_templates_and_slots):

- [Scoped slots](/guide/components/slots.html#scoped-slots) are not supported.

- Verwenden Sie beim Übergeben benannter Slots das Attribut „slot“ anstelle der Anweisung `v-slot`:

  ```vue-html
  <my-element>
    <div slot="named">hello</div>
  </my-element>
  ```

#### Bereitstellen / Injizieren {#provide-inject}

Die [Provide/Inject API](/guide/components/provide-inject.html#provide-inject) und ihr [Composition API-Äquivalent](/api/composition-api-dependency-injection.html#provide) funktionieren auch zwischen Vue -definierte benutzerdefinierte Elemente. Beachten Sie jedoch, dass dies **nur zwischen benutzerdefinierten Elementen** funktioniert. Das heißt, ein von Vue definiertes benutzerdefiniertes Element kann keine Eigenschaften einfügen, die von einer Vue-Komponente bereitgestellt werden, die kein benutzerdefiniertes Element ist.

### SFC as Custom Element {#sfc-as-custom-element}

„defineCustomElement“ funktioniert auch mit Vue Single-File Components (SFCs). Mit der Standard-Tool-Einrichtung wird der „<style>“ in den SFCs jedoch während der Produktionserstellung weiterhin extrahiert und in einer einzigen CSS-Datei zusammengeführt. Bei der Verwendung eines SFC als benutzerdefiniertes Element ist es oft wünschenswert, die „<style>“-Tags stattdessen in das Schattenstammverzeichnis des benutzerdefinierten Elements einzufügen.

Die offiziellen SFC-Tools unterstützen den Import von SFCs im `benutzerdefinierten Elementmodus` (erfordert `@vitejs/plugin-vue@^1.4.0` oder `vue-loader@^16.5.0`). Ein im benutzerdefinierten Elementmodus geladenes SFC fügt seine „<style>“-Tags als CSS-Strings ein und stellt sie unter der `styles`-Option der Komponente bereit. Dies wird von `defineCustomElement` erfasst und bei der Instanziierung in die Schattenwurzel des Elements eingefügt.

Um diesen Modus zu aktivieren, beenden Sie einfach den Namen Ihrer Komponentendatei mit `.ce.vue`:

```js
import { defineCustomElement } from 'vue'
import Example from './Example.ce.vue'

console.log(Example.styles) // ["/* inlined css */"]

// convert into custom element constructor
const ExampleElement = defineCustomElement(Example)

// register
customElements.define('my-example', ExampleElement)
```

Wenn Sie anpassen möchten, welche Dateien im benutzerdefinierten Elementmodus importiert werden sollen (z. B. indem Sie _alle_ SFCs als benutzerdefinierte Elemente behandeln), können Sie die Option `customElement` an die jeweiligen Build-Plugins übergeben:

- [@vitejs/plugin-vue](https://github.com/vitejs/vite/tree/main/packages/plugin-vue#using-vue-sfcs-as-custom-elements)
- [vue-loader](https://github.com/vuejs/vue-loader/tree/next#v16-only-options)

### Tipps für eine Vue Custom Elements Library {#tips-for-a-vue-custom-elements-library}

Beim Erstellen benutzerdefinierter Elemente mit Vue stützen sich die Elemente auf die Laufzeit von Vue. Abhängig davon, wie viele Funktionen verwendet werden, fallen Kosten für die Grundgröße von ca. 16 KB an. Dies bedeutet, dass die Verwendung von Vue nicht ideal ist, wenn Sie ein einzelnes benutzerdefiniertes Element versenden. Möglicherweise möchten Sie Vanilla-JavaScript, [petite-vue](https://github.com/vuejs/petite-vue) oder Frameworks verwenden spezialisieren sich auf kleine Laufzeitgrößen. Die Basisgröße ist jedoch mehr als gerechtfertigt, wenn Sie eine Sammlung benutzerdefinierter Elemente mit komplexer Logik versenden, da Vue es ermöglicht, jede Komponente mit viel weniger Code zu erstellen. Je mehr Elemente Sie zusammen versenden, desto besser ist der Kompromiss.

Wenn die benutzerdefinierten Elemente in einer Anwendung verwendet werden, die auch Vue verwendet, können Sie Vue aus dem erstellten Bundle externalisieren, sodass die Elemente dieselbe Kopie von Vue aus der Hostanwendung verwenden.

Es wird empfohlen, die einzelnen Elementkonstruktoren zu exportieren, um Ihren Benutzern die Flexibilität zu geben, sie bei Bedarf zu importieren und mit den gewünschten Tag-Namen zu registrieren. Sie können auch eine Komfortfunktion exportieren, um alle Elemente automatisch zu registrieren. Hier ist ein Beispiel-Einstiegspunkt einer benutzerdefinierten Vue-Elementbibliothek:

```js
import { defineCustomElement } from 'vue'
import Foo from './MyFoo.ce.vue'
import Bar from './MyBar.ce.vue'

const MyFoo = defineCustomElement(Foo)
const MyBar = defineCustomElement(Bar)

// export individual elements
export { MyFoo, MyBar }

export function register() {
  customElements.define('my-foo', MyFoo)
  customElements.define('my-bar', MyBar)
}
```

Wenn Sie viele Komponenten haben, können Sie auch Build-Tool-Funktionen wie die von Vite nutzen [glob import](https://vitejs.dev/guide/features.html#glob-import) oder webpack's [`require.context`](https://webpack.js.org/guides/dependency-management/#requirecontext) um alle Komponenten aus einem Verzeichnis zu laden.

## Webkomponenten vs. Vue-Komponenten {#web-components-vs-vue-components}

Einige Entwickler glauben, dass Framework-proprietäre Komponentenmodelle vermieden werden sollten und dass die ausschließliche Verwendung von Custom Elements eine Anwendung „zukunftssicher“ macht. Hier versuchen wir zu erklären, warum wir glauben, dass dies eine zu vereinfachte Sicht auf das Problem ist.

Es gibt tatsächlich eine gewisse Funktionsüberschneidung zwischen benutzerdefinierten Elementen und Vue-Komponenten: Beide ermöglichen es uns, wiederverwendbare Komponenten mit Datenübergabe, Ereignisausgabe und Lebenszyklusverwaltung zu definieren. Webkomponenten-APIs sind jedoch relativ niedrig und einfach aufgebaut. Um eine tatsächliche Anwendung zu erstellen, benötigen wir einige zusätzliche Funktionen, die die Plattform nicht abdeckt:

- Ein deklaratives und effizientes Vorlagensystem;

- Ein reaktives Zustandsverwaltungssystem, das die Extraktion und Wiederverwendung komponentenübergreifender Logik erleichtert;

– Eine leistungsstarke Möglichkeit, die Komponenten auf dem Server zu rendern und auf dem Client zu hydrieren (SSR), was für SEO wichtig ist [Web Vitals-Metriken wie LCP](https://web.dev/vitals/). SSR für native benutzerdefinierte Elemente umfasst normalerweise die Simulation des DOM in Node.js und die anschließende Serialisierung des mutierten DOM, während Vue SSR nach Möglichkeit in String-Verkettung kompiliert, was viel effizienter ist.

Das Komponentenmodell von Vue ist unter Berücksichtigung dieser Anforderungen als kohärentes System konzipiert.

Mit einem kompetenten Engineering-Team könnten Sie wahrscheinlich das Äquivalent auf nativen Custom Elements aufbauen – das bedeutet aber auch, dass Sie den langfristigen Wartungsaufwand eines internen Frameworks auf sich nehmen, während Sie auf das Ökosystem und die Community-Vorteile verzichten ein ausgereiftes Framework wie Vue.

Es gibt auch Frameworks, die auf der Grundlage ihres Komponentenmodells benutzerdefinierte Elemente verwenden, aber alle müssen zwangsläufig ihre proprietären Lösungen für die oben aufgeführten Probleme einführen. Die Verwendung dieser Frameworks bedeutet, dass Sie sich mit deren technischen Entscheidungen zur Lösung dieser Probleme auseinandersetzen müssen – was Sie, ungeachtet aller Ankündigungen, nicht automatisch vor möglichen zukünftigen Abwanderungen schützt.

Es gibt auch einige Bereiche, in denen wir benutzerdefinierte Elemente als einschränkend empfinden:

- Eine sorgfältige Slot-Bewertung behindert die Komponentenzusammensetzung. Vue's [scoped slots](/guide/components/slots.html#scoped-slots) sind ein leistungsstarker Mechanismus für die Komponentenkomposition, der aufgrund der Eitelkeit nativer Slots nicht von benutzerdefinierten Elementen unterstützt werden kann. Eager-Slots bedeuten auch, dass die empfangende Komponente nicht steuern kann, wann oder ob ein Teil des Slot-Inhalts gerendert wird.

– Der heutige Versand benutzerdefinierter Elemente mit CSS im Schatten-DOM-Bereich erfordert die Einbettung des CSS in JavaScript, damit sie zur Laufzeit in Schattenwurzeln eingefügt werden können. Sie führen auch zu duplizierten Stilen im Markup in SSR-Szenarien. Es gibt [platform features](https://github.com/whatwg/html/pull/4898/) In diesem Bereich wird derzeit daran gearbeitet, sie werden jedoch derzeit noch nicht allgemein unterstützt und es müssen noch Bedenken bezüglich der Produktionsleistung/SSR geklärt werden. In der Zwischenzeit bieten Vue SFCs [CSS scoping mechanisms](/api/sfc-css-features.html) die das Extrahieren der Stile in einfache CSS-Dateien unterstützen.

Vue bleibt immer auf dem neuesten Stand der Webplattform-Standards und wir nutzen gerne alles, was die Plattform bietet, wenn es uns die Arbeit erleichtert. Unser Ziel ist es jedoch, Lösungen bereitzustellen, die gut funktionieren und auch heute noch funktionieren. Das bedeutet, dass wir neue Plattformfunktionen mit einer kritischen Einstellung integrieren müssen – und dazu müssen wir die Lücken schließen, in denen die Standards noch nicht erfüllt sind.
