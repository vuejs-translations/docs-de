---
outline: deep
---

# Serverseitiges Rendering (SSR) {#server-side-rendering-ssr}

## Überblick {#overview}

### Was ist SSR? {#what-is-ssr}

Vue.js ist ein Framework zur Entwicklung clientseitiger Anwendungen. Standardmäßig erzeugen und manipulieren Vue-Komponenten das DOM im Browser. Es ist jedoch auch möglich, dieselben Komponenten serverseitig in HTML-Strings zu rendern, diese direkt an den Browser zu senden und das statische Markup schließlich clientseitig in eine interaktive Anwendung umzuwandeln.

Eine serverseitig gerenderte Vue.js-App kann auch als „isomorph“ oder „universell“ betrachtet werden, da der Großteil des App-Codes sowohl auf dem Server **als auch** auf dem Client ausgeführt wird.

### Warum SSR? {#why-ssr}

Im Vergleich zu einer clientseitigen Single-Page-Anwendung (SPA) liegt der Vorteil von SSR vor allem in Folgendem:

- **Schnellere Ladezeit:** Dies macht sich besonders bei langsamen Internetverbindungen oder Geräten mit geringer Leistung bemerkbar. Serverseitig gerendertes Markup muss nicht warten, bis der gesamte JavaScript-Code heruntergeladen und ausgeführt wurde, sodass Ihre Nutzer die vollständig gerenderte Seite schneller sehen. Zudem erfolgt das Abrufen der Daten beim ersten Besuch serverseitig, wodurch in der Regel eine schnellere Verbindung zu Ihrer Datenbank besteht als beim Client. Dies führt im Allgemeinen zu verbesserten [Core Web Vitals](https://web.dev/vitals/)-Werten, einer besseren Nutzererfahrung und kann für Anwendungen, bei denen die Ladezeit direkt mit der Konversionsrate zusammenhängt, entscheidend sein.

- **Einheitliches Denkmodell**: Sie können für die Entwicklung Ihrer gesamten App dieselbe Sprache und dasselbe deklarative, komponentenorientierte Denkmodell verwenden, anstatt zwischen einem Backend-Templating-System und einem Frontend-Framework hin und her zu wechseln.

- **Bessere Suchmaschinenoptimierung**: Die Suchmaschinen-Crawler sehen direkt die vollständig gerenderte Seite.

  :::tip
  Aktuell können Google und Bing synchrone JavaScript-Anwendungen problemlos indexieren. Synchronität ist hierbei der entscheidende Punkt. Wenn Ihre Anwendung mit einem Ladeindikator beginnt und anschließend Inhalte per Ajax abruft, wartet der Crawler nicht auf den Abschluss des Vorgangs. Das bedeutet: Wenn Sie auf SEO-relevanten Seiten Inhalte asynchron laden, kann serverseitiges Rendering (SSR) erforderlich sein.
  :::

Bei der Verwendung von SSR müssen auch einige Kompromisse berücksichtigt werden:

- Entwicklungsbeschränkungen. Browserspezifischer Code kann nur innerhalb bestimmter Lebenszyklus-Hooks verwendet werden; einige externe Bibliotheken benötigen möglicherweise eine spezielle Behandlung, um in einer serverseitig gerenderten Anwendung ausgeführt werden zu können.

- Aufwändigere Einrichtung und Bereitstellungsanforderungen. Im Gegensatz zu einer vollständig statischen Single-Page-Anwendung (SPA), die auf jedem beliebigen statischen Dateiserver bereitgestellt werden kann, benötigt eine serverseitig gerenderte Anwendung eine Umgebung, in der ein Node.js-Server ausgeführt werden kann.

- Höhere serverseitige Last. Das Rendern einer vollständigen Anwendung in Node.js ist CPU-intensiver als das Ausliefern statischer Dateien. Wenn Sie also mit hohem Traffic rechnen, sollten Sie sich auf eine entsprechende Serverlast einstellen und Caching-Strategien sinnvoll einsetzen.

Bevor Sie serverseitiges Rendering (SSR) für Ihre App einsetzen, sollten Sie sich zunächst fragen, ob Sie es überhaupt benötigen. Das hängt hauptsächlich davon ab, wie wichtig die Ladezeit für Ihre App ist. Wenn Sie beispielsweise ein internes Dashboard entwickeln, bei dem ein paar hundert Millisekunden beim ersten Laden keine große Rolle spielen, wäre SSR übertrieben. Ist die Ladezeit jedoch absolut entscheidend, kann SSR Ihnen helfen, die bestmögliche Performance beim ersten Laden zu erzielen.

### SSR vs. SSG {#ssr-vs-ssg}

**Static Site Generation (SSG)**, auch als Vorab-Rendering bezeichnet, ist eine weitere beliebte Technik zur Erstellung schneller Websites. Wenn die für das Server-Rendering einer Seite erforderlichen Daten für jeden Nutzer identisch sind, können wir die Seite nicht bei jeder Anfrage neu rendern, sondern nur einmal im Voraus während des Erstellungsprozesses. Vorab gerenderte Seiten werden generiert und als statische HTML-Dateien bereitgestellt.

SSG bietet dieselben Leistungsmerkmale wie SSR-Anwendungen: Es sorgt für eine hervorragende Geschwindigkeit bei der Bereitstellung von Inhalten. Gleichzeitig ist es kostengünstiger und einfacher zu implementieren als SSR-Anwendungen, da die Ausgabe aus statischem HTML und Assets besteht. Das Schlüsselwort hierbei ist **statisch**: SSG kann nur auf Seiten angewendet werden, die statische Daten bereitstellen, d. h. Daten, die zum Zeitpunkt der Erstellung bekannt sind und sich zwischen den Anfragen nicht ändern können. Jedes Mal, wenn sich die Daten ändern, ist eine neue Bereitstellung erforderlich.

Wenn Sie sich nur mit SSR beschäftigen, um die SEO einer Handvoll Marketing-Seiten (z. B. `/`, `/about`, `/contact` usw.) zu verbessern, dann ist SSG wahrscheinlich die bessere Wahl als SSR. SSG eignet sich auch hervorragend für inhaltsorientierte Websites wie Dokumentationsseiten oder Blogs. Tatsächlich wird diese Website, die Sie gerade lesen, statisch mit [VitePress](https://vitepress.dev/) generiert, einem Vue-basierten Static-Site-Generator.

## Einführungsanleitung {#basic-tutorial}

### Eine App rendern {#rendering-an-app}

Schauen wir uns einmal das einfachste Beispiel für Vue-SSR in der Praxis an.

1. Erstelle ein neues Verzeichnis und wechsle mit `cd` dorthin.
2. Führe `npm init -y` aus.
3. Füge `„type“: „module“` in `package.json` ein, damit Node.js im [ES-Module-Modus](https://nodejs.org/api/esm.html#modules-ecmascript-modules) läuft.
4. Führen Sie `npm install vue` aus.
5. Erstellen Sie eine Datei `example.js`:

```js
// Das läuft unter Node.js auf dem Server.
import { createSSRApp } from 'vue'
// Die Server-Rendering-API von Vue ist unter `vue/server-renderer` verfügbar.
import { renderToString } from 'vue/server-renderer'

const app = createSSRApp({
  data: () => ({ count: 1 }),
  template: `<button @click="count++">{{ count }}</button>`
})

renderToString(app).then((html) => {
  console.log(html)
})
```

Führe dann Folgendes aus:

```sh
> node example.js
```

Es sollte Folgendes in die Befehlszeile ausgeben:

```
<button>1</button>
```

[`renderToString()`](/api/ssr#rendertostring) nimmt eine Vue-App-Instanz entgegen und gibt ein Promise zurück, das den gerenderten HTML-Code der App liefert. Es ist auch möglich, das Rendern mithilfe der [Node.js Stream API](https://nodejs.org/api/stream.html) oder der [Web Streams API](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) zu streamen. Ausführliche Informationen findest du in der [SSR-API-Referenz](/api/ssr).

Anschließend können wir den Vue-SSR-Code in einen Server-Request-Handler verschieben, der das Anwendungs-Markup mit dem vollständigen HTML-Code der Seite umschließt. Für die nächsten Schritte werden wir [`express`](https://expressjs.com/) verwenden:

- Führe `npm install express` aus
- Erstellen Sie die folgende Datei `server.js`:

```js
import express from 'express'
import { createSSRApp } from 'vue'
import { renderToString } from 'vue/server-renderer'

const server = express()

server.get('/', (req, res) => {
  const app = createSSRApp({
    data: () => ({ count: 1 }),
    template: `<button @click="count++">{{ count }}</button>`
  })

  renderToString(app).then((html) => {
    res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        <title>Vue SSR Example</title>
      </head>
      <body>
        <div id="app">${html}</div>
      </body>
    </html>
    `)
  })
})

