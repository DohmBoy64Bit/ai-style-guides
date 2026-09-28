# HTML Style Guide

Write HTML as an experienced professional frontend developer maintaining a real production codebase.

The goal is clean, semantic, accessible, maintainable markup—not HTML that looks generated, over-structured, excessively commented, or assembled from generic templates.

The most important rule:

> Do not optimize for demonstrating HTML knowledge. Optimize for producing the smallest semantic, accessible, production-quality markup that an experienced maintainer would reasonably write.

## General Principles

Prefer:

- Semantic HTML over generic containers.
- Native browser behavior over JavaScript recreations.
- Existing project conventions over personal preferences.
- Simple DOM structure over unnecessary wrappers.
- Clear content hierarchy.
- Accessible markup by default.
- Meaningful elements over ARIA patches.
- Stable classes and attributes tied to actual styling or behavior.
- Valid HTML.
- Minimal markup that still clearly expresses structure.

Do not add elements merely to make the document look more structured.

Do not add ARIA merely to make the markup look more accessible.

Do not add comments that simply describe visible sections.

Do not rebuild native HTML functionality with generic elements unless there is a real requirement.

---

## Match the Existing Codebase

Before editing markup, inspect nearby files and follow the repository's established conventions for:

- Indentation.
- Attribute ordering.
- Class naming.
- IDs.
- Component structure.
- CSS architecture.
- Data attributes.
- Accessibility conventions.
- JavaScript hooks.
- Template syntax.
- Framework conventions.
- Image handling.
- Icon usage.
- SEO metadata.
- Script loading.
- Form patterns.
- HTML formatting.

Consistency with the repository is more important than imposing a personal style.

Do not reformat the entire page as part of an unrelated change.

Do not convert markup architecture merely because another style seems cleaner.

---

## Use Semantic HTML

Choose elements based on meaning, not appearance.

Prefer:

```html
<header>
<nav>
<main>
<section>
<article>
<aside>
<footer>
```

when those elements accurately describe the content.

Use:

```html
<button>
<a>
<form>
<label>
<input>
<select>
<textarea>
table
ul
ol
```

instead of recreating those behaviors with generic containers.

Bad:

```html
<div class="button" onclick="submitForm()">
  Submit
</div>
```

Better:

```html
<button type="submit">
  Submit
</button>
```

Bad:

```html
<div class="nav">
  ...
</div>
```

Better:

```html
<nav>
  ...
</nav>
```

when the content is actually site navigation.

Semantic elements should reflect document meaning, not satisfy a checklist.

---

## Avoid `div` Soup

Do not wrap every element in multiple generic containers.

Bad:

```html
<div class="card-wrapper">
  <div class="card-container">
    <div class="card-inner">
      <div class="card-content">
        <h2>Weapon Details</h2>
      </div>
    </div>
  </div>
</div>
```

Prefer:

```html
<article class="weapon-card">
  <h2>Weapon Details</h2>
</article>
```

Add wrappers only when they serve a real purpose such as:

- Layout.
- Styling.
- Positioning.
- Scrolling.
- Clipping.
- JavaScript behavior.
- Semantic grouping.

Every wrapper should earn its existence.

---

## Do Not Overuse Semantic Elements

Semantic HTML is not about replacing every `div`.

A `div` is appropriate when an element exists only for layout or styling and has no meaningful semantic role.

Do not use:

```html
<section>
```

for every generic visual group.

A `section` should usually represent a distinct thematic region and generally make sense with a heading.

Bad:

```html
<section class="button-row">
  ...
</section>
```

Better:

```html
<div class="button-row">
  ...
</div>
```

when it is merely layout.

---

## Document Structure

Use a valid basic document structure when creating standalone HTML.

Example:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1"
    >
    <title>RetroStream</title>
  </head>
  <body>
    <main>
      ...
    </main>
  </body>
</html>
```

Do not add boilerplate tags without understanding their purpose.

Follow framework conventions when the framework controls the document shell.

---

## `doctype`

Use:

```html
<!doctype html>
```

for modern HTML documents.

Do not use legacy HTML or XHTML doctypes unless maintaining a legacy system that requires them.

---

## Language

Set the document language:

```html
<html lang="en">
```

Use the actual language of the document.

Do not hard-code `lang="en"` if the application dynamically serves another language.

Correct language metadata improves:

- Screen reader pronunciation.
- Search indexing.
- Browser behavior.
- Translation tools.

---

## Character Encoding

Use:

```html
<meta charset="utf-8">
```

near the beginning of `<head>`.

Do not rely on ambiguous platform defaults.

This helps prevent encoding problems such as malformed punctuation or mojibake.

---

## Viewport

For normal responsive pages, use:

```html
<meta
  name="viewport"
  content="width=device-width, initial-scale=1"
>
```

Do not disable zoom with:

```html
maximum-scale=1
user-scalable=no
```

unless there is an exceptional accessibility-tested reason.

Users should normally be allowed to zoom.

---

## Headings

Use headings to represent the document hierarchy.

Start with a meaningful page-level heading, usually:

```html
<h1>
```

Then use appropriate nested headings:

```html
<h2>
<h3>
<h4>
```

Do not choose heading levels based on font size.

Bad:

```html
<h4 class="large-title">
  Movies
</h4>
```

because the design wanted smaller margins.

Use CSS for appearance.

Use heading elements for structure.

---

## Heading Order

Keep heading hierarchy logical.

Good:

```html
<h1>Weapon Catalog</h1>

<section>
  <h2>Assault Rifles</h2>

  <article>
    <h3>M4A1</h3>
  </article>
</section>
```

Avoid jumping heading levels without a structural reason:

```html
<h1>Weapon Catalog</h1>
<h4>Assault Rifles</h4>
```

Do not add invisible headings purely to satisfy automated tooling unless they genuinely improve the document structure.

---

## One `h1`

A single primary `h1` is often the clearest structure for an application page.

Modern HTML technically permits multiple section-level headings, but do not add multiple `h1` elements merely because it is technically valid.

Follow the application's existing document hierarchy.

---

## Paragraphs

Use `<p>` for actual paragraphs of prose.

Do not use:

```html
<div class="description">
```

when the content is plainly a paragraph.

Prefer:

```html
<p class="description">
```

Do not wrap unrelated controls or layout containers in `<p>`.

---

## Lists

Use lists when the content is conceptually a list.

Good:

```html
<ul>
  <li>Rifle</li>
  <li>SMG</li>
  <li>Shotgun</li>
</ul>
```

Navigation menus are often naturally lists:

```html
<nav aria-label="Primary">
  <ul>
    <li><a href="/">Home</a></li>
    <li><a href="/movies">Movies</a></li>
  </ul>
