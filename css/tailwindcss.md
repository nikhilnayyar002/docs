https://tailwindcss.com/docs

- [1. Core concepts](#1-core-concepts)
  - [1.1. Styling with utility classes (tailwind overview)](#11-styling-with-utility-classes-tailwind-overview)
  - [1.2. Hover, focus, and other states](#12-hover-focus-and-other-states)
    - [1.2.1. Pseudo-classes](#121-pseudo-classes)
    - [1.2.2. Pseudo-elements](#122-pseudo-elements)
    - [1.2.3. Media and feature queries](#123-media-and-feature-queries)
    - [1.2.4. Attribute selectors (aria-, data-, RTL, `<details>` etc)](#124-attribute-selectors-aria--data--rtl-details-etc)
    - [1.2.5. Child selectors](#125-child-selectors)
    - [1.2.6. Custom variants](#126-custom-variants)
    - [1.2.7. Quick reference](#127-quick-reference)
  - [Responsive design](#responsive-design)
    - [Using custom breakpoints](#using-custom-breakpoints)
    - [Container queries](#container-queries)


# 1. Core concepts

## 1.1. Styling with utility classes (tailwind overview)

- The display and padding utilities (`flex, shrink-0, and p-6`) to control the overall layout
- The `max-width` and margin utilities (`max-w-sm` and `mx-auto`) to constrain the card width and center it horizontally
- The background-color, border-radius, and box-shadow utilities (`bg-white`, `rounded-xl`, and `shadow-lg`) to style the card's appearance
- The width and height utilities (`size-12`) to set the width and height of the logo image
- The gap utilities (`gap-x-4`) to handle the spacing between the logo and the text
- The `font-size, color, and font-weight` utilities (`text-xl, text-black, font-medium`, etc.) to style the card text

**Styling hover and focus states:**
```html
<button class="bg-sky-500 hover:bg-sky-700 ...">Save changes</button>
```
**Media queries and breakpoints**
```html
<!-- makes sure that grid-cols-3 only triggers at the sm breakpoint and above -->
<div class="grid grid-cols-2 sm:grid-cols-3"></div>
```
**Targeting dark mode**
```html
<!-- light mode is default, you style things in dark mode -->
<div class="bg-white dark:bg-gray-800"></div>
```
**Using class composition**

A lot of the time with Tailwind you'll even use multiple classes to build up the value for a single CSS property (`filter` here):
```html
<div class="blur-sm grayscale"></div>
```
```css
.blur-sm {
  --tw-blur: blur(var(--blur-sm));
  /* filter: blur(var(--blur-sm)); */
  filter: var(--tw-blur,) var(--tw-brightness,) var(--tw-grayscale,);
}
.grayscale {
  --tw-grayscale: grayscale(100%);
  /* filter: blur(var(--blur-sm)) grayscale(100%); */
  filter: var(--tw-blur,) var(--tw-brightness,) var(--tw-grayscale,);
}
```

**Using arbitrary values**

Many utilities in Tailwind are driven by theme variables, like `bg-blue-500, text-xl, and shadow-md`, which map to your underlying color palette, type scale, and shadows.

When you need to use a one-off value outside of your theme, use the special square bracket syntax:
```html
<button class="bg-[#316ff6] ...">
<div class="grid grid-cols-[24rem_2.5rem_minmax(0,1fr)]">
<div class="max-h-[calc(100dvh-(--spacing(6)))]">
<div class="[--gutter-width:1rem] lg:[--gutter-width:2rem]">
```

**Complex selectors**

```html
<button class="dark:lg:data-current:hover:bg-indigo-600 ...">
```
```css
@media (prefers-color-scheme: dark) and (width >= 64rem) {
  button[data-current]:hover {
    background-color: var(--color-indigo-600);
  }
}
```

For really complex scenarios (especially when styling HTML you don't control), Tailwind supports arbitrary variants which let you write any selector you want, directly in a class name:
```html
<div class="[&>[data-active]+span]:text-blue-600 ...">
  <span data-active><!-- ... --></span>
  <span>This text will be blue</span>
</div>
```
```css
div > [data-active] + span {
  color: var(--color-blue-600);
}
```

**When to use inline styles**
- when a value is coming from a dynamic source like a database or API. `style={{backgroundColor: buttonColor}}`
- very complicated arbitrary values that are difficult to read when formatted as a class name
  ```html
  <div class="grid-[2fr_max(0,var(--gutter-width))_calc(var(--gutter-width)+10px)]">
  <div style="grid-template-columns: 2fr max(0, var(--gutter-width)) calc(var(--gutter-width) + 10px)">
  ```
- setting CSS variables based on dynamic sources
  ```jsx
  <button
    style={{
      "--bg-color": buttonColor,
      "--bg-color-hover": buttonColorHover,
      "--text-color": textColor,
    }}
    className="bg-(--bg-color) text-(--text-color) hover:bg-(--bg-color-hover)"
  >
  ```

**Using custom CSS**

```css
@import "tailwindcss";
@layer components {
  .btn-primary {
    border-radius: calc(infinity * 1px);
    background-color: var(--color-violet-500);
    padding-inline: --spacing(5);
    padding-block: --spacing(2);
    font-weight: var(--font-weight-semibold);
    color: var(--color-white);
    box-shadow: var(--shadow-md);
    &:hover {
      @media (hover: hover) {
        background-color: var(--color-violet-700);
      }
    }
  }
}
```

**Conflicting utility classes**

you should just never add two conflicting classes to the same element — only ever add the one you actually want to take effect:
```jsx
<div className={gridLayout ? "grid" : "flex"}>
```
```html
<div class="bg-teal-500 bg-red-500!"> <!-- worst case, use important modifier -->
```
```css
@import "tailwindcss" important; /* mark all utilities as !important */
```

**Using the prefix option**

```css
@import "tailwindcss" prefix(tw); /* prefix all classes and CSS variables */
```

## 1.2. Hover, focus, and other states

### 1.2.1. [Pseudo-classes](https://tailwindcss.com/docs/hover-focus-and-other-states#pseudo-classes)

https://tailwindcss.com/docs/hover-focus-and-other-states#pseudo-class-reference

...

### 1.2.2. [Pseudo-elements](https://tailwindcss.com/docs/hover-focus-and-other-states#pseudo-elements)

...

### 1.2.3. [Media and feature queries](https://tailwindcss.com/docs/hover-focus-and-other-states#media-and-feature-queries)

...

### 1.2.4. [Attribute selectors](https://tailwindcss.com/docs/hover-focus-and-other-states#attribute-selectors) (aria-, data-, RTL, `<details>` etc)

...

### 1.2.5. [Child selectors](https://tailwindcss.com/docs/hover-focus-and-other-states#child-selectors)

...

### 1.2.6. Custom variants

**Using arbitrary variants**

Just like arbitrary values (`<div class="top-[117px]">`) let you use custom values with your utility classes, arbitrary variants let you write custom selector variants directly in your HTML. Arbitrary variants are just format strings that represent the selector, wrapped in square brackets.
```html
<!-- changes the cursor to grabbing when the element has the is-dragging class -->
<li class="[&.is-dragging]:cursor-grabbing">{item}</li>
<!-- If you need spaces in your selector, you can use an underscore.
selects all p elements within the element  -->
<div class="[&_p]:mt-4">
<!-- @media or @supports in arbitrary variants -->
<div class="flex [@supports(display:grid)]:grid">
```

**Registering a custom variant**

```css
@custom-variant theme-midnight (&:where([data-theme="midnight"] *));
```
```html
<button class="theme-midnight:bg-black">
```
Learn more about adding custom variants in the [adding custom variants documentation](https://tailwindcss.com/docs/adding-custom-styles#adding-custom-variants)


### 1.2.7. [Quick reference](https://tailwindcss.com/docs/hover-focus-and-other-states#quick-reference)

## Responsive design

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<!-- Width of 16 by default, 32 on medium screens, and 48 on large screens -->
<img class="w-16 md:w-32 lg:w-48" src="..." />
```
```txt
Breakpoint prefix	    Minimum width	      CSS
sm	                    40rem (640px)	      @media (width >= 40rem) { ... }
md	                    48rem (768px)	      @media (width >= 48rem) { ... }
lg	                    64rem (1024px)	      @media (width >= 64rem) { ... }
xl	                    80rem (1280px)	      @media (width >= 80rem) { ... }
2xl	                    96rem (1536px)	      @media (width >= 96rem) { ... }
```

**Targeting a breakpoint range**

If you'd like to apply a utility only when a specific breakpoint range is active, stack a responsive variant like `md` with a `max-*` variant to limit that style to a specific range:
```html
<!-- @media (48rem <= width <= 80rem) -->
<div class="md:max-xl:flex">
```

### Using custom breakpoints

**Customizing your theme**
```css
@theme {
  --breakpoint-2xl: 100rem; /* existing */
  --breakpoint-3xl: 120rem; /* create new */
}
```
[**Removing default breakpoints**](https://tailwindcss.com/docs/responsive-design#removing-default-breakpoints)

...

**Using arbitrary values**

use the min or max variants to generate a custom breakpoint on the fly
```html
<div class="max-[600px]:bg-sky-300 min-[320px]:text-center">
```

### [Container queries](https://tailwindcss.com/docs/responsive-design#container-queries)

...