server.listen(3000, () => {
  console.log('ready')
})
```

Führe abschließend `node server.js` aus und rufe `http://localhost:3000` auf. Die Seite sollte nun mit der Schaltfläche funktionieren.

[Probier es auf StackBlitz aus](https://stackblitz.com/fork/vue-ssr-example-basic?file=index.js)

### Flüssigkeitszufuhr für Patienten {#client-hydration}

Wenn du auf die Schaltfläche klickst, wirst du feststellen, dass sich die Zahl nicht ändert. Der HTML-Code ist auf dem Client vollständig statisch, da wir Vue nicht im Browser laden.

Um die clientseitige App interaktiv zu machen, muss Vue den Schritt der **Hydration** durchführen. Während der Hydration erstellt Vue dieselbe Vue-Anwendung, die auf dem Server ausgeführt wurde, ordnet jede Komponente den DOM-Knoten zu, die sie steuern soll, und fügt DOM-Ereignis-Listener hinzu.

Um eine App im Hydration-Modus zu mounten, müssen wir [`createSSRApp()`](/api/application#createssrapp) anstelle von `createApp()` verwenden:

```js{2}
// this runs in the browser.
import { createSSRApp } from 'vue'

const app = createSSRApp({
  // ...same app as on server
})

// Das Einbinden einer SSR-Anwendung auf dem Client setzt Folgendes voraus
// Das HTML wurde vorgerendert und wird funktionieren
// Hydratisierung statt der Einbindung neuer DOM-Knoten
app.mount('#app')
```

### Codestruktur {#code-structure}

Beachten Sie, dass wir dieselbe Anwendungsimplementierung wie auf dem Server wiederverwenden müssen. An diesem Punkt müssen wir uns Gedanken über die Codestruktur in einer SSR-Anwendung machen – wie können wir denselben Anwendungscode zwischen Server und Client teilen?

Hier demonstrieren wir die einfachste Konfiguration. Zunächst lagern wir die Logik zur App-Erstellung in eine separate Datei, `app.js`, aus:

```js
// app.js (shared between server and client)
import { createSSRApp } from 'vue'

export function createApp() {
  return createSSRApp({
    data: () => ({ count: 1 }),
    template: `<button @click="count++">{{ count }}</button>`
  })
}
```

Diese Datei und ihre Abhängigkeiten werden zwischen Server und Client geteilt – wir nennen sie **universellen Code**. Beim Schreiben von universellem Code gibt es einiges zu beachten, wie wir [unten](#writing-ssr-friendly-code) erläutern werden.

Unser Client-Eintrag importiert den universellen Code, erstellt die App und führt die Einbindung durch:

```js
// client.js
import { createApp } from './app.js'

createApp().mount('#app')
```

Und der Server verwendet im Request-Handler dieselbe Logik zur App-Erstellung:

```js{2,5}
// server.js (irrelevant code omitted)
import { createApp } from './app.js'

server.get('/', (req, res) => {
  const app = createApp()
  renderToString(app).then(html => {
    // ...
  })
})
```

Um die Client-Dateien im Browser zu laden, müssen wir außerdem Folgendes tun:

1. Um Client-Dateien bereitzustellen, fügen Sie in der Datei `server.js` die Zeile `server.use(express.static('.'))` ein.
2. Laden Sie den Client-Eintrag, indem Sie `<script type="module" src="/client.js"></script>` in die HTML-Shell einfügen.
3. Unterstützung für die Verwendung von Befehlen wie `import * from 'vue'` im Browser durch Hinzufügen einer [Import Map](https://github.com/WICG/import-maps) zum HTML-Shell.
[Probieren Sie das vollständige Beispiel auf StackBlitz aus](https://stackblitz.com/fork/vue-ssr-example?file=index.js). Der Button ist jetzt interaktiv!
## Lösungen auf höherer Ebene {#higher-level-solutions}

Der Schritt vom Beispiel hin zu einer produktionsreifen SSR-Anwendung umfasst noch weitaus mehr. Wir werden Folgendes tun müssen:

- Unterstützung von Vue-SFCs und anderen Anforderungen an den Build-Prozess. Tatsächlich müssen wir zwei Builds für dieselbe App koordinieren: einen für den Client und einen für den Server.

  :::tip
  Vue-Komponenten werden bei der Verwendung für SSR anders kompiliert – Vorlagen werden zu Zeichenfolgenverkettungen statt zu Virtual-DOM-Rendering-Funktionen kompiliert, um eine effizientere Rendering-Leistung zu erzielen.
  :::

- Im Server-Request-Handler wird der HTML-Code mit den korrekten Links zu den clientseitigen Assets und optimalen Ressourcenhinweisen gerendert. Möglicherweise müssen wir auch zwischen SSR- und SSG-Modus wechseln oder sogar beide in derselben App kombinieren.

- Routing, Datenabruf und Speicher für die Zustandsverwaltung einheitlich verwalten.

Eine vollständige Implementierung wäre recht komplex und hängt von der von Ihnen gewählten Build-Toolchain ab. Daher empfehlen wir Ihnen dringend, sich für eine übergeordnete, vorgefertigte Lösung zu entscheiden, die diese Komplexität für Sie abstrahiert. Im Folgenden stellen wir Ihnen einige empfohlene SSR-Lösungen aus dem Vue-Ökosystem vor.

### Nuxt {#nuxt}

[Nuxt](https://nuxt.com/) ist ein auf dem Vue-Ökosystem aufbauendes Framework der höheren Ebene, das eine optimierte Entwicklungsumgebung für die Erstellung universeller Vue-Anwendungen bietet. Und das Beste daran: Man kann es auch als Generator für statische Websites nutzen! Wir empfehlen Ihnen wärmstens, es einmal auszuprobieren.

### Quasar {#quasar}

[Quasar](https://quasar.dev) ist eine umfassende, auf Vue basierende Lösung, mit der Sie SPA-, SSR-, PWA-, Mobil- und Desktop-Anwendungen sowie Browser-Erweiterungen auf der Basis einer einzigen Codebasis entwickeln können. Sie übernimmt nicht nur die Build-Konfiguration, sondern bietet auch eine vollständige Sammlung von UI-Komponenten, die dem Material-Design-Standard entsprechen.

### Vite SSR {#vite-ssr}

Vite bietet integrierte [Unterstützung für serverseitiges Rendering mit Vue](https://vitejs.dev/guide/ssr.html), ist dabei jedoch bewusst auf einer niedrigen Abstraktionsebene angesiedelt. Wenn du direkt mit Vite arbeiten möchtest, schau dir [vite-plugin-ssr](https://vite-plugin-ssr.com/) an – ein Community-Plugin, das dir viele der komplexen Details abnimmt.

Ein Beispiel für ein Vue- und Vite-SSR-Projekt mit manueller Einrichtung findest du außerdem [hier](https://github.com/vitejs/vite-plugin-vue/tree/main/playground/ssr-vue); es kann als Grundlage für eigene Entwicklungen dienen. Beachte jedoch, dass dieses Vorgehen nur dann empfohlen wird, wenn du bereits Erfahrung mit SSR sowie Build-Tools hast und tatsächlich die volle Kontrolle über die übergeordnete Architektur behalten möchtest.

## Schreiben von SSR-freundlichem Code {#writing-ssr-friendly-code}

Unabhängig von Ihrem Build-Setup oder der Wahl eines übergeordneten Frameworks gibt es einige Prinzipien, die für alle Vue-SSR-Anwendungen gelten.

### Reaktivität auf dem Server {#reactivity-on-the-server}

Während des SSR wird jede Anfrage-URL einem gewünschten Zustand unserer Anwendung zugeordnet. Da keine Benutzerinteraktion und keine DOM-Aktualisierungen stattfinden, ist Reaktivität auf dem Server nicht erforderlich. Standardmäßig ist die Reaktivität während des SSR deaktiviert, um die Leistung zu optimieren.

### Komponenten-Lebenszyklus-Hooks {#component-lifecycle-hooks}

Da keine dynamischen Aktualisierungen stattfinden, werden Lifecycle-Hooks wie <span class="options-api">`mounted`</span><span class="composition-api">`onMounted`</span> oder <span class="options-api">`updated`</span><span class="composition-api">`onUpdated`</span> während des SSR **nicht** aufgerufen und nur auf dem Client ausgeführt.<span class="options-api"> Die einzigen Hooks, die während des SSR aufgerufen werden, sind `beforeCreate` und `created`.</span>

Sie sollten Code vermeiden, der Nebenwirkungen erzeugt, die in <span class="options-api">`beforeCreate` und `created`</span><span class="composition-api">`setup()` oder im Root-Scope von `<script setup>`</span> bereinigt werden müssen. Ein Beispiel für solche Nebenwirkungen ist das Einrichten von Timern mit `setInterval`. In rein clientseitigem Code können wir einen Timer einrichten und ihn anschließend in <span class="options-api">`beforeUnmount`</span><span class="composition-api">`onBeforeUnmount`</span> oder <span class="options-api">`unmounted`</span><span class="composition-api">`onUnmounted`</span> wieder abbauen. Da die Unmount-Hooks während des SSR jedoch niemals aufgerufen werden, bleiben die Timer für immer bestehen. Um dies zu vermeiden, verschieben Sie Ihren Code mit Nebenwirkungen stattdessen in <span class="options-api">`mounted`</span><span class="composition-api">`onMounted`</span>.

### Zugriff auf plattformspezifische APIs {#access-to-platform-specific-apis}

Universeller Code kann nicht davon ausgehen, dass plattformspezifische APIs verfügbar sind. Wenn Ihr Code also direkt browser-spezifische globale Variablen wie `window` oder `document` verwendet, führt dies bei der Ausführung in Node.js zu Fehlern – und umgekehrt.

Bei Aufgaben, die sowohl auf dem Server als auch auf dem Client ausgeführt werden, für die jedoch unterschiedliche Plattform-APIs verwendet werden, empfiehlt es sich, die plattformspezifischen Implementierungen in eine universelle API einzubinden oder Bibliotheken zu verwenden, die dies für Sie übernehmen. Sie können beispielsweise [`node-fetch`](https://github.com/node-fetch/node-fetch) nutzen, um sowohl auf dem Server als auch auf dem Client dieselbe Fetch-API zu verwenden.

Bei reinen Browser-APIs ist es üblich, innerhalb von rein clientseitigen Lebenszyklus-Hooks wie <span class="options-api">`mounted`</span><span class="composition-api">`onMounted`</span> verzögert auf diese zuzugreifen.

Beachten Sie, dass es schwierig sein kann, eine Bibliothek eines Drittanbieters in eine servergerenderte App zu integrieren, wenn diese nicht für den universellen Einsatz konzipiert wurde. Sie könnten sie zwar möglicherweise zum Laufen bringen, indem Sie einige der globalen Variablen simulieren, doch dies wäre eine Notlösung und könnte den Code zur Umgebungserkennung anderer Bibliotheken beeinträchtigen.

### Grenzüberschreitende Umweltverschmutzung {#cross-request-state-pollution}

Im Kapitel „Zustandsverwaltung“ haben wir ein [einfaches Muster zur Zustandsverwaltung unter Verwendung von Reactivity-APIs](state-management#simple-state-management-with-reactivity-api) vorgestellt. Im SSR-Kontext erfordert dieses Muster einige zusätzliche Anpassungen.

Das Muster deklariert gemeinsam genutzte Zustände im Stamm-Gültigkeitsbereich eines JavaScript-Moduls. Dadurch werden sie zu **Singletons** – das heißt, es gibt während des gesamten Lebenszyklus unserer Anwendung nur eine Instanz des reaktiven Objekts. Dies funktioniert in einer rein clientseitigen Vue-Anwendung wie erwartet, da die Module in unserer Anwendung bei jedem Aufruf einer Browser-Seite neu initialisiert werden.

Im SSR-Kontext werden die Anwendungsmodule jedoch in der Regel nur einmal auf dem Server initialisiert, nämlich beim Hochfahren des Servers. Dieselben Modulinstanzen werden über mehrere Serveranfragen hinweg wiederverwendet, ebenso wie unsere Singleton-Zustandsobjekte. Wenn wir den gemeinsam genutzten Singleton-Zustand mit datenspezifischen Informationen eines einzelnen Benutzers verändern, können diese versehentlich in eine Anfrage eines anderen Benutzers gelangen. Wir bezeichnen dies als **Cross-Request-State-Pollution**.

Technisch gesehen könnten wir alle JavaScript-Module bei jeder Anfrage neu initialisieren, genau wie es in Browsern der Fall ist. Die Initialisierung von JavaScript-Modulen kann jedoch ressourcenintensiv sein, sodass dies die Serverleistung erheblich beeinträchtigen würde.

Die empfohlene Lösung besteht darin, bei jeder Anfrage eine neue Instanz der gesamten Anwendung – einschließlich des Routers und der globalen Speicher – zu erstellen. Anstatt diese dann direkt in unsere Komponenten zu importieren, stellen wir den gemeinsamen Zustand mithilfe von [app-level provide](/guide/components/provide-inject#app-level-provide) bereit und injizieren ihn in die Komponenten, die ihn benötigen:

```js
// app.js (shared between server and client)
import { createSSRApp } from 'vue'
import { createStore } from './store.js'

// called on each request
export function createApp() {
  const app = createSSRApp(/* ... */)
  // create new instance of store per request
  const store = createStore(/* ... */)
  // provide store at the app level
  app.provide('store', store)
  // also expose store for hydration purposes
  return { app, store }
}
```

State-Management-Bibliotheken wie Pinia wurden unter Berücksichtigung dieser Aspekte entwickelt. Weitere Informationen finden Sie im [SSR-Leitfaden von Pinia](https://pinia.vuejs.org/ssr/).

### Flüssigkeitsungleichgewicht {#hydration-mismatch}

Wenn die DOM-Struktur des vorgerenderten HTML-Codes nicht mit der erwarteten Ausgabe der clientseitigen Anwendung übereinstimmt, tritt ein Fehler aufgrund einer Hydration-Diskrepanz auf. Eine Hydration-Diskrepanz wird meist durch folgende Ursachen hervorgerufen:

1. Die Vorlage enthält eine ungültige HTML-Verschachtelungsstruktur, und der gerenderte HTML-Code wurde durch das native HTML-Parsing-Verhalten des Browsers „korrigiert“. Ein häufiges Problem ist beispielsweise, dass [`<div>` nicht innerhalb von `<p>` platziert werden kann](https://stackoverflow.com/questions/8397852/why-cant-the-p-tag-contain-a-div-tag-inside-it):

   ```html
   <p><div>hi</div></p>
   ```

   Wenn wir dies in unserem serverseitig gerenderten HTML ausgeben, bricht der Browser die erste `<p>` ab, sobald er auf `<div>` stößt, und wandelt sie in die folgende DOM-Struktur um:

   ```html
   <p></p>
   <div>hi</div>
   <p></p>
   ```

2. Die beim Rendern verwendeten Daten enthalten zufällig generierte Werte. Da dieselbe Anwendung zweimal ausgeführt wird – einmal auf dem Server und einmal auf dem Client –, kann nicht garantiert werden, dass die Zufallswerte bei beiden Durchläufen identisch sind. Es gibt zwei Möglichkeiten, durch Zufallswerte verursachte Abweichungen zu vermeiden:

   1. Verwenden Sie `v-if` + `onMounted`, um den Teil, der von Zufallswerten abhängt, ausschließlich auf dem Client darzustellen. Möglicherweise verfügt Ihr Framework auch über integrierte Funktionen, die dies vereinfachen, beispielsweise die Komponente `<ClientOnly>` in VitePress.

   2. Verwenden Sie eine Zufallszahlengenerator-Bibliothek, die die Erzeugung von Zufallszahlen anhand von Startwerten unterstützt, und stellen Sie sicher, dass sowohl auf dem Server als auch auf dem Client derselbe Startwert verwendet wird (z. B. indem Sie den Startwert in den serialisierten Zustand einfügen und ihn auf dem Client abrufen).

3. Der Server und der Client befinden sich in unterschiedlichen Zeitzonen. Manchmal möchten wir einen Zeitstempel in die Ortszeit des Benutzers umrechnen. Allerdings stimmen die Zeitzone während der Serverausführung und die Zeitzone während der Clientausführung nicht immer überein, und wir können die Zeitzone des Benutzers während der Serverausführung möglicherweise nicht zuverlässig ermitteln. In solchen Fällen sollte die Umrechnung in die Ortszeit ebenfalls ausschließlich auf dem Client erfolgen.

When Vue encounters a hydration mismatch, it will attempt to automatically recover and adjust the pre-rendered DOM to match the client-side state. This will lead to some rendering performance loss due to incorrect nodes being discarded and new nodes being mounted, but in most cases, the app should continue to work as expected. That said, it is still best to eliminate hydration mismatches during development.

#### Suppressing Hydration Mismatches <sup class="vt-badge" data-text="3.5+" /> {#suppressing-hydration-mismatches}

In Vue 3.5+, it is possible to selectively suppress inevitable hydration mismatches by using the [`data-allow-mismatch`](/api/ssr#data-allow-mismatch) attribute.

### Custom Directives {#custom-directives}

Since most custom directives involve direct DOM manipulation, they are ignored during SSR. However, if you want to specify how a custom directive should be rendered (i.e. what attributes it should add to the rendered element), you can use the `getSSRProps` directive hook:

```js
const myDirective = {
  mounted(el, binding) {
    // client-side implementation:
    // directly update the DOM
    el.id = binding.value
  },
  getSSRProps(binding) {
    // server-side implementation:
    // return the props to be rendered.
    // getSSRProps only receives the directive binding.
    return {
      id: binding.value
    }
  }
}
```

### Teleports {#teleports}

Teleports require special handling during SSR. If the rendered app contains Teleports, the teleported content will not be part of the rendered string. An easier solution is to conditionally render the Teleport on mount.

If you do need to hydrate teleported content, they are exposed under the `teleports` property of the ssr context object:

```js
const ctx = {}
const html = await renderToString(app, ctx)

console.log(ctx.teleports) // { '#teleported': 'teleported content' }
```

You need to inject the teleport markup into the correct location in your final page HTML similar to how you need to inject the main app markup.

:::tip
Avoid targeting `body` when using Teleports and SSR together - usually, `<body>` will contain other server-rendered content which makes it impossible for Teleports to determine the correct starting location for hydration.

Instead, prefer a dedicated container, e.g. `<div id="teleported"></div>` which contains only teleported content.
:::