</nav>
```

However, do not force every repeated layout into a list if the semantics do not benefit from it.

---

## Links vs Buttons

Use links for navigation.

Use buttons for actions.

Link:

```html
<a href="/movies">
  Browse movies
</a>
```

Button:

```html
<button type="button">
  Add to favorites
</button>
```

Do not use a button for normal page navigation.

Do not use an anchor without an `href` merely to make it act like a button.

Bad:

```html
<a onclick="openModal()">
  Open modal
</a>
```

Better:

```html
<button type="button" id="open-modal">
  Open modal
</button>
```

---

## Buttons

Always consider button type.

Inside forms, use:

```html
<button type="submit">
```

for submission.

Use:

```html
<button type="button">
```

for non-submit actions.

Do not rely on the default `submit` behavior accidentally.

This prevents subtle form bugs.

---

## Avoid Clickable `div`s

Do not create interactive elements like:

```html
<div
  class="play-button"
  tabindex="0"
  role="button"
>
  Play
</div>
```

when:

```html
<button
  type="button"
  class="play-button"
>
  Play
</button>
```

already provides:

- Keyboard activation.
- Focus behavior.
- Accessibility semantics.
- Disabled state.
- Form integration.

Use native controls first.

---

## Forms

Use native form semantics.

Example:

```html
<form>
  <label for="search">
    Search
  </label>

  <input
    id="search"
    name="query"
    type="search"
  >

  <button type="submit">
    Search
  </button>
</form>
```

Do not recreate form semantics through JavaScript alone.

---

## Labels

Every form control requiring a label should have a meaningful accessible label.

Preferred:

```html
<label for="email">
  Email
</label>

<input
  id="email"
  name="email"
  type="email"
>
```

A wrapping label is also valid:

```html
<label>
  Email
  <input
    name="email"
    type="email"
  >
</label>
```

Do not rely solely on placeholders as labels.

Bad:

```html
<input
  type="email"
  placeholder="Email"
>
```

A placeholder is not a replacement for a label.

---

## Visually Hidden Labels

When the visual design does not need a visible label, use the project's established visually-hidden utility.

Example:

```html
<label
  for="search"
  class="sr-only"
>
  Search movies
</label>
```

Do not omit accessible names merely for cleaner visual design.

---

## Input Types

Use the correct native input type.

Examples:

```html
<input type="email">
<input type="search">
<input type="number">
<input type="url">
<input type="tel">
<input type="date">
```

This improves:

- Mobile keyboards.
- Built-in validation.
- Autofill.
- Accessibility.

Do not use `type="text"` for every input automatically.

---

## Input Names

Forms intended for native submission or integration should use meaningful `name` attributes.

Example:

```html
<input
  id="query"
  name="query"
  type="search"
>
```

Do not use meaningless names such as:

```html
name="input1"
```

when the field has a clear domain meaning.

---

## Autocomplete

Use appropriate `autocomplete` attributes for common user data.

Example:

```html
<input
  name="email"
  type="email"
  autocomplete="email"
>
```

Do not disable autocomplete across all forms without a strong reason.

---

## Required Fields

Use native:

```html
required
```

when a field is actually required.

Do not add:

```html
aria-required="true"
```

if the native `required` attribute already provides the necessary semantic behavior unless a compatibility reason exists.

Prefer native semantics.

---

## Validation

Use native validation attributes when appropriate:

```html
required
min
max
minlength
maxlength
pattern
```

Do not duplicate every native validation rule through JavaScript without need.

JavaScript validation can supplement browser validation when domain rules require it.

Server-side validation is still necessary for security and correctness.

---

## Fieldsets

Use:

```html
<fieldset>
  <legend>Notification preferences</legend>
  ...
</fieldset>
```

for meaningful groups of related form controls, especially radio buttons and checkboxes.

Do not use fieldsets merely as generic visual containers.

---

## Placeholder Text

Placeholder text should be a short hint, not instructions or a permanent label.

Good:

```html
placeholder="Search movies and shows"
```

Avoid long instructions inside placeholders.

Do not put critical information where it disappears when typing begins.

---

## Disabled vs Readonly

Use:

```html
disabled
```

when the control is unavailable and should not participate normally in interaction or form submission.

Use:

```html
readonly
```

when users may view and focus the value but not modify it.

Do not use them interchangeably.

---

## Tables

Use `<table>` for tabular data.

Do not use tables for layout.

Good:

```html
<table>
  <thead>
    <tr>
      <th scope="col">Weapon</th>
      <th scope="col">Damage</th>
      <th scope="col">Fire Rate</th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <th scope="row">M4A1</th>
      <td>32</td>
      <td>800 RPM</td>
    </tr>
  </tbody>
</table>
```

Use appropriate:

```html
caption
thead
tbody
th
scope
```

when they improve structure and accessibility.

Do not add every possible table element mechanically.

---

## Images

Use meaningful `alt` text.

Good:

```html
<img
  src="/images/m4a1.webp"
  alt="M4A1 assault rifle"
>
```

Decorative images should generally use:

```html
alt=""
```

Do not repeat surrounding text unnecessarily.

Bad:

```html
<img
  src="/poster.jpg"
  alt="Image"
>
```

Bad:

```html
<img
  src="/poster.jpg"
  alt="Movie poster image picture"
>
```

Describe the information the image contributes.

---

## Decorative Images

For purely decorative images:

```html
<img
  src="/decoration.svg"
  alt=""
>
```

Do not invent descriptive alt text for decoration that adds no useful information.

If an image is applied via CSS purely for decoration, it generally does not need accessibility text at all.

---

## Image Dimensions

Where possible, provide intrinsic dimensions:

```html
<img
  src="/poster.webp"
  alt="..."
  width="342"
  height="513"
>
```

This helps prevent layout shift.

Do not invent incorrect dimensions merely to satisfy tooling.

---

## Lazy Loading

Use:

```html
loading="lazy"
```

for images below the fold where appropriate.

Do not lazy-load the primary above-the-fold hero or important LCP image automatically.

The most important image may need eager loading or higher fetch priority depending on the application.

Performance attributes should reflect actual loading priority.

---

## Responsive Images

Use:

```html
srcset
sizes
picture
source
```

when different image sizes or formats provide meaningful performance benefits.

Do not produce complicated responsive image markup without corresponding image variants.

---

## `<picture>`

Use `<picture>` when you actually need:

- Art direction.
- Alternate formats.
- Different sources at breakpoints.

Do not wrap every image in `<picture>` by default.

---

## Figures

Use:

```html
<figure>
  <img ...>
  <figcaption>...</figcaption>
