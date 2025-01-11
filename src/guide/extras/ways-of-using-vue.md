# Möglichkeiten zur Verwendung von Vue {#ways-of-using-vue}

Wir glauben, dass es für das Web keine Einheitslösung gibt, die für alle passt. Aus diesem Grund ist Vue so konzipiert, dass es flexibel und schrittweise anpassbar ist. Abhängig von Ihrem Anwendungsfall kann Vue auf unterschiedliche Weise verwendet werden, um die optimale Balance zwischen Stack-Komplexität, Entwicklererfahrung und Endleistung zu finden.

## Eigenständiges Skript {#standalone-script}

Vue kann als eigenständige Skriptdatei verwendet werden – kein Build-Schritt erforderlich! Wenn Sie über ein Backend-Framework verfügen, das bereits den größten Teil des HTML-Codes rendert, oder Ihre Frontend-Logik nicht komplex genug ist, um einen Build-Schritt zu rechtfertigen, ist dies die einfachste Möglichkeit, Vue in Ihren Stack zu integrieren. In solchen Fällen können Sie sich Vue als einen deklarativeren Ersatz von jQuery vorstellen.

Vue bietet auch eine alternative Distribution namens [petite-vue](https://github.com/vuejs/petite-vue) an, die speziell für die schrittweise Verbesserung vorhandener HTML-Codes optimiert ist. Es verfügt über einen kleineren Funktionsumfang, ist aber extrem leichtgewichtig und verwendet eine Implementierung, die in Szenarios ohne Build-Schritte effizienter ist.

## Eingebettete Webkomponenten {#embedded-web-components}

Mit Vue können Sie [Standard-Webkomponenten erstellen](/guide/extras/web-components), die in jede HTML-Seite eingebettet werden können, unabhängig davon, wie sie gerendert werden. Mit dieser Option können Sie Vue völlig verbraucherunabhängig nutzen: Die resultierenden Webkomponenten können in Legacy-Anwendungen, statisches HTML oder sogar Anwendungen eingebettet werden, die mit anderen Frameworks erstellt wurden.

## Einseitige Bewerbung (SPA) {#single-page-application-spa}

Einige Anwendungen erfordern umfassende Interaktivität, tiefe Sitzungstiefe und nicht triviale Stateful-Logik im Frontend. Der beste Weg, solche Anwendungen zu erstellen, ist die Verwendung einer Architektur, bei der Vue nicht nur die gesamte Seite steuert, sondern auch Datenaktualisierungen und Navigation übernimmt, ohne dass die Seite neu geladen werden muss. Diese Art von Anwendung wird typischerweise als Single-Page Application (SPA) bezeichnet.

Vue bietet Kernbibliotheken und [umfassende Tool-Unterstützung](/guide/scaling-up/tooling) mit erstaunlicher Entwicklererfahrung für die Erstellung moderner SPAs, darunter:

- Clientseitiger Router
- Blitzschnelle Build-Toolkette
- IDE-Unterstützung
- Browser-Devtools
- TypeScript-Integrationen
- Testen von Dienstprogrammen

SPAs erfordern in der Regel, dass das Backend API-Endpunkte verfügbar macht. Sie können Vue jedoch auch mit Lösungen wie [Inertia.js](https://inertiajs.com) kombinieren, um die SPA-Vorteile zu nutzen und gleichzeitig ein serverzentriertes Entwicklungsmodell beizubehalten.

## Fullstack / SSR {#fullstack-ssr}

Reine clientseitige SPAs sind problematisch, wenn die App sensibel auf SEO und Time-to-Content reagiert. Dies liegt daran, dass der Browser eine weitgehend leere HTML-Seite erhält und warten muss, bis das JavaScript geladen ist, bevor er etwas rendert.

Vue bietet erstklassige APIs zum „Rendern“ einer Vue-App in HTML-Strings auf dem Server. Dadurch kann der Server bereits gerendertes HTML zurücksenden, sodass Endbenutzer den Inhalt sofort sehen können, während das JavaScript heruntergeladen wird. Vue „hydratisiert“ dann die Anwendung auf der Clientseite, um sie interaktiv zu machen. Dies wird als [Server-Side Rendering (SSR)](/guide/scaling-up/ssr) bezeichnet und verbessert Core Web Vital-Metriken wie [Largest Contentful Paint (LCP)](https://web.dev/lcp) erheblich /).

Es gibt übergeordnete Vue-basierte Frameworks, die auf diesem Paradigma aufbauen, wie etwa [Nuxt](https://v3.nuxtjs.org/), mit denen Sie eine Fullstack-Anwendung mit Vue und JavaScript entwickeln können.

## JAMStack / SSG {#jamstack-ssg}

Das serverseitige Rendering kann im Voraus durchgeführt werden, wenn die erforderlichen Daten statisch sind. Das bedeutet, dass wir eine gesamte Anwendung vorab in HTML rendern und als statische Dateien bereitstellen können. Dies verbessert die Leistung der Website und vereinfacht die Bereitstellung erheblich, da wir Seiten nicht mehr bei jeder Anfrage dynamisch rendern müssen. Vue kann solche Anwendungen weiterhin mit Feuchtigkeit versorgen, um dem Client umfassende Interaktivität zu bieten. Diese Technik wird allgemein als Static-Site Generation (SSG) bezeichnet, auch bekannt als [JAMStack](https://jamstack.org/what-is-jamstack/).

Es gibt zwei Arten von SSG: einseitig und mehrseitig. Bei beiden Varianten wird die Website vorab in statisches HTML gerendert. Der Unterschied besteht darin:

– Nach dem ersten Laden der Seite „hydratisiert“ ein Single-Page-SSG die Seite in ein SPA. Dies erfordert im Voraus mehr JS-Nutzlast und Hydratationskosten, aber nachfolgende Navigationen werden schneller sein, da der Seiteninhalt nur teilweise aktualisiert werden muss, anstatt die gesamte Seite neu zu laden.

- Eine mehrseitige SSG lädt bei jeder Navigation eine neue Seite. Der Vorteil ist, dass es nur minimales JS liefern kann – oder überhaupt kein JS, wenn die Seite keine Interaktion erfordert! Einige mehrseitige SSG-Frameworks wie [Astro](https://astro.build/) unterstützen auch „partielle Hydration“ – was Ihnen die Verwendung von Vue-Komponenten ermöglicht, um interaktive „Inseln“ innerhalb von statischem HTML zu erstellen.

Einseitige SSGs eignen sich besser, wenn Sie nicht triviale Interaktivität, große Sitzungslängen oder persistente Elemente/Zustände über Navigationen hinweg erwarten. Andernfalls wäre mehrseitiges SSG die bessere Wahl.

Das Vue-Team unterhält außerdem einen Generator für statische Websites namens [VitePress](https://vitepress.vuejs.org/), der diese Website unterstützt, die Sie gerade lesen! VitePress unterstützt beide Varianten von SSG. [Nuxt](https://v3.nuxtjs.org/) unterstützt auch SSG. Sie können sogar SSR und SSG für verschiedene Routen in derselben Nuxt-App kombinieren.

## Jenseits des Webs {#beyond-the-web}

Obwohl Vue in erster Linie für die Erstellung von Webanwendungen konzipiert ist, ist es keineswegs nur auf den Browser beschränkt. Du kannst:

- Erstellen Sie Desktop-Apps mit [Electron](https://www.electronjs.org/) oder [Tauri](https://tauri.studio/en/)
- Erstellen Sie mobile Apps mit [Ionic Vue](https://ionicframework.com/docs/vue/overview)
- Erstellen Sie mit [Quasar](https://quasar.dev/) Desktop- und mobile Apps aus derselben Codebasis.
- Verwenden Sie die [Custom Renderer API](/api/custom-renderer) von Vue, um benutzerdefinierte Renderer zu erstellen, die auf [WebGL](https://troisjs.github.io/) oder sogar [das Terminal](https://github.com) abzielen /ycmjason/vuminal)!
