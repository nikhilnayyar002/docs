
- [1. Cascading variables](#1-cascading-variables)
  - [1.1. Using CSS custom properties](#11-using-css-custom-properties)
- [2. Containment](#2-containment)
  - [2.1. CSS container queries](#21-css-container-queries)
    - [2.1.1. Using container size queries](#211-using-container-size-queries)
    - [2.1.2. Naming containment contexts](#212-naming-containment-contexts)
    - [2.1.3. Name-only container queries](#213-name-only-container-queries)
    - [2.1.4. Container query length units](#214-container-query-length-units)
  - [2.2. Using CSS containment](#22-using-css-containment)
  - [2.3. Using container size and style queries](#23-using-container-size-and-style-queries)


# 1. Cascading variables

## 1.1. Using CSS custom properties

They are set using the `@property` at-rule or by custom property syntax (e.g., `--primary-color: blue;`). Custom properties are accessed using the CSS `var()` function (e.g., `color: var(--primary-color);`).

```css
/* The selector given defines the scope for that custom property*/
section { --main-bg-color: brown; }
/* properties on the :root pseudo-class, will be referenced globally */
:root { --main-bg-color: brown; }
```
A custom property defined using two dashes `-- `instead of `@property` always inherits the value of its parent. Below `class="four"` gets `teal` (inherited from its parent)

```html
<style>
div {
  background-color: var(--box-color);
}
.two {
  --box-color: teal;
}
.three {
  --box-color: pink;
}
</style>
  <div class="two">
    <div class="three"><p>Three</p></div>
    <div class="four"><p>Four</p></div> 
  </div>
```
```css
@property --box-color {
  syntax: "<color>";
  inherits: false;
  initial-value: teal;
}

.parent {
  --box-color: green;
  background-color: var(--box-color); /* green */
}

.child {
  background-color: var(--box-color);  /* teal */
}
```
```css
.one {
  color: var(--my-var, red); /* Red if --my-var is not defined */
  color: var(--my-var, var(--my-background, pink));
}
```
When the browser encounters an invalid `var()` substitution and no fallback value is provided, then the `initial` or `inherited` value of the property is used.

> Note: when you set property in some class for a given element, then that property gets set on that element. It can be accessed by any other class or inline style of that element. Example:
> ```html
> <style>
> .some { --my-var: blur(3px);}
> .dude { filter: var(--my-var); }
> </style>
> <div class="some dude"></div>
> ```

# 2. Containment

## 2.1. [CSS container queries](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Containment/Container_queries)

Container queries are an alternative to media queries, which apply styles to elements based on viewport size or other device characteristics.

### 2.1.1. Using container size queries

To use container size queries, you need to declare a containment context on an element so that the browser knows you might want to query the dimensions of this container later. [@container](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@container) at-rule to define a container query. The query in the following example will apply styles to elements based on the size of the nearest ancestor with a containment context.
```html
<div class="post">
  <div class="card">
    <h2>Card title</h2>
    <p>Card content</p>
  </div>
</div>
```
```css
.post {
  container-type: inline-size; /* containment context */
}
.card h2 { font-size: 1em; }
@container (width > 700px) {
  .card h2 { font-size: 2em;}
}
```

### 2.1.2. Naming containment contexts

the name can be used in a @container query so as to target a specific container.
```css
.post { container-type: inline-size; container-name: sidebar; }
@container sidebar (width > 700px) { .card { font-size: 2em; } }
```

### 2.1.3. Name-only container queries

As well as using a `container-name` along with a [`<container-query>`](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@container#container-query), you can query a container using just its name

```html
<div id="container">
  <p>I'm in the container.</p>
</div>
```
```css
#container { container-name: my-container; }
@container my-container { p { background-color: lime; } }
```

### 2.1.4. [Container query length units](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Containment/Container_queries#container_query_length_units)

```css
@container (width > 700px) {
  .card h2 { font-size: max(1.5em, 1.23em + 2cqi); }
}
```

## 2.2. [Using CSS containment](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Containment/Using)

## 2.3. [Using container size and style queries](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Containment/Container_size_and_style_queries)