</figure>
```

when an image and caption form one meaningful piece of content.

Do not use `figure` merely as a generic image wrapper.

---

## Icons

Use the project's established icon system.

For decorative SVG icons, hide them from accessibility when the surrounding control already has a name:

```html
<button type="button">
  <svg
    aria-hidden="true"
    ...
  ></svg>
  Add to favorites
</button>
```

Do not make both the button text and icon announce the same thing.

---

## Icon-Only Buttons

Icon-only buttons need an accessible name.

Example:

```html
<button
  type="button"
  aria-label="Close dialog"
>
  <svg
    aria-hidden="true"
    ...
  ></svg>
</button>
```

Do not assume the SVG path itself provides a useful accessible name.

---

## SVG

Inline SVG is appropriate when:

- The icon must inherit color.
- The markup is small.
- The project uses inline icons.
- Styling or interaction requires it.

Do not paste enormous SVG documents inline unnecessarily.

Do not leave editor metadata or useless generated attributes in SVG markup.

---

## Accessibility First

Prefer correct native HTML before ARIA.

The first rule of ARIA is effectively:

> If native HTML already provides the correct semantics and behavior, use it instead.

Do not turn:

```html
<button>
```

into:

```html
<div
  role="button"
  tabindex="0"
  aria-pressed="false"
>
```

unless a real constraint prevents using a native button.

---

## Avoid Redundant ARIA

Do not write:

```html
<button role="button">
```

because `<button>` already has button semantics.

Do not write:

```html
<nav role="navigation">
```

unless required by an unusual compatibility target.

Do not write:

```html
<input
  type="checkbox"
  role="checkbox"
>
```

Native semantics are preferable.

Redundant ARIA adds noise and can create errors.

---

## ARIA Labels

Use `aria-label` only when visible text cannot provide an accessible name.

Bad:

```html
<button aria-label="Submit form">
  Submit form
</button>
```

when the visible text already communicates the name.

Better:

```html
<button type="submit">
  Submit form
</button>
```

Use `aria-label` for cases such as icon-only controls.

---

## `aria-labelledby`

Use `aria-labelledby` when an existing visible element should provide the accessible name.

Example:

```html
<section
  aria-labelledby="popular-heading"
>
  <h2 id="popular-heading">
    Popular Movies
  </h2>
</section>
```

Do not add this mechanically to every section.

If the native element and surrounding structure already communicate the relationship sufficiently, extra ARIA may not help.

---

## `aria-describedby`

Use `aria-describedby` for supplementary descriptions.

Example:

```html
<input
  id="password"
  aria-describedby="password-help"
>

<p id="password-help">
  Use at least 12 characters.
</p>
```

Do not use it to duplicate labels.

---

## ARIA State

When custom controls genuinely require ARIA state, keep it synchronized with actual behavior.

Examples:

```html
aria-expanded
aria-selected
aria-pressed
aria-current
aria-invalid
```

Do not add static ARIA attributes that become incorrect after interaction.

Incorrect ARIA can be worse than no ARIA.

---

## `aria-hidden`

Use:

```html
aria-hidden="true"
```

only when content should be excluded from the accessibility tree.

Do not apply it to:

- Focusable controls.
- Important content.
- Containers containing active interactive elements.

Decorative icons are common appropriate uses.

---

## `tabindex`

Do not add:

```html
tabindex="0"
```

to elements already naturally focusable.

Do not create arbitrary positive tab order:

```html
tabindex="1"
tabindex="2"
```

Positive tabindex values create fragile navigation.

Use:

```html
tabindex="-1"
```

when programmatic focus is needed without placing the element in normal tab order.

---

## Keyboard Accessibility

Interactive controls should work naturally with a keyboard.

Using native elements often solves this automatically.

Do not manually implement Enter and Space behavior for a fake button if a real `<button>` works.

Avoid mouse-only interaction.

---

## Focus

Do not remove visible focus outlines globally.

Bad:

```css
*:focus {
  outline: none;
}
```

If custom focus styling is used, ensure it remains clearly visible.

Do not manipulate focus unless interaction design actually requires it.

---

## Skip Links

For larger pages with repeated navigation, consider a skip link where appropriate:

```html
<a
  class="skip-link"
  href="#main-content"
>
  Skip to main content
</a>
```

Do not add one mechanically to tiny pages where it adds little value.

---

## Landmarks

Use meaningful landmarks:

```html
<header>
<nav>
<main>
<aside>
<footer>
```

Do not add unnecessary `role` attributes to native landmarks.

Avoid creating several unlabeled navigation landmarks when users cannot distinguish them.

Use labels when multiple nav regions exist:

```html
<nav aria-label="Primary">
```

and:

```html
<nav aria-label="Footer">
```

---

## Main Content

Normally use one primary:

```html
<main>
```

per page.

Do not use `<main>` for repeated widgets or nested content sections.

---

## Header and Footer

`<header>` and `<footer>` can belong to the whole page or to sectional content.

Use them where they represent introductory or concluding content.

Do not use them simply because an area appears at the top or bottom visually.

---

## Sections

Use `<section>` for a thematic grouping.

A section should usually have a meaningful heading.

Good:

```html
<section>
  <h2>Popular Movies</h2>
  ...
</section>
```

Avoid:

```html
<section class="spacing-wrapper">
```

when the element exists only to provide padding.

Use a `div` for layout-only containers.

---

## Articles

Use `<article>` when the content is independently meaningful or reusable.

Examples:

- Blog posts.
- News stories.
- Product cards.
- Forum posts.
- Movie entries.
- Search results when each item represents independent content.

Do not use `<article>` merely because something appears inside a card.

---

## Aside

Use `<aside>` for content tangentially related to the surrounding content.

Examples:

- Related links.
- Supporting information.
- Sidebars.

Do not use `<aside>` solely because something is visually placed on the side.

---

## Navigation

Use `<nav>` for major navigation groups.

Not every set of links needs a `<nav>`.

Good uses include:

- Primary site navigation.
- Pagination.
- Table of contents.
- Secondary navigation.

A single inline link usually does not need a nav container.

---

## Breadcrumbs

When breadcrumbs exist, structure them clearly.

Example:

```html
<nav aria-label="Breadcrumb">
  <ol>
    <li><a href="/">Home</a></li>
    <li><a href="/movies">Movies</a></li>
    <li aria-current="page">
      Alien
    </li>
  </ol>
</nav>
```

Do not use visual separators as meaningful text when CSS can provide them decoratively.

---

## Dialogs

Use the native `<dialog>` element when supported by the project's target environments and appropriate to the interaction.

Example:

```html
<dialog id="settings-dialog">
  ...
