---
outline: deep
---

# Rendering-Mechanismus {#rendering-mechanism}

Wie nimmt Vue eine Vorlage und verwandelt sie in tatsächliche DOM-Knoten? Wie aktualisiert Vue diese DOM-Knoten effizient? Wir werden hier versuchen, diese Fragen zu klären, indem wir in den internen Rendering-Mechanismus von Vue eintauchen.

## Virtuelles DOM {#virtual-dom}

You have probably heard about the term "virtual DOM", which Vue's rendering system is based upon.

The virtual DOM (VDOM) is a programming concept where an ideal, or “virtual”, representation of a UI is kept in memory and synced with the “real” DOM. The concept was pioneered by [React](https://reactjs.org/), and has been adapted in many other frameworks with different implementations, including Vue.

Virtual DOM is more of a pattern than a specific technology, so there is no one canonical implementation. We can illustrate the idea using a simple example:

```js
const vnode = {
  type: 'div',
  props: {
    id: 'hello'
  },
  children: [
    /* more vnodes */
  ]
}
```

Hier ist „vnode“ ein einfaches JavaScript-Objekt (ein „virtueller Knoten“), das ein „<div>“-Element darstellt. Es enthält alle Informationen, die wir benötigen, um das eigentliche Element zu erstellen. Es enthält auch weitere untergeordnete vnodes, was es zur Wurzel eines virtuellen DOM-Baums macht.

Ein Laufzeit-Renderer kann einen virtuellen DOM-Baum durchlaufen und daraus einen realen DOM-Baum konstruieren. Dieser Vorgang wird **mount** genannt.

Wenn wir zwei Kopien von virtuellen DOM-Bäumen haben, kann der Renderer auch die beiden Bäume durchlaufen und vergleichen, die Unterschiede herausfinden und diese Änderungen auf das tatsächliche DOM anwenden. Dieser Vorgang wird **Patch** genannt, auch bekannt als „Diffing“ oder „Reconciliation“.

Der Hauptvorteil der virtuellen DOM besteht darin, dass sie dem Entwickler die Möglichkeit gibt, gewünschte UI-Strukturen auf deklarative Weise programmatisch zu erstellen, zu untersuchen und zusammenzustellen, während die direkte DOM-Manipulation dem Renderer überlassen bleibt.

## Render-Pipeline {#render-pipeline}

Auf hoher Ebene passiert Folgendes, wenn eine Vue-Komponente gemountet wird:

1. **Kompilieren**: Vue-Vorlagen werden in **Renderfunktionen** kompiliert: Funktionen, die virtuelle DOM-Bäume zurückgeben. Dieser Schritt kann entweder im Voraus über einen Build-Schritt oder spontan mithilfe des Laufzeit-Compilers durchgeführt werden.

2. **Mount**: Der Laufzeitrenderer ruft die Renderfunktionen auf, durchläuft den zurückgegebenen virtuellen DOM-Baum und erstellt darauf basierend tatsächliche DOM-Knoten. Dieser Schritt wird als [reaktiver Effekt](./reactivity-in-tiefe) ausgeführt, sodass alle verwendeten reaktiven Abhängigkeiten verfolgt werden.

3. **Patch**: Wenn sich eine beim Mounten verwendete Abhängigkeit ändert, wird der Effekt erneut ausgeführt. Dieses Mal wird ein neuer, aktualisierter virtueller DOM-Baum erstellt. Der Laufzeitrenderer durchläuft den neuen Baum, vergleicht ihn mit dem alten und wendet notwendige Aktualisierungen auf das tatsächliche DOM an.

![render pipeline](./images/render-pipeline.png)

<!-- https://www.figma.com/file/elViLsnxGJ9lsQVsuhwqxM/Rendering-Mechanism -->

## Vorlagen vs. Renderfunktionen {#templates-vs-render-functions}

Vue-Vorlagen werden in virtuelle DOM-Renderfunktionen kompiliert. Vue bietet auch APIs, die es uns ermöglichen, den Schritt der Vorlagenkompilierung zu überspringen und Renderfunktionen direkt zu erstellen. Renderfunktionen sind beim Umgang mit hochdynamischer Logik flexibler als Vorlagen, da Sie mit V-Nodes die volle Leistungsfähigkeit von JavaScript nutzen können.

Warum empfiehlt Vue also standardmäßig Vorlagen? Dafür gibt es mehrere Gründe:

1. Vorlagen sind näher am tatsächlichen HTML. Dies macht es einfacher, vorhandene HTML-Snippets wiederzuverwenden, Best Practices für Barrierefreiheit anzuwenden, mit CSS zu formatieren und für Designer einfacher zu verstehen und zu ändern.

2. Vorlagen lassen sich aufgrund ihrer deterministischeren Syntax einfacher statisch analysieren. Dadurch kann der Vorlagen-Compiler von Vue zahlreiche Optimierungen zur Kompilierungszeit anwenden, um die Leistung des virtuellen DOM zu verbessern (worauf wir weiter unten eingehen werden).

In der Praxis reichen Vorlagen für die meisten Anwendungsfälle in Anwendungen aus. Renderfunktionen werden normalerweise nur in wiederverwendbaren Komponenten verwendet, die mit hochdynamischer Renderinglogik umgehen müssen. Die Verwendung von Renderfunktionen wird ausführlicher in [Renderfunktionen und JSX](./render-function) erläutert.

## Compiler-informiertes virtuelles DOM {#compiler-informed-virtual-dom}

Die virtuelle DOM-Implementierung in React und die meisten anderen virtuellen DOM-Implementierungen sind reine Laufzeitimplementierungen: Der Abstimmungsalgorithmus kann keine Annahmen über den eingehenden virtuellen DOM-Baum treffen, daher muss er den Baum vollständig durchlaufen und die Eigenschaften jedes V-Knotens unterscheiden, um dies sicherzustellen Richtigkeit. Darüber hinaus werden bei jedem erneuten Rendern immer neue V-Knoten für sie erstellt, selbst wenn sich ein Teil des Baums nie ändert, was zu unnötigem Speicherdruck führt. Dies ist einer der am meisten kritisierten Aspekte des virtuellen DOM: Der etwas brutale Abstimmungsprozess opfert Effizienz im Gegenzug für Aussagekraft und Korrektheit.

Aber das muss nicht so sein. In Vue steuert das Framework sowohl den Compiler als auch die Laufzeit. Dadurch können wir viele Optimierungen zur Kompilierungszeit implementieren, die nur ein eng gekoppelter Renderer nutzen kann. Der Compiler kann die Vorlage statisch analysieren und Hinweise im generierten Code hinterlassen, sodass die Laufzeit nach Möglichkeit Verknüpfungen verwenden kann. Gleichzeitig behalten wir weiterhin die Möglichkeit für den Benutzer bei, zur Renderfunktionsebene zu wechseln, um in Randfällen eine direktere Kontrolle zu erhalten. Wir nennen diesen hybriden Ansatz **Compiler-Informed Virtual DOM**.

Im Folgenden besprechen wir einige wichtige Optimierungen, die der Vue-Vorlagencompiler vorgenommen hat, um die Laufzeitleistung des virtuellen DOM zu verbessern.

### Statisches Heben {#static-hoisting}

Sehr oft gibt es Teile in einer Vorlage, die keine dynamischen Bindungen enthalten:

```vue-html{2-3}
<div>
  <div>foo</div> <!-- hoisted -->
  <div>bar</div> <!-- hoisted -->
  <div>{{ dynamic }}</div>
</div>
```

[Überprüfen Sie im Vorlagen-Explorer](https://vue-next-template-explorer.netlify.app/#eyJzcmMiOiI8ZGl2PlxuICA8ZGl2PmZvbzwvZGl2PlxuICA8ZGl2PmJhcjwvZGl2PlxuICA8ZGl2Pnt7IGR5bmFtaWMgfX08L2Rpdj5cbjwvZGl2PiIsInNzciI6ZmFsc2UsIm9wdGlvbnMiOnsiaG9pc3RTdGF0aWMiOnRydWV9fQ==)

Die „foo“- und „bar“-Divs sind statisch – es ist nicht erforderlich, Vnodes neu zu erstellen und sie bei jedem erneuten Rendern zu unterscheiden. Der Vue-Compiler holt seine VNode-Erstellungsaufrufe automatisch aus der Renderfunktion und verwendet bei jedem Rendern dieselben VNodes wieder. Der Renderer kann die Differenzierung auch vollständig überspringen, wenn er feststellt, dass der alte V-Knoten und der neue V-Knoten identisch sind.

Wenn genügend aufeinanderfolgende statische Elemente vorhanden sind, werden diese außerdem zu einem einzigen „statischen V-Knoten“ zusammengefasst, der die einfache HTML-Zeichenfolge für alle diese Knoten enthält ([Beispiel](https://vue-next-template-explorer.netlify.app/#eyJzcmMiOiI8ZGl2PlxuICA8ZGl2IGNsYXNzPVwiZm9vXCI+Zm9vPC9kaXY+XG4gIDxkaXYgY2xhc3M9XCJmb29cIj5mb288L2Rpdj5cbiAgPGRpdiBjbGFzcz1cImZvb1wiPmZvbzwvZGl2PlxuICA8ZGl2IGNsYXNzPVwiZm9vXCI+Zm9vPC9kaXY+XG4gIDxkaXYgY2xhc3M9XCJmb29cIj5mb288L2Rpdj5cbiAgPGRpdj57eyBkeW5hbWljIH19PC9kaXY+XG48L2Rpdj4iLCJzc3IiOmZhbHNlLCJvcHRpb25zIjp7ImhvaXN0U3RhdGljIjp0cnVlfX0=)). Diese statischen VNodes werden durch direktes Festlegen von „innerHTML“ gemountet. Außerdem speichern sie beim ersten Mounten ihre entsprechenden DOM-Knoten im Cache – wenn derselbe Inhalt an anderer Stelle in der App wiederverwendet wird, werden neue DOM-Knoten mit nativem „cloneNode()“ erstellt, was äußerst effizient ist.

### Patch-Flags {#patch-flags}

Für ein einzelnes Element mit dynamischen Bindungen können wir zur Kompilierungszeit auch viele Informationen daraus ableiten:

```vue-html
<!-- class binding only -->
<div :class="{ active }"></div>

<!-- id and value bindings only -->
<input :id="id" :value="value">

<!-- text children only -->
<div>{{ dynamic }}</div>
```

[Überprüfen Sie im Vorlagen-Explorer](https://template-explorer.vuejs.org/#eyJzcmMiOiI8ZGl2IDpjbGFzcz1cInsgYWN0aXZlIH1cIj48L2Rpdj5cblxuPGlucHV0IDppZD1cImlkXCIgOnZhbHVlPVwidmFsdWVcIj5cblxuPGRpdj57eyBkeW5hbWljIH19PC9kaXY+Iiwib3B0aW9ucyI6e319)

Beim Generieren des Render-Funktionscodes für diese Elemente kodiert Vue den Aktualisierungstyp, den jedes von ihnen benötigt, direkt im Vnode-Erstellungsaufruf:

```js{3}
createElementVNode("div", {
  class: _normalizeClass({ active: _ctx.active })
}, null, 2 /* CLASS */)
```

Das letzte Argument, „2“, ist ein [Patch-Flag](https://github.com/vuejs/core/blob/main/packages/shared/src/patchFlags.ts). Ein Element kann mehrere Patch-Flags haben, die zu einer einzigen Nummer zusammengefasst werden. Der Laufzeitrenderer kann dann mithilfe von [bitweisen Operationen](https://en.wikipedia.org/wiki/Bitwise_operation) anhand der Flags ermitteln, ob bestimmte Arbeiten ausgeführt werden müssen:

```js
if (vnode.patchFlag & PatchFlags.CLASS /* 2 */) {
  // update the element's class
}
```

Bitweise Prüfungen sind extrem schnell. Mit den Patch-Flags ist Vue in der Lage, beim Aktualisieren von Elementen mit dynamischen Bindungen den geringsten Arbeitsaufwand zu verursachen.

Vue kodiert auch die Art der untergeordneten Elemente eines V-Knotens. Beispielsweise wird eine Vorlage mit mehreren Stammknoten als Fragment dargestellt. In den meisten Fällen wissen wir sicher, dass sich die Reihenfolge dieser Root-Knoten nie ändern wird, daher können diese Informationen auch als Patch-Flag an die Laufzeit bereitgestellt werden:

```js{4}
export function render() {
  return (_openBlock(), _createElementBlock(_Fragment, null, [
    /* children */
  ], 64 /* STABLE_FRAGMENT */))
}
```

Die Laufzeit kann daher den Abgleich der untergeordneten Reihenfolge für das Stammfragment vollständig überspringen.

### Baumabflachung {#tree-flattening}

Wenn Sie sich den generierten Code aus dem vorherigen Beispiel noch einmal ansehen, werden Sie feststellen, dass die Wurzel des zurückgegebenen virtuellen DOM-Baums mit einem speziellen „createElementBlock()“-Aufruf erstellt wird:

```js{2}
export function render() {
  return (_openBlock(), _createElementBlock(_Fragment, null, [
    /* children */
  ], 64 /* STABLE_FRAGMENT */))
}
```

Konzeptionell ist ein „Block“ ein Teil der Vorlage, der eine stabile innere Struktur aufweist. In diesem Fall besteht die gesamte Vorlage aus einem einzigen Block, da sie keine Strukturanweisungen wie „v-if“ und „v-for“ enthält.

Jeder Block verfolgt alle untergeordneten Knoten (nicht nur direkte untergeordnete Knoten), die Patch-Flags haben. Zum Beispiel:

```vue-html{3,5}
<div> <!-- root block -->
  <div>...</div>         <!-- not tracked -->
  <div :id="id"></div>   <!-- tracked -->
  <div>                  <!-- not tracked -->
    <div>{{ bar }}</div> <!-- tracked -->
  </div>
</div>
```

Das Ergebnis ist ein abgeflachtes Array, das nur die dynamischen untergeordneten Knoten enthält:

```
div (block root)
- div with :id binding
- div with {{ bar }} binding
```

Wenn diese Komponente erneut gerendert werden muss, muss sie nur den abgeflachten Baum und nicht den gesamten Baum durchlaufen. Dies wird als **Tree Flattening** bezeichnet und reduziert die Anzahl der Knoten, die während der virtuellen DOM-Abstimmung durchlaufen werden müssen, erheblich. Alle statischen Teile der Vorlage werden effektiv übersprungen.

`v-if` und `v-for` Direktiven erstellen neue Blockknoten:

```vue-html
<div> <!-- root block -->
  <div>
    <div v-if> <!-- if block -->
      ...
    <div>
  </div>
</div>
```

Ein untergeordneter Block wird innerhalb des Arrays dynamischer Nachkommen des übergeordneten Blocks verfolgt. Dadurch bleibt eine stabile Struktur für den übergeordneten Block erhalten.

### Auswirkungen auf die SSR-Hydratation {#impact-on-ssr-hydration}

Sowohl Patch-Flags als auch Tree-Flattening verbessern auch die [SSR Hydration](/guide/scaling-up/ssr.html#client-hydration)-Leistung von Vue erheblich:

- Die Hydratation einzelner Elemente kann basierend auf der Patch-Flagge des entsprechenden V-Knotens schnelle Wege nehmen.

- Während der Hydratation müssen nur Blockknoten und ihre dynamischen Nachkommen durchlaufen werden, wodurch effektiv eine teilweise Hydratation auf Template-Ebene erreicht wird.
