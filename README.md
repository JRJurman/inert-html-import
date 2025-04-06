# inert-html-import
Proposal / Demo for Inert HTML Import for Declarative Custom Elements

> [!important]
> At this point, this repository represents a proposal only. A demo implementation will (hopefully) be coming soon.

## Proposal

The intent of this repository is to propose and describe an interface for importing inert HTML content via a `src` attribute on the `<template>` element.
This would allows us an elegant interface for importing Web Component templates, without the complexity associated with processing and evaluating external DOM elements in the current document. It is a stepping stone towards a fully Declarative Custom Element interface.

```html
<template src="external.html"></template>
```

## Example

Take the `<element-details>`, presented by mdn ([code here](https://github.com/mdn/web-components-examples/tree/main/element-details), [live example here](https://mdn.github.io/web-components-examples/element-details/))

We could imagine two new implementations for the above sample.

### Component Definition using imported Template (with Javascript)

A more 1-to-1 mapping of the existing implementation might involve putting the template content in a new `element-details.html`, and then importing that in our `index.html`

`element-details.html`
```html
<style>
  details {font-family: "Open Sans Light",Helvetica,Arial}
  .name {font-weight: bold; color: #217ac0; font-size: 120%}
  h4 { margin: 10px 0 -8px 0; }
  h4 span { background: #217ac0; padding: 2px 6px 2px 6px }
  h4 span { border: 1px solid #cee9f9; border-radius: 4px }
  h4 span { color: white }
  .attributes { margin-left: 22px; font-size: 90% }
  .attributes p { margin-left: 16px; font-style: italic }
</style>
<details>
  <summary>
    <span>
      <code class="name">&lt;<slot name="element-name">NEED NAME</slot>&gt;</code>
      <i class="desc"><slot name="description">NEED DESCRIPTION</slot></i>
    </span>
  </summary>
  <div class="attributes">
    <h4><span>Attributes</span></h4>
    <slot name="attributes"><p>None</p></slot>
  </div>
</details>
<hr>
```

`index.html`
```html
<!DOCTYPE html>
<html>
  <head>
    <!-- ... -->
  </head>
  <body>
    <template id="element-details-template" src="element-details.html">
    </template>

    <element-details>
      <span slot="element-name">slot</span>
      <span slot="description">A placeholder inside a web
        component that users can fill with their own markup,
        with the effect of composing different DOM trees
        together.</span>
      <dl slot="attributes">
        <dt>name</dt>
        <dd>The name of the slot.</dd>
      </dl>
    </element-details>

    <element-details>
      <span slot="element-name">template</span>
      <span slot="description">A mechanism for holding client-
        side content that is not to be rendered when a page is
        loaded but may subsequently be instantiated during
        runtime using JavaScript.</span>
    </element-details>

    <script src="main.js"></script>
  </body>
</html>
```

This would still depend on the `main.js` to pull the template content and define it as a new web-component.

### Using Declarative Shadow DOM with imported Template (no javascript)

We could avoid the need for any javascript by instead pairing this with our existing DSD implemention. With the same `element-details.html` above, we could imagine the `index.html` looking something more like the following:

`index.html`
```html
<!DOCTYPE html>
<html>
  <head>
    <!-- ... -->
  </head>
  <body>

    <element-details>
      <template src="element-details.html" shadowrootmode="open"></template>
      <span slot="element-name">slot</span>
      <span slot="description">A placeholder inside a web
        component that users can fill with their own markup,
        with the effect of composing different DOM trees
        together.</span>
      <dl slot="attributes">
        <dt>name</dt>
        <dd>The name of the slot.</dd>
      </dl>
    </element-details>

    <element-details>
      <template src="element-details.html" shadowrootmode="open"></template>
      <span slot="element-name">template</span>
      <span slot="description">A mechanism for holding client-
        side content that is not to be rendered when a page is
        loaded but may subsequently be instantiated during
        runtime using JavaScript.</span>
    </element-details>

  </body>
</html>
```

By including the template directly in the `<element-details>`, and adding the `shadowrootmode` attribute, we have an external web-component definition with no required javascript!  

### Future Interface with Declarative Custom Elements

While not part of this proposal, the end goal is still to provide an elegant interface for defining custom elements purely in HTML, without relying on Javascript or Javascript imports. We could, taking the above example into account again, imagine a new definition element, which uses this imported template functionality.

`index.html`
```html
<!DOCTYPE html>
<html>
  <head>
    <!-- ... -->
  </head>
  <body>
    <define tag="element-details">
      <template src="element-details.html" shadowrootmode="open"></template>
    </define>

    <element-details>
      <span slot="element-name">slot</span>
      <span slot="description">A placeholder inside a web
        component that users can fill with their own markup,
        with the effect of composing different DOM trees
        together.</span>
      <dl slot="attributes">
        <dt>name</dt>
        <dd>The name of the slot.</dd>
      </dl>
    </element-details>

    <element-details>
      <span slot="element-name">template</span>
      <span slot="description">A mechanism for holding client-
        side content that is not to be rendered when a page is
        loaded but may subsequently be instantiated during
        runtime using JavaScript.</span>
    </element-details>

  </body>
</html>
```

## Motivation

The `<template>` element is a good starting place for a couple of reasons:

1. It allows us to make use of the existing and future Declarative Shadow DOM attributes already supported by browsers (`shadowrootmode`, `delegatesfocus`, etc)
2. The inert nature of the `<template>` element means we may avoid some unclear side-effects of injecting certain elements (like a `<script>` or `<style>`) 
3. The browser already supports the ability to change and update the contents of this element based on its attributes (this is what happens today when `shadowrootmode` is present)