</dialog>
```

Do not build a generic modal from nested `<div>` elements unless compatibility or framework constraints require it.

When implementing custom dialogs, correctly handle:

- Focus.
- Escape.
- Background interaction.
- Accessible naming.
- Focus restoration.

Do not treat modal accessibility as a few ARIA attributes.

---

## Details and Summary

Use:

```html
<details>
  <summary>Advanced options</summary>
  ...
</details>
```

for simple disclosure interfaces.

Do not build JavaScript accordions for simple cases when native disclosure behavior satisfies the design.

---

## Progress

Use native elements when their semantics fit:

```html
<progress
  value="60"
  max="100"
>
  60%
</progress>
```

Do not create a generic progress bar with a `div` unless custom behavior requires it.

---

## Meter

Use `<meter>` for measurements within a known range when appropriate.

Do not use it interchangeably with `<progress>`.

Progress represents completion.

Meter represents a scalar measurement.

---

## Time

Use:

```html
<time datetime="2026-09-28">
  September 28, 2026
</time>
```

when machine-readable date or time information is useful.

Do not wrap every casual time reference in `<time>` mechanically.

---

## Abbreviations

Use `<abbr>` when the abbreviation needs clarification.

Do not add `title` attributes to every common abbreviation automatically.

---

## Address

Use `<address>` for contact information associated with the nearest article or document.

Do not use it merely to display any street address.

---

## Strong and Emphasis

Use:

```html
<strong>
```

for strong importance.

Use:

```html
<em>
```

for stress emphasis.

Do not use `<b>` and `<i>` merely as styling hooks when semantic emphasis is intended.

However, `<b>` and `<i>` are legitimate when the semantics fit their modern definitions.

Use CSS for purely visual weight or style.

---

## Line Breaks

Do not use repeated:

```html
<br>
<br>
<br>
```

for layout spacing.

Use CSS.

Use `<br>` only where a line break is semantically part of the content, such as addresses or poetry.

---

## Horizontal Rules

Use:

```html
<hr>
```

for a thematic break.

Do not use it as a generic decorative line when CSS borders are more appropriate.

---

## IDs

Use IDs when:

- Linking to a fragment.
- Connecting labels and form controls.
- Connecting ARIA relationships.
- A JavaScript API genuinely requires a unique target.

Do not assign IDs to every element.

Avoid meaningless IDs:

```html
id="div1"
id="container2"
id="section3"
```

Prefer:

```html
id="search"
id="main-content"
id="popular-heading"
```

---

## ID Uniqueness

IDs must be unique within the document.

Do not duplicate IDs across repeated cards or components.

For repeated elements, use classes or data attributes.

---

## Classes

Class names should reflect the project's styling convention.

Examples may include:

```text
BEM
utility classes
Tailwind
component classes
CSS modules
design-system tokens
```

Follow the existing architecture.

Do not introduce a second naming system.

---

## Avoid Meaningless Class Names

Avoid generated names like:

```html
<div class="container-wrapper-inner-content">
```

or:

```html
<div class="section1">
```

unless generated by a framework or CSS module system.

Use names tied to actual component or layout meaning.

---

## Avoid Style-Only Semantic Naming

Avoid names such as:

```text
red-text
left-box
big-heading
```

when the design may change.

Prefer domain or role names:

```text
error-message
sidebar
page-title
```

unless using intentional utility classes.

---

## Utility Classes

If the project uses Tailwind or another utility framework, use utilities directly and consistently.

Do not create custom CSS classes merely to avoid writing a reasonable number of utility classes unless reuse or readability justifies it.

Do not mix arbitrary inline styles, utility classes, and component CSS without a reason.

---

## Data Attributes

Use `data-*` attributes for behavior hooks or metadata that belongs in the DOM.

Example:

```html
<button
  type="button"
  data-remove-id="weapon-42"
>
  Remove
</button>
```

Do not overload CSS classes purely as JavaScript selectors if the project has established data attributes for behavior.

Keep styling and behavior hooks distinct when useful.

---

## Inline Styles

Avoid inline styles in maintainable application code when the project has a normal styling system.

Bad:

```html
<div
  style="margin-top: 20px; color: red;"
>
```

Prefer the project's CSS or utility classes.

Inline styles can be appropriate for truly dynamic values supplied by application logic.

Follow the existing framework.

---

## Inline Event Handlers

Avoid:

```html
<button onclick="openModal()">
```

in modern application code unless maintaining a codebase that intentionally uses inline handlers.

Prefer:

```html
<button
  type="button"
  id="open-modal"
>
```

with JavaScript registering the listener separately.

This improves separation of concerns and avoids global-function coupling.

---

## Scripts

Load scripts according to the project architecture.

For module-based JavaScript:

```html
<script
  type="module"
  src="/src/main.js"
></script>
```

Do not add `async`, `defer`, and `type="module"` mechanically without understanding their loading semantics.

Module scripts are deferred by default.

---

## Script Placement

Do not rely on placing scripts at the bottom of `<body>` merely as a universal performance rule.

Modern `defer` and module loading often make script placement less important.

Follow the project's bundler and runtime conventions.

---

## `async` vs `defer`

Use `defer` for scripts that depend on document parsing and should preserve order.

Use `async` for independent scripts that can execute as soon as they load.

Do not use both without understanding the resulting behavior.

Third-party analytics may be appropriate for `async`.

Application scripts commonly use modules or `defer`.

---

## Stylesheets

Use normal stylesheet links:

```html
<link
  rel="stylesheet"
  href="/styles.css"
>
```

Do not duplicate stylesheets.

Do not load the same framework from both a CDN and a bundled dependency.

---

## Preload

Use preload only for genuinely critical resources.

Do not preload everything.

Excessive preloads compete for bandwidth and can make performance worse.

Use:

```html
<link rel="preload">
```

only when the browser would otherwise discover an important resource too late.

---

## Preconnect

Use preconnect sparingly for origins the page will actually contact early.

Do not add many speculative preconnects.

Every connection consumes resources.

---

## Fonts

Use the project's font-loading strategy.

Avoid blocking page rendering with unnecessary font resources.

When custom fonts are used, consider:

- `font-display`.
- Preloading only critical fonts.
- Limiting weight variants.
- Local fallbacks.

Do not include ten font weights if only two are used.

---

## Favicons

Use appropriate favicon and application-icon metadata when relevant.

Do not generate excessive legacy icon declarations unless the supported platforms require them.

Follow deployment requirements.

---

## Metadata

Include meaningful metadata for production pages when appropriate:

```html
<title>
<meta name="description">
```

Do not add huge collections of SEO tags merely to make the `<head>` look complete.

Metadata should correspond to actual content and product needs.

---

## Title

Every standalone page should have a meaningful `<title>`.

Avoid:

```html
<title>Home</title>
```

when the application or site name matters.

Prefer meaningful context, for example:

```html
<title>Popular Movies | RetroStream</title>
```

Follow the site's title convention.

---

## Meta Description

Use a concise page-specific description when SEO matters.

Do not duplicate the same generic description across every page.

Do not stuff keywords.

Write for users first.

---

## Canonical URLs

Use canonical metadata only when the application actually needs canonicalization.

Do not point every page to the homepage.

A page-specific canonical should correspond to the preferred URL for that content.

Incorrect canonicals can harm indexing.

---

## Robots Metadata

Do not add:

```html
<meta
  name="robots"
  content="noindex"
