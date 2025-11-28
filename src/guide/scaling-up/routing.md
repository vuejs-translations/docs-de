# Routing {#routing}

## Clientseitiges vs. serverseitiges Routing {#client-side-vs-server-side-routing}

Routing auf der Serverseite bedeutet, dass der Server eine Antwort basierend auf dem URL-Pfad sendet, den der Benutzer besucht. Wenn wir in einer herkömmlichen, vom Server gerenderten Webanwendung auf einen Link klicken, erhält der Browser eine HTML-Antwort vom Server und lädt die gesamte Seite mit dem neuen HTML neu.

In einer [Single-Page-Anwendung](https://developer.mozilla.org/en-US/docs/Glossary/SPA) (SPA) kann das clientseitige JavaScript jedoch die Navigation abfangen, dynamisch neue Daten abrufen und die aktuelle Seite aktualisieren, ohne dass die gesamte Seite neu geladen werden muss. Dies führt in der Regel zu einer flüssigeren Benutzererfahrung, insbesondere bei Anwendungsfällen, die eher tatsächlichen „Anwendungen” ähneln, bei denen vom Benutzer über einen längeren Zeitraum hinweg viele Interaktionen erwartet werden.

In solchen SPAs erfolgt das „Routing“ auf der Client-Seite, im Browser. Ein clientseitiger Router ist für die Verwaltung der gerenderten Ansicht der Anwendung mithilfe von Browser-APIs wie der [History API](https://developer.mozilla.org/en-US/docs/Web/API/History) oder dem [`hashchange`-Ereignis](https://developer.mozilla.org/en-US/docs/Web/API/Window/hashchange_event) verantwortlich.

## Offizieller Router {#official-router}

<!-- TODO update links -->
<div>
  <VueSchoolLink href="https://vueschool.io/courses/vue-router-4-for-everyone" title="Kostenloser Vue Router-Kurs">
    Sehen Sie sich einen kostenlosen Videokurs auf Vue School an
  </VueSchoolLink>
</div>

Vue eignet sich gut für die Erstellung von SPAs. Für die meisten SPAs wird empfohlen, die offiziell unterstützte [Vue Router-Bibliothek](https://github.com/vuejs/router) zu verwenden. Weitere Informationen finden Sie in der [Dokumentation](https://router.vuejs.org/) zu Vue Router.

## Einfaches Routing von Grund auf {#simple-routing-from-scratch}

Wenn Sie nur sehr einfache Routing-Funktionen benötigen und keine voll ausgestattete Router-Bibliothek verwenden möchten, können Sie dies mit [Dynamic Components](/guide/essentials/component-basics# dynamic-components) und den aktuellen Komponentenstatus aktualisieren, indem Sie auf [`hashchange`-Ereignisse](https://developer.mozilla.org/en-US/docs/Web/API/Window/hashchange_event) im Browser achten oder die [History API](https://developer.mozilla.org/en-US/docs/Web/API/History) verwenden.

Hier ist ein einfaches Beispiel:

<div class="composition-api">

```vue
<script setup>
import { ref, computed } from 'vue'
import Home from './Home.vue'
import About from './About.vue'
import NotFound from './NotFound.vue'

const routes = {
  '/': Home,
  '/about': About
}

const currentPath = ref(window.location.hash)

window.addEventListener('hashchange', () => {
  currentPath.value = window.location.hash
})

const currentView = computed(() => {
  return routes[currentPath.value.slice(1) || '/'] || NotFound
})
</script>

<template>
  <a href="#/">Home</a> |
  <a href="#/about">About</a> |
  <a href="#/non-existent-path">Broken Link</a>
  <component :is="currentView" />
</template>
```

[Try it in the Playground](https://play.vuejs.org/#eNptUk1vgkAQ/SsTegAThZp4MmhikzY9mKanXkoPWxjLRpgly6JN1P/eWb5Eywlm572ZN2/m5GyKwj9U6CydsIy1LAyUaKpiHZHMC6UNnEDjbgqxyovKYAIX2GmVg8sktwe9qhzbdz+wga15TW++VWX6fB3dAt6UeVEVJT2me2hhEcWKSgOamVjCCk4RAbiBu6xbT5tI2ML8VDeI6HLlxZXWSOZdmJTJPJB3lJSoo5+pWBipyE9FmU4soU2IJHk+MGUrS4OE2nMtIk4F/aA7BW8Cq3WjYlDbP4isQu4wVp0F1Q1uFH1IPDK+c9cb1NW8B03tyJ//uvhlJmP05hM4n60TX/bb2db0CoNmpbxMDgzmRSYMcgQQCkjZhlXkPASRs7YmhoFYw/k+WXvKiNrTcQgpmuFv7ZOZFSyQ4U9a7ZFgK2lvSTXFDqmIQbCUJTMHFkQOBAwKg16kM3W6O7K3eSs+nbeK+eee1V/XKK0dY4Q3vLhR6uJxMUK8/AFKaB6k)

</div>

<div class="options-api">

```vue
<script>
import Home from './Home.vue'
import About from './About.vue'
import NotFound from './NotFound.vue'

const routes = {
  '/': Home,
  '/about': About
}

export default {
  data() {
    return {
      currentPath: window.location.hash
    }
  },
  computed: {
    currentView() {
      return routes[this.currentPath.slice(1) || '/'] || NotFound
    }
  },
  mounted() {
    window.addEventListener('hashchange', () => {
		  this.currentPath = window.location.hash
		})
  }
}
</script>

<template>
  <a href="#/">Home</a> |
  <a href="#/about">About</a> |
  <a href="#/non-existent-path">Broken Link</a>
  <component :is="currentView" />
</template>
```

[Try it in the Playground](https://play.vuejs.org/#eNptUstO6zAQ/ZVR7iKtVJKLxCpKK3Gli1ggxIoNZmGSKbFoxpEzoUi0/87YeVBKNonHPmfOmcdndN00yXuHURblbeFMwxtFpm6sY7i1NcLW2RriJPWBB8bT8/WL7Xh6D9FPwL3lG9tROWHGiwGmqLDUMjhhYgtr+FQEEKdxFqRXfaR9YrkKAoqOnocfQaDEre523PNKzXqx7M8ADrlzNEYAReccEj9orjLYGyrtPtnZQrOxlFS6rXqgZJdPUC5s3YivMhuTDCkeDe6/dSalvognrkybnIgl7c4UuLhcwuHgS3v2/7EPvzRruRXJ7/SDU12W/98l451pGQndIvaWi0rTK8YrEPx64ymKFQOce5DOzlfs4cdlkA+NzdNpBSRgrJudZpQIINdQOdyuVfQnVdHGzydP9QYO549hXIII45qHkKUL/Ail8EUjBgX+z9k3JLgz9OZJgeInYElAkJlWmCcDUBGkAsrTyWS0isYV9bv803x1OTiWwzlrWtxZ2lDGDO90mWepV3+vZojHL3QQKQE=)

</div>
