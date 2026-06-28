https://tailwindcss.com/docs

v4.3

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
  - [1.3. Responsive design](#13-responsive-design)
    - [1.3.1. Using custom breakpoints](#131-using-custom-breakpoints)
    - [1.3.2. Container queries](#132-container-queries)
  - [1.4. Dark mode](#14-dark-mode)
  - [1.5. Theme variables](#15-theme-variables)
    - [1.5.1. Overview](#151-overview)
    - [1.5.2. Customizing your theme](#152-customizing-your-theme)
    - [1.5.3. Using your theme variables](#153-using-your-theme-variables)
  - [1.6. Colors](#16-colors)
  - [1.7. Adding custom styles](#17-adding-custom-styles)
    - [1.7.1. Using arbitrary values](#171-using-arbitrary-values)
    - [1.7.2. Using custom CSS](#172-using-custom-css)
    - [1.7.3. Adding custom utilities](#173-adding-custom-utilities)
  - [1.8. Detecting classes in source files](#18-detecting-classes-in-source-files)
  - [1.9. Functions and directives](#19-functions-and-directives)
    - [1.9.1. Directives](#191-directives)
    - [1.9.2. Functions](#192-functions)
- [2. Base styles](#2-base-styles)
  - [2.1. Preflight](#21-preflight)


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

## 1.3. Responsive design

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

### 1.3.1. Using custom breakpoints

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

### 1.3.2. [Container queries](https://tailwindcss.com/docs/responsive-design#container-queries)

...

## 1.4. Dark mode

...

[**Toggling dark mode manually**](https://tailwindcss.com/docs/dark-mode#toggling-dark-mode-manually)

If you want your dark theme to be driven by a CSS selector instead of the `prefers-color-scheme` media query, override the dark variant to use your custom selector:
```css
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));
```
Now instead of `dark:*` utilities being applied based on `prefers-color-scheme`, they will be applied whenever the `dark` class is present earlier in the HTML tree:
```html
<html class="dark">
  <body>
    <div class="bg-white dark:bg-black">
```

To build three-way theme toggles that support light, dark, and system theme:

https://tailwindcss.com/docs/dark-mode#with-system-theme-support

## 1.5. Theme variables

### 1.5.1. Overview

you can add a new color to your project by defining a theme variable like:
```css
@theme { --color-mint-500: oklch(0.72 0.11 178); }
```
Now you can use utility classes like `bg-mint-500`, `text-mint-500`, or `fill-mint-500` and CSS variable `--color-mint-500`

Some theme variables are used to define variants rather than utilities (eg: `--breakpoint-*`)

https://tailwindcss.com/docs/theme#theme-variable-namespaces

https://tailwindcss.com/docs/theme#default-theme-variable-reference

### 1.5.2. Customizing your theme

Override a default theme variable value by redefining it within `@theme`.

To completely override an entire namespace in the default theme, set the entire namespace to `initial`.
```css
@theme { --color-*: initial; --color-white: #fff;}
```
all of the default utilities that use that namespace (like `bg-red-500`) will be removed

To completely disable the default theme:
```css
@theme { --*: initial; /*... custom values */ }
```

**Defining animation keyframes**

Define the `@keyframes` rules for your `--animate-*` theme variables within `@theme` to use it as a animate class:
```css
@theme {
  --animate-fade-in-scale: fade-in-scale 0.3s ease-out;
  @keyframes fade-in-scale { ... }
}
```

**Referencing other variables**
```css
@theme inline { --font-sans: var(--font-inter); } /*  use the inline option */
```

**Generating all CSS variables**

By default only used CSS variables will be generated in the final CSS output. If you want to always generate all CSS variables:
```css
@theme static { --color-primary: var(--color-red-500); }
```

**Sharing across projects**

```css
/* ./packages/brand/theme.css */
@theme { ... }

/* ./packages/admin/app.css */
@import "tailwindcss";
@import "../brand/theme.css";
```

### 1.5.3. Using your theme variables

**With arbitrary values**
```html
<div class="absolute inset-px rounded-[calc(var(--radius-xl)-1px)]">
```

## 1.6. [Colors](https://tailwindcss.com/docs/colors)

Use color utilities like `bg-white`, `border-pink-300`, `text-gray-950` etc

full list of utilities: https://tailwindcss.com/docs/colors#using-color-utilities

[**Adjusting opacity**](https://tailwindcss.com/docs/colors#adjusting-opacity)
```html
<div class="bg-sky-500/10">
<div class="bg-pink-500/[71.37%]">
<div class="bg-cyan-400/(--my-alpha-value)">
```
in css you can use [alpha funtion](https://tailwindcss.com/docs/functions-and-directives#alpha-function)

## 1.7. Adding custom styles

### 1.7.1. Using arbitrary values

```html
<div class="top-[117px] lg:top-[344px]">
<div class="bg-[#bada55] text-[22px] before:content-['Festivus']">
<div class="fill-(--my-brand-color) ..."> <!-- shorthand for fill-[var(--my-brand-color)] -->

<!-- Arbitrary properties -->
<div class="[mask-type:luminance] hover:[mask-type:alpha]">
<div class="[--scroll-offset:56px] lg:[--scroll-offset:44px]">

<!-- Arbitrary variants -->
<li class="lg:[&:nth-child(-n+3)]:hover:underline">
```

**Handling whitespace**
```html
<!-- Handling whitespace, use _ -->
<div class="grid grid-cols-[1fr_500px_2fr]">

<!-- In situations where _ is common but spaces are invalid, Tailwind will preserve _ -->
<div class="bg-[url('/what_a_rush.png')]">

<!-- In the rare case when it's ambiguous, escape the _ -->
<div class="before:content-['hello\_world']">
```
If you're using something like JSX where the backslash is stripped from the rendered HTML:
```jsx
<div className={String.raw`before:content-['hello\_world']`}>
```

**Resolving ambiguities**
```html
<div class="text-[22px]"> <!-- Will generate a font-size utility -->
<div class="text-[#bada55]"> <!-- Will generate a color utility -->
<div class="text-(--my-var)"> <!-- ambiguous -->
<!-- https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Types -->
<div class="text-(length:--my-var)"> <!-- Will generate a font-size utility -->
<div class="text-(color:--my-var)">  <!-- Will generate a color utility -->
```

### 1.7.2. Using custom CSS

**Adding base styles**
```html
<html lang="en" class="bg-gray-100 font-serif text-gray-900">
```
If you want to add your own default base styles for specific HTML elements, use the `@layer` directive to add those styles to Tailwind's `base` layer:
```css
@layer base { h1 { font-size: var(--text-2xl); } }
```

**Adding component classes**

Use the `components` layer for any more complicated classes you want to add to your project
```css
@layer components { .card { ... } }
```
```html
<div class="card rounded-none">
```
The components layer is also a good place to put custom styles for any third-party components you're using

**Using variants**
```css
.my-element {
  background: white;
  @variant dark { background: black; }
  @variant hover:focus { background: black; }
  @variant hover, focus {
    background: black;
  }
}
/* Compiled CSS */
.my-element { ...
  @media (prefers-color-scheme: dark) { background: black; }
  
  &:hover {
    @media (hover: hover) {
      &:focus {
        background: black;
      }
    }
  }

  &:hover { @media (hover: hover) { background: black; } }
  &:focus { background: black; }
}
```

### 1.7.3. Adding custom utilities

```css
@utility content-auto { content-visibility: auto; }
@utility scrollbar-hidden { &::-webkit-scrollbar { display: none; } }
```
```html
<div class="hover:content-auto">
```

[**Functional utilities**](https://tailwindcss.com/docs/adding-custom-styles#functional-utilities)
```css
@utility tab-* { tab-size: --value(integer); }
```
...

## 1.8. Detecting classes in source files

Tailwind treats all of your source files as plain text. it just looks for any tokens in your file that could be classes based on which characters Tailwind is expecting in class names.

it has no way of understanding string concatenation or interpolation in the programming language you're using.
```html
<!-- Don't construct class names dynamically -->
<div class="text-{{ error ? 'red' : 'green' }}-600"></div>
<!-- Always use complete class names -->
<div class="{{ error ? 'text-red-600' : 'text-green-600' }}"></div>
```
```jsx
function Button({ color, children }) {
  const colorVariants = {
    blue: "bg-blue-600 hover:bg-blue-500 text-white",
    red: "bg-red-500 hover:bg-red-400 text-white",
  };
  return <button className={`${colorVariants[color]} ...`}>{children}</button>;
}
```

[**Which files are scanned**](https://tailwindcss.com/docs/detecting-classes-in-source-files#which-files-are-scanned
)

**Explicitly registering sources**
```css
@import "tailwindcss";
@source "../node_modules/@acmecorp/ui-lib";
```

**Setting your base path**

suppose your CSS file is nested inside `src/styles/index.css`. If you want tailwind to only scan within `src`, you must step up two directories using `../../src` so the compiler can correctly resolve the path to the main `src/` folder from where the stylesheet sits.
```css
@import "tailwindcss" source("../../src"); 
```

**Ignoring specific paths**
```css
@import "tailwindcss";
@source not "../../src/components/legacy";
```
This is useful when you have large directories in your project that you know don't use Tailwind classes

**Disabling automatic detection**

to completely disable automatic source detection and register all of your sources explicitly:
```css
@import "tailwindcss" source(none);
@source "../admin";
```

[**Safelisting specific utilities**](https://tailwindcss.com/docs/detecting-classes-in-source-files#safelisting-specific-utilities)

If you need to make sure Tailwind generates certain class names that don’t exist in your content files, force them to be generated

...

## 1.9. [Functions and directives](https://tailwindcss.com/docs/functions-and-directives)

### 1.9.1. Directives

...

**@apply**

Use the directive to inline any existing utility classes into your own custom CSS:
```css
.select2-dropdown {
  @apply rounded-b-lg shadow-md;
}
```

[**@reference**](https://tailwindcss.com/docs/functions-and-directives#reference-directive)

If you want to use `@apply` or `@variant` in the `<style>` block of a Vue or Svelte component, or within CSS modules, you will need to import your theme variables, custom utilities, and custom variants to make those values available in that context.

To do this without duplicating any CSS in your output, use the `@reference` directive to import your main stylesheet for reference without actually including the styles:
```css
@reference "../../app.css";
```
If you’re just using the default theme with no customizations (e.g. by using things like `@theme`, `@custom-variant`, etc…), you can import `tailwindcss` directly:
```css
@reference "tailwindcss";
```

### 1.9.2. [Functions](https://tailwindcss.com/docs/functions-and-directives#functions)

# 2. Base styles

## 2.1. [Preflight](https://tailwindcss.com/docs/preflight)