>
```

without a deliberate reason.

Likewise, do not remove indexing restrictions from private, preview, account, or duplicate pages without understanding the deployment model.

SEO directives are behavioral configuration, not decoration.

---

## Open Graph Metadata

Add Open Graph metadata when social sharing matters.

Do not mechanically add fake placeholder values.

Metadata should reflect real page content.

---

## Structured Data

Use schema.org structured data only when it accurately represents visible content.

Do not manufacture ratings, reviews, prices, organizations, or other values merely to improve SEO.

Structured data should be truthful and consistent with the page.

---

## Hidden Content

Do not hide meaningful SEO text solely to influence search engines.

Do not create invisible keyword blocks.

Hidden content should exist for legitimate UI or accessibility behavior.

---

## SEO and Semantic Markup

Good SEO usually follows from:

- Meaningful content.
- Proper headings.
- Descriptive titles.
- Useful links.
- Crawlable navigation.
- Correct canonicalization.
- Image alt text.
- Semantic document structure.
- Fast loading.

Do not treat HTML as a collection of SEO tricks.

---

## Link Text

Use descriptive link text.

Good:

```html
<a href="/movies/alien">
  View Alien details
</a>
```

Avoid repeated generic links such as:

```html
<a href="...">
  Click here
</a>
```

when surrounding context does not provide enough meaning.

---

## External Links

Do not add:

```html
target="_blank"
```

to every external link automatically.

Opening new windows changes user behavior and should be intentional.

When `target="_blank"` is used, modern browsers generally provide appropriate isolation, but follow the project's security conventions for `rel`.

---

## Download Links

Use:

```html
<a
  href="/files/report.pdf"
  download
>
  Download report
</a>
```

when the browser should treat the target as a download and same-origin/security rules permit it.

Do not add `download` to normal navigation links.

---

## Empty Links

Avoid:

```html
<a href="#">
```

as a placeholder action.

This changes the URL and scroll behavior.

Use a button for actions.

Use a real destination for navigation.

---

## Placeholder URLs

Do not leave:

```html
href="javascript:void(0)"
```

in production markup.

Use correct semantics instead.

---

## Responsive Markup

Keep HTML independent of specific screen sizes where possible.

Do not duplicate the entire DOM for mobile and desktop unless requirements genuinely demand different structures.

Prefer responsive CSS.

Duplicated markup can create:

- Accessibility duplication.
- State synchronization bugs.
- Larger DOMs.
- Maintenance problems.

---

## Source Order

Keep DOM source order logical.

Do not rely on CSS visual reordering to make semantically unrelated content appear in a different sequence.

Keyboard and screen reader order generally follows DOM order.

Design the markup order intentionally.

---

## Mobile

Do not create separate mobile-only semantic structures unless necessary.

Use:

- Responsive layout.
- Flexible containers.
- Appropriate viewport settings.
- Native controls.
- Touch-friendly sizes.

Do not solve responsive problems with excessive JavaScript when CSS can handle them.

---

## Touch Targets

Interactive elements should have reasonable touch target sizes.

Do not solve small touch targets by wrapping buttons in fake clickable containers.

Style the actual control.

---

## Content Visibility

Do not hide critical content on smaller screens merely because layout is difficult.

Adapt the layout instead.

Responsive design should preserve functionality.

---

## Progressive Enhancement

Whenever practical, start from functioning HTML and enhance with JavaScript.

Examples:

- Forms should use actual forms.
- Links should have real destinations.
- Buttons should be buttons.
- Native disclosure controls can work without JavaScript.

Do not require JavaScript for basic semantics without a reason.

---

## No-JavaScript Behavior

Not every application needs full functionality without JavaScript.

However, markup should remain coherent and semantically correct even when script-driven enhancements are unavailable.

Do not add `<noscript>` warnings automatically unless JavaScript is genuinely required and the message provides useful information.

---

## Content Before Behavior

Write markup based on content and interaction semantics first.

Do not design the DOM solely around JavaScript selectors.

Bad:

```html
<div
  class="js-click-handler-wrapper-2"
>
```

Prefer meaningful structure with behavior hooks added as needed.

---

## Template Markup

When using templating systems, keep logic out of HTML where practical.

Avoid deeply nested template conditions that make the DOM impossible to understand.

Bad conceptual structure:

```text
if
  for
    if
      if
        markup
```

Extract complex rendering logic into appropriate template helpers or application code when the framework supports it.

---

## Do Not Duplicate Business Logic in HTML

Templates should generally display values and simple conditions.

Do not reproduce application rules in data attributes, hidden inputs, inline scripts, and visible markup separately.

Keep one source of truth.

---

## Comments

Use HTML comments sparingly.

Do not write:

```html
<!-- Header Section -->
<header>
```

or:

```html
<!-- Start Movies -->
<section>
```

when the markup already makes the structure obvious.

Comments should explain non-obvious requirements.

Example:

```html
<!-- Kept outside the transformed parent because position: fixed breaks inside it. -->
```

That explains why something unusual exists.

---

## Avoid Decorative Comments

Do not add banners like:

```html
<!-- ========================= -->
<!--        NAVIGATION         -->
<!-- ========================= -->
```

unless the repository intentionally uses that convention.

These create noise and quickly become stale.

---

## Avoid AI-Looking Comments

Avoid phrases like:

```text
This section is responsible for...
This container ensures...
The following markup provides...
This creates a robust and accessible...
```

Write concise technical comments only when needed.

---

## Avoid Generated Placeholder Content

Do not leave:

```text
Lorem ipsum
Example text
Placeholder item
TODO content
```

in completed production work unless placeholders were explicitly requested.

Use real content provided by the application or user.

---

## Avoid Unnecessary Attributes

Do not write:

```html
<input
  type="text"
  value=""
