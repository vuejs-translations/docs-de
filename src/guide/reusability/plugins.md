# Plugins

## Einführung {#introduction}

Plugins sind in sich geschlossener Code, der Vue normalerweise um Funktionen auf App-Ebene erweitert. So installieren wir ein Plugin:

```js
import { createApp } from 'vue'

const app = createApp({})

app.use(myPlugin, {
  /* optional options */
})
```

Ein Plugin ist entweder ein Objekt, das eine `install()`-Methode bereitstellt, oder eine Funktion, die selbst als Installationsfunktion fungiert. Die Installationsfunktion empfängt die [App-Instanz](/api/application.html) zusammen mit zusätzlichen Optionen, die gegebenenfalls an `app.use()` übergeben werden:

```js
const myPlugin = {
  install(app, options) {
    // configure the app
  }
}
```

Es gibt keinen streng definierten Anwendungsbereich für ein Plugin, aber gängige Szenarien, in denen Plugins nützlich sind, sind unter anderem:

1. Registrieren Sie eine oder mehrere globale Komponenten oder benutzerdefinierte Anweisungen mit [`app.component()`](/api/application.html#app-component) und [`app.directive()`](/api/application.html#app-directive).

2. Machen Sie eine Ressource [injizierbar](/guide/components/provide-inject.html) in der gesamten App durch Anrufen [`app.provide()`](/api/application.html#app-provide).

3. Fügen Sie einige globale Instanzeigenschaften oder Methoden hinzu, indem Sie sie anhängen an [`app.config.globalProperties`](/api/application.html#app-config-globalproperties).

4. Eine Bibliothek, die eine Kombination der oben genannten Aufgaben ausführen muss (z. B. [vue-router](https://github.com/vuejs/vue-router-next)).

## Ein Plugin schreiben {#writing-a-plugin}

Um besser zu verstehen, wie Sie Ihre eigenen Vue.js-Plugins erstellen, erstellen wir eine sehr vereinfachte Version eines Plugins, das `i18n`-Strings (kurz für [Internationalization](https://en.wikipedia.org/wiki/Internationalization_and_localization)) anzeigt.

Beginnen wir mit der Einrichtung des Plugin-Objekts. Es wird empfohlen, es in einer separaten Datei zu erstellen und wie unten gezeigt zu exportieren, um die Logik in sich geschlossen und getrennt zu halten.

```js
// plugins/i18n.js
export default {
  install: (app, options) => {
    // Plugin code goes here
  }
}
```

Wir möchten eine Übersetzungsfunktion erstellen. Diese Funktion erhält einen durch Punkte getrennten Schlüsselstring, mit dem wir den übersetzten String in den benutzerdefinierten Optionen suchen. Dies ist die vorgesehene Verwendung in Vorlagen:

```vue-html
<h1>{{ $translate('greetings.hello') }}</h1>
```

Da diese Funktion global in allen Vorlagen verfügbar sein sollte, machen wir das so, indem wir sie in unserem Plugin an `app.config.globalProperties` anhängen:

```js{4-11}
// plugins/i18n.js
export default {
  install: (app, options) => {
    // inject a globally available $translate() method
    app.config.globalProperties.$translate = (key) => {
      // retrieve a nested property in `options`
      // using `key` as the path
      return key.split('.').reduce((o, i) => {
        if (o) return o[i]
      }, options)
    }
  }
}
```

Unsere Funktion `$translate` nimmt eine Zeichenfolge wie `greetings.hello`, sieht sich die vom Benutzer bereitgestellte Konfiguration an und gibt den übersetzten Wert zurück.

Das Objekt mit den übersetzten Schlüsseln sollte während der Installation über zusätzliche Parameter an `app.use()` an das Plugin übergeben werden:

```js
import i18nPlugin from './plugins/i18n'

app.use(i18nPlugin, {
  greetings: {
    hello: 'Bonjour!'
  }
})
```

Nun wird unser ursprünglicher Ausdruck `$translate(‚greetings.hello‘)` zur Laufzeit durch `Bonjour!` ersetzt.

Siehe auch: [Erweiterung globaler Eigenschaften](/guide/typescript/options-api.html#augmenting-global-properties) <sup class="vt-badge ts" />

:::tip
Verwenden Sie globale Eigenschaften nur sparsam, da es schnell zu Verwirrung kommen kann, wenn zu viele globale Eigenschaften, die von verschiedenen Plugins eingefügt wurden, in einer App verwendet werden.
:::

### Bereitstellen/Einfügen mit Plugins {#provide-inject-with-plugins}

Plugins also allow us to use `inject` to provide a function or attribute to the plugin's users. For example, we can allow the application to have access to the `options` parameter to be able to use the translations object.

```js{10}
// plugins/i18n.js
export default {
  install: (app, options) => {
    app.config.globalProperties.$translate = (key) => {
      return key.split('.').reduce((o, i) => {
        if (o) return o[i]
      }, options)
    }

    app.provide('i18n', options)
  }
}
```

Plugin users will now be able to inject the plugin options into their components using the `i18n` key:

<div class="composition-api">

```vue
<script setup>
import { inject } from 'vue'

const i18n = inject('i18n')

console.log(i18n.greetings.hello)
</script>
```

</div>
<div class="options-api">

```js
export default {
  inject: ['i18n'],
  created() {
    console.log(this.i18n.greetings.hello)
  }
}
```

</div>