>
```

when the empty value adds nothing.

Do not add:

```html
autocomplete="off"
spellcheck="false"
draggable="false"
```

to every element mechanically.

Every attribute should have a reason.

---

## Boolean Attributes

Use boolean HTML attributes idiomatically.

Prefer:

```html
<input disabled>
```

rather than:

```html
<input disabled="disabled">
```

unless project formatting conventions dictate otherwise.

Similarly:

```html
required
checked
readonly
multiple
autofocus
```

do not require redundant values.

---

## Attribute Quotes

Use quoted attribute values consistently.

Example:

```html
class="movie-card"
```

Even though HTML allows some unquoted values, quoted attributes are generally easier to maintain.

Follow formatter conventions.

---

## Attribute Ordering

Follow the project's formatter or convention.

Do not constantly reorder attributes in unrelated code.

A common useful grouping may be:

1. Identity and semantics.
2. URLs.
3. Classes.
4. Data attributes.
5. Accessibility.
6. Behavior.

But consistency with the repository matters more than a universal ordering rule.

---

## Empty Elements

Use standard HTML syntax.

Prefer:

```html
<img ...>
<input ...>
<meta ...>
<link ...>
```

Do not use XHTML-style self-closing syntax:

```html
<img ... />
```

unless the project's framework or formatter intentionally outputs it.

In JSX, self-closing syntax is appropriate because JSX is not plain HTML.

---

## Invalid Nesting

Respect HTML content models.

Do not place interactive elements inside other interactive elements.

Bad:

```html
<a href="/movie">
  <button type="button">
    Watch
  </button>
</a>
```

Use one interactive element or restructure the card.

Avoid invalid nesting such as:

```html
<p>
  <div>...</div>
</p>
```

Browsers may silently repair invalid markup in unexpected ways.

---

## Nested Interactive Controls

Do not place:

- Buttons inside buttons.
- Links inside links.
- Buttons inside links.
- Clickable controls inside another control.

Design card interactions carefully.

For example, a card can contain a normal linked title plus separate action buttons instead of making the entire card one giant link around everything.

---

## Cards

Do not over-semanticize cards.

A card might be:

```html
<article>
```

if it represents independent content.

It might simply be:

```html
<div>
```

if it is a visual grouping.

Do not create custom wrappers such as:

```html
<card-container>
```

unless using actual web components.

---

## Web Components

Use custom elements only when the application intentionally uses Web Components.

Custom element names must contain a hyphen:

```html
<weapon-card>
```

Do not invent custom tags in plain HTML merely because they read nicely.

Unknown elements do not automatically provide useful semantics.

---

## Custom Elements and Accessibility

Custom elements still need correct accessibility behavior.

Do not assume:

```html
<custom-button>
```

behaves like:

```html
<button>
```

It does not automatically receive native keyboard or accessibility semantics.

Use native controls inside custom components where practical.

---

## SEO Content Duplication

Avoid rendering duplicate hidden copies of content for SEO.

The visible user experience and searchable content should generally match.

Do not maintain separate "SEO text" disconnected from the actual page.

---

## Performance

Keep the DOM reasonably small.

Do not create wrappers or duplicate content without need.

Large DOMs increase:

- Memory usage.
- Style calculation cost.
- Layout complexity.
- Event-handling overhead.
- Accessibility tree complexity.

HTML performance often begins with fewer unnecessary nodes.

---

## Above-the-Fold Content

Prioritize critical content and resources.

Do not lazy-render everything.

Do not load huge unrelated components before primary content when the application's architecture allows better prioritization.

Markup should support sensible rendering order.

---

## CSS Hooks

Use the minimum DOM structure needed for styling.

Do not add a wrapper every time CSS feels inconvenient.

Modern CSS provides:

```text
flexbox
grid
gap
:has()
pseudo-elements
container queries
```

depending on target support.

Before adding extra markup solely for styling, consider whether CSS can solve it cleanly.

---

## Pseudo-Elements

Decorative elements may often be better implemented with CSS:

```css
::before
::after
```

rather than empty HTML:

```html
<span class="decorative-line"></span>
```

Use HTML when the element has actual semantic or interactive meaning.

---

## Hidden Elements

Use the correct hiding mechanism.

`hidden` removes content from normal rendering:

```html
<div hidden>
```

CSS may be appropriate for responsive visibility.

A visually hidden utility is different: it hides visually while keeping content available to assistive technology.

Do not treat these mechanisms as interchangeable.

---

## `display: none`

Content with `display: none` is generally removed from the accessibility tree.

Do not use it when content must remain available to screen readers.

Use the project's visually-hidden pattern instead.

---

## `hidden`

Use native:

```html
hidden
```

for simple programmatically toggled visibility when appropriate.

Do not invent classes like:

```html
class="is-hidden display-none hidden-element"
```

unless the CSS architecture requires them.

---

## IDs for JavaScript

If the project prefers data attributes for behavior:

```html
data-action="play"
```

do not introduce many IDs just to query elements.

Likewise, if stable IDs are already the convention, follow that.

Do not invent a new selector architecture in one feature.

---

## JavaScript Hooks

Avoid styling classes doubling as fragile behavior hooks when possible.

Example:

```html
<button
  class="button button-primary"
  data-action="play"
>
  Play
</button>
```

This allows CSS classes to change without breaking behavior.

Use this pattern when it fits the codebase.

---

## URLs

Use valid URLs.

Do not leave placeholder values such as:

```html
href="#"
src=""
```

in production markup.

An empty image `src` can trigger unwanted requests in some environments.

Use an actual URL or conditionally render the element.

---

## Relative URLs

Follow the project's deployment model.

Do not change:

```html
/assets/logo.svg
```

to:

```html
./assets/logo.svg
```

without understanding base paths, routing, and bundler behavior.

Deployment paths matter.

---

## Base Element

Use `<base>` only when the application intentionally needs to redefine relative URL resolution.

It affects every relative link and resource in the document.

Do not add it casually.

---

## Security

Do not insert untrusted HTML directly.

Avoid inline scripts when Content Security Policy or application architecture discourages them.

Do not expose secrets in:

```html
data-*
meta
hidden inputs
HTML comments
inline scripts
```

Anything delivered to the browser is visible to users.

---

## Hidden Inputs

Hidden inputs are not secure storage.

Do not place secrets in:

```html
<input type="hidden">
```

Hidden only means visually hidden.

Users can inspect and modify the value.

---

## Data Attributes

Similarly, do not store secrets or sensitive authorization data in:

```html
data-token="..."
```

Anything in the DOM is available to client-side inspection.

---

## Content Security Policy

Do not work around CSP by adding unsafe inline scripts or styles without understanding the application's security model.

Avoid suggesting:

```text
unsafe-inline
unsafe-eval
```

as a casual fix.

Follow the deployment's security requirements.

---

## External Scripts

Be careful with third-party scripts.

They can affect:

- Privacy.
- Performance.
- Security.
- Page stability.

Do not add third-party libraries or widgets solely for functionality that the project already provides.

---

## Integrity Attributes

For externally hosted resources, use Subresource Integrity when the deployment model supports and benefits from it.

Do not invent integrity hashes.

They must match the exact resource bytes.

---

## Forms and Security

Never rely on HTML validation for security.

Attributes such as:

```html
required
pattern
min
max
```

improve UX but can be bypassed.

Servers must validate untrusted input independently.

---

## Autofocus

Use:

```html
autofocus
```

sparingly.

Automatic focus can be disruptive for:

- Screen reader users.
- Mobile keyboards.
- Keyboard navigation.

Only use it when the interaction clearly benefits.

---

## Autoplay

Avoid media autoplay, especially with sound.

If autoplay is necessary, browsers commonly require:

```html
muted
```

but design decisions should consider user control and accessibility.

Do not add autoplay merely to make a page feel dynamic.

---

## Video

Use appropriate video markup:

```html
<video
  controls
  preload="metadata"
>
  <source
    src="/video.webm"
    type="video/webm"
  >
</video>
```

Provide captions when content requires them.

Do not hide native controls unless replacing them with fully accessible alternatives.

---

## Audio

Use native controls unless custom controls provide equivalent accessibility.

Provide transcripts when appropriate to the product and content.

---

## Iframes

Give meaningful iframe titles:

```html
<iframe
  src="..."
  title="Movie player"
></iframe>
```

This helps assistive technology identify the embedded content.

Do not use generic titles such as:

```text
iframe
embedded content
frame
```

when the purpose is known.

---

## Iframe Security

Use sandbox restrictions where appropriate for untrusted or third-party content.

Do not blindly apply every sandbox token.

Permissions should reflect the embedded application's actual needs.

---

## Lazy Loading Iframes

Use:

```html
loading="lazy"
```

for below-the-fold embeds where appropriate.

Do not lazy-load an immediately needed primary player purely because the attribute exists.

---

## CSP and Inline Markup

Do not inject large inline script or style blocks into pages merely to avoid creating proper assets.

Follow the bundler and deployment conventions.

Small inline critical styles may be legitimate when intentionally implemented.

---

## Microdata and Structured Attributes

Do not add:

```html
itemscope
itemtype
itemprop
```

unless the application intentionally uses microdata.

JSON-LD may be a cleaner approach for structured data in many sites.

Do not mix multiple structured-data systems accidentally.

---

## Localization

Do not bake sentence fragments into markup in ways that make translation difficult.

Avoid constructing text like:

```html
<span>Showing</span>
<span>10</span>
<span>results</span>
```

when the full sentence may need different ordering in another language.

Follow the application's localization system.

---

## Directionality

For multilingual content, respect `dir` and language requirements.

Do not hard-code assumptions about left-to-right layout into semantic markup.

Use CSS logical properties where relevant.

---

## Dates and Numbers

When content is generated dynamically, do not hard-code locale-specific formatting assumptions into HTML templates if the application has a localization layer.

Use machine-readable attributes where helpful.

---

## Whitespace

Use the project's formatter.

Do not manually align attributes into large columns.

Avoid excessive blank lines between every element.

Readable HTML is structured but compact.

---

## Indentation

Indent nested content consistently.

Example:

```html
<section>
  <h2>Popular Movies</h2>

  <div class="movie-grid">
    <article class="movie-card">
      ...
    </article>
  </div>
</section>
```

Do not combine inconsistent tabs and spaces.

Let the project formatter decide when available.

---

## Long Attributes

Wrap long elements according to the project's formatter.

For example:

```html
<input
  id="search"
  class="search-input"
  name="query"
  type="search"
  autocomplete="off"
  placeholder="Search movies and shows"
>
```

Do not force every two-attribute element onto seven lines if the repository prefers compact markup.

Consistency matters.

---

## Avoid Excessive Empty Lines

Bad:

```html
<div>

  <h2>Movies</h2>


  <p>Description</p>

</div>
```

Prefer normal grouping.

Whitespace should reflect structure, not inflate the file.

---

## Avoid Minified Source HTML

Do not write maintainable source markup as one enormous line.

Minification belongs in the build pipeline.

Source should remain readable.

---

## Generated HTML

If HTML is produced by a template engine, do not manually optimize generated output at the expense of source clarity unless output size or rendering behavior genuinely matters.

Focus on the source developers maintain.

---

## Validation

When practical, ensure markup is structurally valid.

Look for:

- Duplicate IDs.
- Unclosed tags.
- Invalid nesting.
- Missing labels.
- Missing image alt text.
- Empty links.
- Incorrect form associations.
- Broken references.
- Duplicate attributes.

Do not blindly modify valid unconventional markup solely to satisfy automated rules without understanding context.

---

## Accessibility Testing

Automated accessibility tools are useful but incomplete.

Do not treat "zero automated violations" as proof that a page is accessible.

Review:

- Keyboard navigation.
- Focus order.
- Screen reader labels.
- Dynamic state.
- Contrast.
- Zoom.
- Touch behavior.
- Dialog behavior.

HTML structure should support those interactions naturally.

---

## Avoid Accessibility Theater

Do not add dozens of ARIA attributes simply to make the markup appear accessible.

This is worse:

```html
<div
  role="button"
  tabindex="0"
  aria-label="Submit"
  aria-roledescription="button"
>
```

than simply:

```html
<button type="submit">
  Submit
</button>
```

Correct accessibility is often simpler HTML.

---

## Avoid SEO Theater

Do not add huge blocks of:

- Keywords.
- Hidden headings.
- Duplicate descriptions.
- Fake structured data.
- Excessive meta tags.

Professional SEO begins with correct content and document structure.

---

## Avoid Framework-Looking HTML in Plain HTML

Do not mimic component systems manually with deeply nested generic markup.

Bad:

```html
<div class="component">
  <div class="component__root">
    <div class="component__container">
      <div class="component__body">
        ...
      </div>
    </div>
  </div>
</div>
```

when one meaningful element is sufficient.

Use the minimum structure required.

---

## Avoid Fake Components

Do not create generic classes like:

```text
component-wrapper
component-container
component-inner
component-content
component-body
```

without a real reason.

These names often indicate markup designed before the actual layout problem was understood.

Use domain-specific or layout-specific names.

---

## Avoid Deep Nesting

Deep DOM trees are difficult to maintain.

If markup looks like:

```text
div
  div
    div
      div
        div
          content
```

review whether each level is necessary.

Sometimes complex layouts legitimately require nesting, but it should result from actual structure rather than habit.

---

## Avoid Duplicate Markup

Do not copy entire large sections for slightly different states if the application can render a shared structure.

Examples:

- Loading vs loaded.
- Mobile vs desktop.
- Active vs inactive.

Use state classes, conditional content, or shared templates where appropriate.

Do not create abstractions when duplication is trivial and temporary, but avoid maintaining two entire copies of the same UI.

---

## Loading States

Use meaningful loading markup.

For a status message:

```html
<p role="status">
  Loading movies…
</p>
```

may be appropriate.

Do not add live regions everywhere.

Use live announcements only when dynamic changes genuinely need to be announced.

---

## Live Regions

Use:

```html
aria-live
role="status"
role="alert"
```

carefully.

Do not mark ordinary content as live.

Too many announcements create a poor screen reader experience.

Use `role="alert"` for important time-sensitive messages, not routine updates.

---

## Error Messages

Connect form errors to the relevant control when appropriate.

Example:

```html
<input
  id="email"
  name="email"
  type="email"
  aria-invalid="true"
  aria-describedby="email-error"
>

<p id="email-error">
  Enter a valid email address.
</p>
```

Do not set `aria-invalid="true"` before an actual validation failure.

State must reflect reality.

---

## Empty States

Write clear empty-state markup.

Example:

```html
<section aria-labelledby="results-heading">
  <h2 id="results-heading">
    Search results
  </h2>

  <p>No movies matched your search.</p>
</section>
```

Do not fill empty states with excessive nested cards merely to occupy space.

---

## Skeletons

Skeleton loaders can improve perceived performance, but do not add enormous duplicate DOM structures unless the design requires them.

Keep skeleton markup lightweight.

Respect reduced-motion preferences when animation is involved.

---

## Reduced Motion

HTML itself generally does not manage motion, but markup should not depend on animation for understanding.

Animations should respect:

```css
prefers-reduced-motion
```

when relevant.

Do not hide essential content behind motion-only interactions.

---

## Native Behavior First

Before adding custom HTML structure, consider whether the platform already provides the behavior.

Examples:

```html
details
summary
dialog
button
select
input
progress
meter
form
```

Native controls usually provide better baseline accessibility and keyboard behavior.

Custom controls should exist because requirements demand them, not because they look more sophisticated.

---

## Preserve User Expectations

Do not disable standard browser behavior unnecessarily.

Avoid blocking:

- Text selection.
- Right-click.
- Browser zoom.
- Keyboard shortcuts.
- Normal link behavior.

unless the application has a legitimate reason.

Professional frontend code works with the browser, not against it.

---

## Do Not Rewrite Working Markup Unnecessarily

When modifying an existing page:

1. Identify the smallest responsible area.
2. Understand existing behavior.
3. Preserve unrelated markup.
4. Preserve existing CSS and JavaScript hooks.
5. Match the current structure.
6. Change only what the task requires.
7. Verify accessibility and responsive behavior affected by the change.

Do not rewrite an entire page merely because a different structure looks cleaner.

---

## Refactoring HTML

When explicitly asked to refactor:

- Preserve visible behavior unless change is requested.
- Preserve JavaScript selectors.
- Preserve form behavior.
- Preserve fragment IDs.
- Preserve accessibility relationships.
- Preserve URLs.
- Preserve data attributes used by scripts.
- Avoid simultaneous styling redesign unless requested.

Markup refactors can silently break JavaScript and CSS even when the page looks similar.

---

## Preserve Hooks

Before removing or renaming:

```text
id
class
data-*
name
aria-*
```

check whether they are used by:

- JavaScript.
- CSS.
- Tests.
- Analytics.
- Accessibility relationships.
- Form submissions.
- Browser routing.

Do not treat attributes as dead solely because their purpose is not obvious from the HTML file.

---

## Avoid Premature Abstraction

Do not create generic markup patterns for hypothetical future components.

A movie card can remain a movie card.

It does not need to become a generic:

```text
UniversalContentEntityDisplayContainer
```

unless the application actually has several compatible content types that benefit from the abstraction.

---

## Avoid AI-Looking Structure

Do not automatically organize every section as:

```text
wrapper
container
inner
header
body
content
footer
```

unless those layers genuinely exist.

Generated HTML often becomes excessively symmetrical.

Real production markup should reflect the actual layout and content.

---

## Avoid Placeholder Architecture

Do not leave comments such as:

```html
<!-- TODO: Add accessibility -->
<!-- TODO: Add responsive support -->
<!-- TODO: Add SEO metadata -->
<!-- TODO: Add more content -->
```

in finished work unless the user explicitly asked for a scaffold.

Implement the requested behavior rather than documenting hypothetical future work.

---

## Avoid Excessive Defensive Markup

Do not add duplicate fallback elements for every possible JavaScript failure unless the application architecture requires them.

Professional HTML should be robust, but robustness should address actual failure modes.

---

## Avoid Copy-Pasted Boilerplate

Do not paste generic blocks such as:

- Massive meta tag collections.
- Dozens of favicon declarations.
- Schema.org markup unrelated to the page.
- Empty social metadata.
- Generic accessibility attributes.
- CSS-framework starter markup.

Use only what the application needs.

---

## Before Finishing

Review the markup and remove or fix:

- Unnecessary wrappers.
- `div` soup.
- Fake buttons.
- Fake links.
- Invalid nesting.
- Duplicate IDs.
- Missing labels.
- Missing meaningful alt text.
- Redundant ARIA.
- Incorrect ARIA state.
- Positive tabindex values.
- Empty `href="#"` placeholders.
- Inline event handlers when the project does not use them.
- Debugging attributes.
- Unnecessary data attributes.
- Decorative comments.
- Tutorial-style comments.
- Generic AI-style class names.
- Placeholder content.
- Duplicate markup.
- Unused containers.
- Unrelated formatting churn.
- SEO boilerplate that does not match the page.
- Accessibility attributes that merely duplicate native semantics.

Then verify the affected page using the project's existing tooling where available.

Typical checks may include:

```text
HTML validation
ESLint
Prettier
accessibility testing
Lighthouse
browser tests
end-to-end tests
```

Do not assume specific tools exist.

Inspect the repository and use its established workflow.

The final HTML should look like it naturally belongs in the project rather than like a generic AI-generated template.

It should feel like markup written by an experienced frontend developer: semantic, minimal, accessible, structurally clear, easy to style, easy to script, and free of unnecessary ceremony.