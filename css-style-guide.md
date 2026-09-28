# CSS Style Guide

Write CSS as an experienced professional frontend developer maintaining a real production codebase.

The goal is clean, predictable, maintainable styling—not CSS that looks generated, over-engineered, excessively specific, or assembled from arbitrary one-off values.

The most important rule:

> Do not optimize for demonstrating CSS knowledge. Optimize for producing the smallest maintainable styling change that fits the existing design system and behaves correctly across the intended layouts.

## General Principles

Prefer:

- Existing project conventions over personal preferences.
- Simple selectors.
- Low specificity.
- Reusable design tokens.
- Layout systems such as Flexbox and Grid.
- Natural document flow.
- Responsive CSS over duplicated markup.
- Semantic class names where classes are used.
- Existing utilities over duplicate custom rules.
- Modern CSS when target browsers support it.
- Small focused changes.
- Predictable cascade behavior.
- Content-driven sizing over hard-coded dimensions.

Avoid:

- Fighting the cascade.
- Excessive `!important`.
- Deep selector chains.
- Arbitrary pixel values everywhere.
- Duplicate rules.
- Unnecessary wrappers created purely for styling.
- Layout hacks that modern CSS already solves.
- Styling unrelated parts of the application during a focused change.

Do not redesign the page unless the task actually calls for a redesign.

---

## Match the Existing Codebase

Before writing CSS, inspect the repository and follow its existing conventions for:

- Class naming.
- File organization.
- CSS architecture.
- Design tokens.
- Variables.
- Utility classes.
- CSS Modules.
- Tailwind.
- Sass or Less.
- Component-scoped styling.
- Responsive breakpoints.
- Spacing scale.
- Typography scale.
- Color system.
- Dark mode.
- Motion.
- Browser support.
- Linting.
- Formatting.

Check relevant files such as:

```text
styles.css
globals.css
tokens.css
theme.css
tailwind.config.js
postcss.config.js
stylelint.config.js
package.json
```

Consistency with the repository is more important than imposing another styling methodology.

Do not introduce BEM into a utility-first project.

Do not introduce utility-style classes into a component-oriented CSS architecture unless there is a clear reason.

Do not rewrite existing styles merely because another approach appears cleaner.

---

## Design System First

Before inventing a value, check whether the project already defines one.

Prefer existing tokens such as:

```css
var(--color-surface)
var(--color-text)
var(--spacing-md)
var(--radius-lg)
var(--shadow-card)
```

over:

```css
#171717
#f4f4f4
18px
13px
7px
```

when equivalent design tokens already exist.

Do not duplicate the design system locally.

---

## CSS Custom Properties

Use custom properties for values that are genuinely shared, thematic, configurable, or semantically meaningful.

Good:

```css
:root {
  --page-max-width: 1200px;
  --surface-radius: 0.75rem;
  --content-gap: 1.5rem;
}
```

Do not create variables for every literal.

Avoid:

```css
:root {
  --zero: 0;
  --one-pixel: 1px;
  --white: white;
}
```

unless those values carry real system meaning.

---

## Semantic Tokens

Prefer semantic tokens:

```css
--color-background
--color-surface
--color-text
--color-muted
--color-danger
```

over raw value-oriented names:

```css
--gray-7
--blue-3
```

at component usage sites.

Primitive palette tokens can still exist underneath a design system.

Use the architecture already established by the project.

---

## Avoid Arbitrary Values

Do not introduce random values simply because they visually look close enough.

Bad:

```css
.card {
  padding: 17px;
  border-radius: 11px;
  gap: 13px;
}
```

Prefer values from the existing spacing and radius scale.

For example:

```css
.card {
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  gap: var(--spacing-md);
}
```

Arbitrary values are appropriate when the design genuinely requires them.

Do not force everything onto a scale if precise dimensions are necessary.

---

## The Cascade

Work with the cascade rather than fighting it.

Prefer simple predictable rules.

Do not solve every conflict with:

```css
!important
```

First inspect:

- Source order.
- Selector specificity.
- Inheritance.
- Cascade layers.
- Existing component rules.

Fix the actual conflict.

---

## `!important`

Avoid `!important` in ordinary component styling.

It may be justified for:

- Utility classes intentionally designed to override.
- Third-party styles that cannot reasonably be changed.
- Accessibility helpers.
- Highly constrained legacy systems.

Do not stack:

```css
color: red !important;
margin: 0 !important;
display: block !important;
```

merely because the selector architecture is unclear.

---

## Specificity

Keep specificity low.

Good:

```css
.movie-card {
  ...
}

.movie-card__title {
  ...
}
```

Avoid:

```css
body main .content .movie-grid div.movie-card article h3.title {
  ...
}
```

High specificity makes future changes difficult.

Prefer classes or established component selectors.

---

## Avoid ID Selectors for Styling

Avoid:

```css
#main-content {
  ...
}
```

for normal styling when a class works.

IDs create high specificity.

Use IDs primarily for:

- Fragment navigation.
- Labels.
- ARIA relationships.
- JavaScript hooks when appropriate.

Follow the project convention.

---

## Avoid Element Chains

Avoid selectors like:

```css
header nav ul li a span {
  ...
}
```

when:

```css
.nav-link__label {
  ...
}
```

or a simpler contextual selector would be clearer.

Deep selectors are fragile because markup changes break styling.

---

## Avoid Over-Specific Selectors

Do not write:

```css
div.card
button.action-button
ul.navigation-list
```

unless the element type is intentionally part of the contract.

Prefer:

```css
.card
.action-button
.navigation-list
```

This allows markup to evolve without unnecessary CSS changes.

---

## Descendant Selectors

Use descendant selectors when the relationship itself matters.

Good:

```css
.article-content p {
  max-width: 70ch;
}
```

Avoid deeply nested dependencies.

Keep selectors resilient to minor markup changes.

---

## Child Selectors

Use `>` when styling only direct children is intentional.

Example:

```css
.toolbar > button {
  flex: none;
}
```

Do not use child selectors mechanically.

Use them when they protect against unintended nested matches.

---

## Class Naming

Follow the project's naming system.

Possible conventions include:

- BEM.
- Component classes.
- CSS Modules.
- Utility classes.
- Framework-generated classes.
- Data-state selectors.

Do not introduce a second naming system.

---

## Avoid Generic AI-Style Class Names

Avoid vague names such as:

```text
container
wrapper
inner
content
section
box
item
thing
main-box
left-part
right-part
```

when a meaningful name is available.

Prefer:

```text
movie-grid
player-shell
search-toolbar
episode-list
rating-badge
```

Do not overcompensate with excessively long names either.

---

## Avoid Visual-Only Naming

Avoid:

```text
red-button
blue-box
left-panel
big-text
```

when the style may change.

Prefer role-oriented names:

```text
danger-button
sidebar
page-title
status-message
```

Utility-class systems are an exception because visual naming is intentional there.

---

## BEM

If the project uses BEM, follow it consistently.

Example:

```css
.movie-card {}
.movie-card__poster {}
.movie-card__title {}
.movie-card--featured {}
```

Do not create absurd chains:

```css
.movie-card__content__header__title__text {}
```

BEM is meant to clarify relationships, not encode the entire DOM tree.

---

## CSS Modules

When using CSS Modules, class names can remain locally meaningful:

```css
.card {}
.title {}
.poster {}
```

because scoping prevents collisions.

Do not imitate global BEM verbosity unnecessarily inside scoped modules unless the project convention prefers it.

---

## Utility-First Projects

If the project uses Tailwind or another utility framework, prefer existing utilities rather than duplicating them in custom CSS.

Do not create:

```css
.flexCenter {
  display: flex;
  align-items: center;
  justify-content: center;
}
```

in a utility-first project if equivalent utilities already exist.

Likewise, do not force huge utility strings into a project that intentionally uses component CSS.

---

## Layout

Prefer modern layout primitives:

- Flexbox.
- Grid.
- Normal flow.
- `gap`.
- Logical properties.

Avoid old layout hacks based on:

- Floats.
- Table layouts.
- Negative margins.
- Absolute positioning everywhere.

unless maintaining legacy code or solving a specific edge case.

---

## Flexbox

Use Flexbox for primarily one-dimensional layouts.

Good:

```css
.toolbar {
  display: flex;
  align-items: center;
  gap: 1rem;
}
```

Do not use Flexbox automatically for every container.

If elements naturally flow vertically, normal block layout may already be enough.

---

## Grid

Use Grid for two-dimensional layouts or when explicit track control improves the design.

Good:

```css
.movie-grid {
  display: grid;
  grid-template-columns:
    repeat(auto-fill, minmax(12rem, 1fr));
  gap: 1.5rem;
}
```

Do not use Grid for a simple row of two controls when Flexbox is clearer.

---

## `gap`

Prefer `gap` for spacing between flex or grid children.

Good:

```css
.actions {
  display: flex;
  gap: 0.75rem;
}
```

Avoid child-margin hacks such as:

```css
.actions > * + * {
  margin-left: 0.75rem;
}
```

when `gap` is supported by the target environment.

---

## Natural Flow

Keep elements in normal flow when possible.

Do not use absolute positioning to construct ordinary page layouts.

Bad:

```css
.title {
  position: absolute;
  top: 24px;
  left: 32px;
}
```

when simple padding or grid placement would work.

Absolute positioning is appropriate for:

- Overlays.
- Badges.
- Floating controls.
- Decorative elements.
- Explicit layered interfaces.

---

## Positioning

Use:

```css
position: relative;
```

only when it serves a purpose such as anchoring positioned descendants.

Do not add `position: relative` automatically to every component.

---

## Z-Index

Keep z-index values deliberate.

Avoid:

```css
z-index: 999999;
```

unless the architecture truly requires it.

Prefer an established layering scale:

```css
--z-dropdown
--z-modal
--z-toast
```

or small local stacking contexts.

---

## Stacking Contexts

Understand what creates stacking contexts, including properties such as:

```text
position + z-index
transform
opacity
filter
isolation
```

Do not keep increasing `z-index` when the actual issue is a stacking-context boundary.

Fix the structure.

---

## `isolation`

Use:

```css
isolation: isolate;
```

when a component should establish a predictable local stacking context.

Do not apply it everywhere mechanically.

---

## Widths

Prefer flexible sizing.

Good:

```css
width: 100%;
max-width: 72rem;
```

Avoid fixed widths for general responsive content:

```css
width: 1200px;
```

unless the component truly requires fixed dimensions.

---

## `min-width: 0`

Remember that flex and grid children may refuse to shrink because of intrinsic sizing.

When text overflow appears inside a flex/grid item, this may be appropriate:

```css
min-width: 0;
```

Do not blindly add it everywhere.

Use it when intrinsic sizing is actually causing overflow.

---

## `min-height: 0`

Similarly, scrollable children inside flex/grid layouts may require:

```css
min-height: 0;
```

to shrink properly.

Use it intentionally.

---

## `max-width`

Use readable content widths.

For prose:

```css
.article {
  max-width: 70ch;
}
```

can be appropriate.

Do not apply narrow text measures to layouts like data grids or media galleries.

---

## Viewport Units

Use viewport units deliberately.

Be cautious with:

```css
height: 100vh;
```

on mobile because browser UI can affect viewport measurements.

Where browser support permits, consider:

```css
100dvh
100svh
100lvh
```

based on actual behavior required.

Do not replace all viewport units blindly.

---

## Fixed Heights

Avoid fixed content heights where content should naturally expand.

Bad:

```css
.card {
  height: 340px;
}
```

if descriptions can vary.

Prefer:

```css
min-height
aspect-ratio
grid/flex alignment
```

depending on the design.

Fixed heights are appropriate for intentionally constrained interfaces.

---

## `aspect-ratio`

Use:

```css
aspect-ratio: 2 / 3;
```

for predictable media proportions.

This is often cleaner than padding-top hacks.

Use actual image dimensions in HTML as well when possible.

---

## Overflow

Do not hide overflow as a generic fix.

Bad:

```css
body {
  overflow-x: hidden;
}
```

when a child is incorrectly wider than the viewport.

Find the overflowing element.

Use `overflow: hidden` or `clip` only when clipping is intentional.

---

## Horizontal Overflow

Common causes include:

- Fixed widths.
- `100vw` inside layouts with scrollbars.
- Long unbreakable text.
- Transforms.
- Absolute positioning.
- Large margins.
- Flex children without `min-width: 0`.

Fix the cause rather than masking it globally.

---

## Scrolling

Use scrolling intentionally.

Example:

```css
.episode-list {
  overflow-y: auto;
}
```

Do not create nested scroll containers without a reason.

Multiple independent scrolling regions can make keyboard, touch, and accessibility behavior worse.

---

## Scrollbars

Do not hide scrollbars merely for aesthetics when the region still scrolls.

Hidden scrollbars can reduce discoverability.

If the design intentionally uses visually hidden scrollbars, ensure scrolling remains obvious and usable through touch, wheel, keyboard, and other input methods.

---

## Overscroll

Use properties such as:

```css
overscroll-behavior
```

when preventing scroll chaining is actually useful, such as in a modal or drawer.

Do not disable normal browser scrolling behavior globally without need.

---

## Spacing

Use the project's spacing scale.

Prefer:

```css
gap: var(--space-4);
padding: var(--space-6);
```

over arbitrary values when a scale exists.

Keep spacing relationships intentional.

Do not create dozens of nearly identical spacing values.

---

## Margin vs Padding

Use padding for space inside an element's box.

Use margins for separation outside the box.

Do not choose based solely on which produces the desired pixels.

The box model should remain understandable.

---

## Margin Collapse

Remember vertical margins can collapse in normal block flow.

Do not add mysterious wrapper elements solely because margin behavior was misunderstood.

Use:

- Padding.
- `display: flow-root`.
- Flex/Grid.
- `gap`.

when those better express the layout.

---

## Logical Properties

Prefer logical properties when the application may support different writing directions.

Examples:

```css
margin-inline
padding-inline
margin-block
inset-inline-start
border-inline-start
```

rather than hard-coded left/right assumptions.

Follow existing project conventions and browser targets.

---

## Typography

Use an intentional typography system.

Define or reuse values for:

- Font family.
- Font size.
- Line height.
- Weight.
- Letter spacing.

Do not assign arbitrary typography independently to every component.

---

## Font Sizes

Prefer a coherent scale.

Avoid:

```css
font-size: 15px;
font-size: 17px;
font-size: 19px;
font-size: 21px;
```

throughout unrelated components unless the design explicitly calls for those values.

Use design tokens where possible.

---

## Relative Units

Use `rem` or project-standard relative units for typography and spacing when appropriate.

Do not convert every pixel value to `rem` mechanically.

Pixels are appropriate for:

- Borders.
- Fine visual details.
- Certain fixed media dimensions.

Choose units based on semantics.

---

## Line Height

Set readable line heights for text.

For body text:

```css
line-height: 1.5;
```

or an existing token is often appropriate.

Do not set tiny fixed pixel line heights that clip text when users zoom or fonts change.

---

## Font Weight

Use only weights available in the loaded font.

Do not specify:

```css
font-weight: 650;
```

if the font file does not support a variable weight or that weight is unavailable.

Avoid loading many unused font weights.

---

## Text Truncation

Use truncation when the design requires constrained content.

Single-line:

```css
overflow: hidden;
text-overflow: ellipsis;
white-space: nowrap;
```

Multi-line may use:

```css
display: -webkit-box;
-webkit-line-clamp: 2;
-webkit-box-orient: vertical;
overflow: hidden;
```

when supported and accepted by the project.

Do not truncate important content merely to make card heights match.

---

## Wrapping

For long identifiers or URLs, consider:

```css
overflow-wrap: anywhere;
```

or:

```css
word-break
```

based on actual content.

Do not globally break all words.

---

## Text Alignment

Do not center large amounts of body copy merely because it looks symmetrical.

Use alignment appropriate to the content.

Centering can be effective for short headings or hero text.

---

## Colors

Use the existing color system.

Do not introduce slightly different near-duplicate colors:

```css
#181818
#191919
#1a1a1a
```

across adjacent components without a reason.

Prefer shared tokens.

---

## Contrast

Ensure text and interactive controls have sufficient contrast.

Do not sacrifice readability solely to achieve a muted aesthetic.

Pay particular attention to:

- Disabled-looking text that is not actually disabled.
- Placeholder text.
- Secondary metadata.
- Text over images.
- Focus indicators.

---

## Color as the Only Signal

Do not rely solely on color to convey important state.

For errors, success, selection, or warnings, use additional cues such as:

- Text.
- Icons.
- Shape.
- Borders.
- Labels.

CSS should support accessible state communication.

---

## Dark Mode

Follow the project's dark-mode architecture.

Possible approaches include:

```css
@media (prefers-color-scheme: dark)
```

or:

```css
[data-theme="dark"]
```

Do not add a second theme mechanism.

Use semantic tokens so components do not require large duplicated theme blocks.

---

## Theme Tokens

Good:

```css
:root {
  --surface: #fff;
  --text: #111;
}

[data-theme="dark"] {
  --surface: #151515;
  --text: #f5f5f5;
}
```

Then:

```css
.card {
  background: var(--surface);
  color: var(--text);
}
```

Prefer this over duplicating every component rule for each theme.

---

## Borders

Use borders intentionally.

Avoid adding borders around every container simply to create visual structure.

Use spacing, background contrast, or grouping where appropriate.

Keep border color and width consistent with the design system.

---

## Border Radius

Use a coherent radius scale.

Do not assign random radii per component unless the design clearly calls for them.

Avoid excessive "everything is a pill" styling unless that is actually the product language.

---

## Shadows

Use shadows sparingly.

Avoid giant generic shadows:

```css
box-shadow:
  0 20px 60px rgba(0, 0, 0, 0.4);
```

on every surface.

Use the project's elevation system.

Shadows should communicate hierarchy, not simply decorate every card.

---

## Gradients

Use gradients when they support the design.

Do not automatically add gradients because they make generated UI appear more "modern."

Avoid decorative gradients that reduce readability or clash with the existing visual system.

---

## Backdrop Filters

Use:

```css
backdrop-filter
```

sparingly.

It can be expensive and browser-dependent.

Do not turn every header or card into blurred glass unless the product design actually uses that language.

---

## Blur

Avoid blur-heavy visual design by default.

Blur can:

- Reduce legibility.
- Increase GPU cost.
- Produce inconsistent rendering.
- Make interfaces feel generically AI-designed.

Use it intentionally.

---

## Opacity

Do not lower opacity on an entire container if child content should remain fully opaque.

Bad:

```css
.card {
  opacity: 0.6;
}
```

if only the background should be translucent.

Use an alpha color instead.

---

## Background Images

Use background images for decorative imagery.

Use HTML `<img>` when the image is meaningful content.

Do not use CSS backgrounds for content images that need:

- Alt text.
- Responsive image loading.
- Intrinsic dimensions.
- Semantic meaning.

---

## `object-fit`

For content images constrained to a box:

```css
object-fit: cover;
```

or:

```css
object-fit: contain;
```

can be appropriate.

Choose based on whether cropping is acceptable.

Do not use `cover` blindly for images where the full content must remain visible.

---

## Pseudo-Elements

Use `::before` and `::after` for decorative elements.

Good:

```css
.badge::before {
  content: "";
  ...
}
```

Do not use pseudo-elements to insert important textual content.

Generated CSS content may not be reliably accessible or selectable.

---

## `content`

Keep important content in HTML.

Avoid:

```css
.button::after {
  content: "Download";
}
```

when the word is part of the actual interface.

Pseudo-element content should generally be decorative.

---

## Responsive Design

Prefer content-driven responsive layouts.

Do not start by creating separate desktop, tablet, and mobile versions of every component.

Use flexible layouts first.

Add breakpoints where the layout actually breaks.

---

## Breakpoints

Use the project's established breakpoint system.

Avoid arbitrary media queries such as:

```css
@media (max-width: 913px)
```

unless that exact breakpoint is justified by the component.

Prefer existing design breakpoints or content-driven thresholds.

---

## Mobile First

Use mobile-first styles when that matches the project.

Example:

```css
.grid {
  grid-template-columns: 1fr;
}

@media (min-width: 48rem) {
  .grid {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

Do not force mobile-first rewrites into a desktop-first existing codebase as unrelated cleanup.

Consistency matters.

---

## Container Queries

Use container queries when component behavior should depend on its available space rather than the viewport.

Example:

```css
.card-list {
  container-type: inline-size;
}

@container (min-width: 40rem) {
  .card {
    grid-template-columns: 10rem 1fr;
  }
}
```

Do not introduce them if the browser support target or project architecture does not permit them.

---

## Media Queries

Keep media queries near the component or in the project-defined responsive structure.

Do not scatter contradictory media rules throughout unrelated files.

Avoid duplicating the same breakpoint behavior in multiple places.

---

## Orientation Queries

Use orientation queries only when actual layout behavior depends on orientation.

Do not assume landscape means desktop or portrait means mobile.

Viewport dimensions are more reliable.

---

## Hover

Do not rely on hover as the only way to reveal essential controls.

Touch devices may not have hover.

Use:

```css
@media (hover: hover)
```

when hover-only enhancements are appropriate.

Essential functionality must remain available without hover.

---

## Pointer Queries

Use:

```css
@media (pointer: coarse)
```

when interaction sizing genuinely needs adaptation for coarse pointers.

Do not build separate interfaces solely based on pointer media queries without clear need.

---

## Focus Styles

Always preserve visible keyboard focus.

Use:

```css
:focus-visible
```

where supported and appropriate.

Example:

```css
.button:focus-visible {
  outline: 2px solid var(--focus);
  outline-offset: 2px;
}
```

Do not globally remove outlines.

---

## `:focus-visible`

Prefer `:focus-visible` when visual focus should primarily appear for keyboard-like navigation.

Do not eliminate `:focus` behavior without considering browser fallback and project support.

---

## `:focus-within`

Use `:focus-within` when a container should react to focus inside it.

Example:

```css
.search-field:focus-within {
  border-color: var(--focus);
}
```

Do not recreate this behavior with JavaScript unless needed.

---

## `:has()`

Use `:has()` when it meaningfully simplifies relationship-based styling and browser support allows it.

Example:

```css
.field:has(input:invalid) {
  border-color: var(--danger);
}
```

Do not use complex `:has()` selectors everywhere simply because the feature is modern.

Keep selectors understandable.

---

## States

Represent component states through established mechanisms such as:

```text
.is-active
.is-disabled
[aria-expanded="true"]
[data-state="open"]
```

Follow the project convention.

Do not invent multiple competing state systems.

---

## ARIA State Styling

Styling ARIA states can align visual appearance with accessibility state.

Example:

```css
button[aria-expanded="true"] {
  ...
}
```

This is useful when the ARIA attribute already represents the real component state.

Do not add ARIA solely to gain a CSS selector.

---

## Data Attributes

Data attributes can be clean styling hooks for component state:

```css
.tabs[data-orientation="vertical"] {
  ...
}
```

Use them when they already belong to component behavior.

Do not fill markup with unnecessary data attributes purely for styling convenience.

---

## Disabled State

Style actual disabled controls appropriately.

Remember that:

```html
disabled
```

and:

```html
aria-disabled="true"
```

have different behavior.

Do not style `aria-disabled` as though it automatically prevents interaction.

CSS should reflect the actual interaction model.

---

## Cursor

Do not add:

```css
cursor: pointer;
```

to every clickable element automatically.

Native buttons and links already communicate interactivity through platform behavior.

Use cursor styling when it genuinely improves clarity, especially for custom interactive elements.

Avoid making non-interactive elements look clickable.

---

## Selection

Customize text selection only if the product design benefits from it.

Do not disable selection broadly:

```css
user-select: none;
```

on ordinary text.

Users may need to copy content.

Use it on drag handles or controls where selection interferes with interaction.

---

## Pointer Events

Use:

```css
pointer-events: none;
```

carefully.

Do not use it as a generic way to "disable" controls.

It does not provide semantic disabled behavior and may affect accessibility or keyboard interaction.

---

## Motion

Use motion to communicate state and hierarchy.

Do not animate everything.

Good uses include:

- Opening/closing.
- State changes.
- Hover feedback.
- Progress.
- Spatial relationships.

Avoid decorative motion that distracts from the task.

---

## Transition Scope

Avoid:

```css
transition: all 0.3s ease;
```

Prefer explicit properties:

```css
transition:
  background-color 150ms ease,
  border-color 150ms ease,
  transform 150ms ease;
```

`all` can animate unintended properties and create performance or visual issues.

---

## Transition Duration

Keep durations intentional.

Do not assign random values across components:

```css
173ms
247ms
311ms
```

unless the design system explicitly uses them.

Use shared motion tokens where available.

---

## Transform Animations

Prefer animating:

```text
transform
opacity
```

for smooth visual effects when possible.

Be cautious animating layout-heavy properties such as:

```text
width
height
top
left
margin
```

when performance matters.

Do not contort simple UI solely to optimize animation.

---

## Reduced Motion

Respect user preferences.

Example:

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    scroll-behavior: auto;
  }
}
```

The exact implementation should match the project.

Do not merely slow animations down; often they should be removed or simplified.

---

## Infinite Animations

Avoid endless decorative animations unless they serve a real purpose.

Loading indicators are a reasonable use.

Background ornaments spinning forever usually are not.

Consider CPU, GPU, battery, and accessibility costs.

---

## Keyframes

Name animations after behavior, not appearance.

Prefer:

```css
@keyframes fade-in
@keyframes slide-up
```

over:

```css
@keyframes animation1
```

Do not create duplicate keyframes with nearly identical behavior.

---

## Scroll Behavior

Use:

```css
scroll-behavior: smooth;
```

only when appropriate.

Respect reduced motion.

Do not globally force smooth scrolling if it causes navigation or accessibility problems.

---

## Scroll Snap

Use scroll snapping for interfaces naturally suited to discrete stops such as carousels.

Do not add aggressive snapping to ordinary scrolling pages.

Users should retain control.

---

## Carousels

Do not hide content or controls purely to create a carousel aesthetic.

When CSS is responsible for layout:

- Keep horizontal overflow deliberate.
- Preserve keyboard access.
- Avoid invisible scroll regions.
- Use snap behavior carefully.

Do not remove scrollbars without providing obvious interaction.

---

## Forms

Keep form styling consistent with native interaction semantics.

Do not make:

```css
input,
select,
button {
  appearance: none;
}
```

globally unless you fully restyle all affected states and controls.

Removing native appearance creates accessibility and cross-browser work.

---

## `appearance: none`

Use it only when custom styling requires it and replacement affordances are complete.

For example, custom select arrows require careful consideration.

Do not strip native control appearance casually.

---

## Form Focus

Ensure inputs have a clear focus state.

Do not use only subtle color changes that fail under low contrast.

Keep error, hover, focus, and disabled states visually distinct.

---

## Placeholder Styling

Placeholder text should remain readable but secondary.

Do not make it so faint that it becomes illegible.

Remember placeholders are not labels.

---

## Autofill

If browser autofill clashes badly with the design, style it carefully.

Do not spend large amounts of CSS fighting native autofill without a real problem.

Browser behavior varies.

---

## Tables

Use CSS to enhance actual tables, not replace them with grid-like `<div>` structures solely for styling.

Handle overflow on narrow screens intentionally.

For example:

```css
.table-scroll {
  overflow-x: auto;
}
```

Do not force every table into unreadably compressed columns.

---

## Lists

Reset list styles only when the design calls for it.

Avoid global:

```css
ul,
ol {
  list-style: none;
  padding: 0;
}
```

unless the project intentionally resets lists everywhere.

Content lists often benefit from normal markers.

---

## CSS Resets

Use the project's existing reset or normalization strategy.

Do not add another reset stylesheet.

Avoid huge global resets without understanding their effect.

---

## Global Styles

Keep global selectors restrained.

Rules applied to:

```css
*
body
a
button
input
img
```

affect the entire application.

Do not change them casually for one component problem.

---

## Universal Selector

The universal selector is appropriate for things like:

```css
*,
*::before,
*::after {
  box-sizing: border-box;
}
```

when part of the project's global foundation.

Do not use:

```css
* {
  margin: 0;
  padding: 0;
}
```

without understanding how it affects native elements.

---

## Box Sizing

A global:

```css
box-sizing: border-box;
```

strategy is standard and useful.

Follow the existing setup.

Do not repeatedly specify it per component when already inherited or globally configured.

---

## Inheritance

Use inheritance intentionally for properties such as:

- Font.
- Color.
- Line-height.

Do not repeat inherited values unnecessarily in every nested element.

Let CSS do its job.

---

## `inherit`

Use `inherit` when a property should deliberately match the parent.

Example:

```css
button {
  font: inherit;
}
```

may be appropriate in a reset.

Do not scatter `inherit` everywhere without need.

---

## `currentColor`

Use:

```css
currentColor
```

for borders, icons, and SVG elements that should match text color.

This can reduce duplicate color declarations.

Example:

```css
.icon {
  fill: currentColor;
}
```

---

## SVG Styling

Prefer:

```css
fill: currentColor;
```

for icons intended to inherit text color.

Do not globally override fills on complex illustrations that contain intentional colors.

---

## Specificity Escalation

If you find yourself writing:

```css
.page .section .card .title
```

followed later by:

```css
.page .section.special .card .title
```

and then:

```css
body .page .section.special .card .title
```

stop.

The architecture is escalating.

Consider:

- A component class.
- State class.
- Data attribute.
- Cascade layer.
- Better source ordering.

Do not continue the specificity arms race.

---

## Cascade Layers

Use `@layer` when the project has enough styling sources that explicit cascade ordering helps.

Example:

```css
@layer reset, base, components, utilities;
```

Do not introduce cascade layers to a tiny stylesheet merely because they are modern.

Use them when they solve real cascade organization problems.

---

## Nesting

Native CSS nesting or preprocessor nesting can improve locality.

Keep nesting shallow.

Good:

```css
.card {
  padding: 1rem;

  & .title {
    margin: 0;
  }
}
```

Avoid:

```css
.card {
  .content {
    .header {
      .title {
        span {
          ...
        }
      }
    }
  }
}
```

Deep nesting creates brittle high-specificity selectors.

---

## Sass

If the project uses Sass, use it according to existing conventions.

Do not add Sass to a plain CSS project for one helper.

Avoid excessive use of:

- Mixins.
- Functions.
- Nested selectors.
- Inheritance with `%placeholder`.
- Loops generating large amounts of CSS.

Use preprocessing where it genuinely reduces maintenance.

---

## Sass Variables vs CSS Variables

Use Sass variables for compile-time constants and code generation.

Use CSS custom properties for runtime theming and cascade-aware values.

Do not duplicate the same token system in both without a clear architecture.

---

## Mixins

Use mixins for meaningful repeated declarations.

Do not create:

```scss
@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}
```

if it appears once.

Do not create dozens of one-line mixins merely to avoid writing CSS.

---

## `@extend`

Use Sass `@extend` carefully.

It can produce unexpected selector combinations.

Prefer explicit shared classes or mixins when they are easier to reason about.

Follow existing project conventions.

---

## Generated CSS

Be aware that loops and preprocessors can generate huge stylesheets.

Do not generate hundreds of classes for values the application never uses.

Build output size matters.

---

## Tailwind

If Tailwind is used:

- Prefer existing scale values.
- Reuse design tokens.
- Avoid arbitrary values unless needed.
- Extract components only when repetition or semantics justify it.
- Keep responsive modifiers understandable.
- Avoid enormous class strings caused by repeated conflicting utilities.

Do not fight Tailwind by writing parallel custom CSS for things the framework already handles well.

---

## Tailwind Arbitrary Values

Avoid:

```text
mt-[13px]
w-[437px]
rounded-[11px]
```

unless the design actually requires those values.

Prefer the configured spacing, width, and radius scales.

Arbitrary utilities are useful exceptions, not the default.

---

## Tailwind `@apply`

Use `@apply` according to the project's conventions.

Do not recreate an entire semantic CSS architecture via `@apply` if direct utility composition is already the intended workflow.

Likewise, use `@apply` when repeated utility groups genuinely benefit from extraction.

---

## CSS-in-JS

If the project uses CSS-in-JS, follow its existing patterns.

Do not introduce styled-components, Emotion, or another runtime styling library into a project using normal CSS without a reason.

Avoid dynamically generating styles in JavaScript when a static class or custom property would be clearer.

---

## Inline Styles

Inline styles are appropriate for genuinely dynamic values.

Example:

```html
<div style="--progress: 64%">
```

combined with:

```css
.progress {
  width: var(--progress);
}
```

can be cleaner than dynamically generating entire CSS rules.

Do not place large static style blocks inline in markup or components when the project has a normal stylesheet system.

---

## CSS Variables for Dynamic Values

Prefer CSS custom properties for dynamic styling when they preserve separation of concerns.

Example:

```css
.poster {
  transform: translateX(var(--offset));
}
```

This can be cleaner than toggling many one-off classes.

Use judgment.

---

## Performance

Avoid premature CSS micro-optimization.

Modern browsers handle normal selectors efficiently.

Focus first on:

- DOM size.
- Large layout shifts.
- Expensive filters.
- Huge paint areas.
- Continuous animations.
- Complex fixed backgrounds.
- Oversized shadows.
- Excessive style recalculation from JavaScript.

Do not rewrite readable selectors based on outdated performance myths.

---

## Expensive Visual Effects

Be cautious with:

```text
filter
backdrop-filter
large box-shadow
mix-blend-mode
large blur
continuous transforms
```

over large areas.

Do not remove them automatically, but use them intentionally.

---

## `will-change`

Do not add:

```css
will-change: transform;
```

everywhere.

It can increase memory usage by creating compositor layers.

Use it sparingly for known imminent animations or performance issues.

Remove it when unnecessary.

---

## GPU Hacks

Avoid old hacks like:

```css
transform: translateZ(0);
```

solely to force GPU acceleration.

Modern browsers make their own compositing decisions.

Use profiling before forcing layers.

---

## Layout Thrashing

CSS alone rarely causes layout thrashing; JavaScript repeatedly reading and writing layout does.

Do not add bizarre CSS workarounds for what is actually a JavaScript measurement loop.

Diagnose the real source.

---

## Containment

Use:

```css
contain
content-visibility
```

when large isolated rendering regions benefit from them.

Do not add them blindly.

Containment changes layout and paint semantics.

---

## `content-visibility`

It can help long pages or large off-screen regions.

Use it only when browser support and rendering behavior are understood.

Do not apply it to critical interactive content without testing.

---

## `contain-intrinsic-size`

When using `content-visibility`, consider intrinsic sizing to reduce layout jumps.

Do not invent sizes without understanding expected content dimensions.

---

## Accessibility

CSS must not make accessible HTML unusable.

Do not:

- Hide focus.
- Make text unreadably low contrast.
- Remove control affordances.
- Hide important content only visually without understanding accessibility effects.
- Convey state solely through color.
- Reorder content visually in a way that breaks keyboard or reading order.

Styling should reinforce semantics, not undermine them.

---

## Visually Hidden Content

Use an established visually-hidden class.

Typical pattern:

```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

Use the project's existing implementation rather than inventing another.

Do not use:

```css
display: none;
```

when content must remain accessible to screen readers.

---

## Hiding Content

Understand the difference between:

```css
display: none;
visibility: hidden;
opacity: 0;
```

They affect layout and interaction differently.

Do not use `opacity: 0` as a substitute for hidden content if it remains focusable or clickable.

---

## Invisible Interactive Elements

Be careful with:

```css
opacity: 0;
```

on interactive controls.

An invisible control may still receive pointer or keyboard interaction.

If the element should not be interactive, manage its state appropriately in HTML or JavaScript too.

---

## Reduced Transparency

Where relevant, consider user preferences and readability if the design relies heavily on transparent layers.

Do not make essential content dependent on translucency effects.

---

## Print Styles

Only add print-specific CSS when the application actually supports printable content.

Do not generate a giant print stylesheet automatically.

If print is important, ensure:

- Navigation is removed where appropriate.
- Content remains readable.
- URLs or context are preserved if useful.
- Interactive-only elements do not clutter output.

---

## Browser Support

Use modern CSS according to the project's actual browser support matrix.

Do not avoid useful features because of obsolete browsers the project does not support.

Likewise, do not use bleeding-edge features if the target environment cannot handle them.

Check:

- Browserslist.
- Build tooling.
- Product requirements.
- Existing patterns.

---

## Prefixes

Do not manually add vendor prefixes unless the project requires them.

Use Autoprefixer or the project's build system where available.

Avoid outdated blocks such as:

```css
-webkit-border-radius
-moz-border-radius
```

for universally supported properties.

---

## Feature Queries

Use:

```css
@supports
```

when progressive enhancement for a newer feature is actually needed.

Do not wrap every modern property in a feature query without a compatibility reason.

---

## Fallbacks

Provide fallbacks when a feature materially affects usability in supported browsers.

Do not add duplicate legacy implementations for browsers outside the product's support target.

---

## CSS Resilience

Styles should tolerate:

- Longer text.
- Missing images.
- Extra list items.
- Different viewport widths.
- Localization.
- User zoom.
- Dynamic content.

Do not tune everything exclusively for one screenshot.

---

## Content-Driven Design

Do not hard-code a component around one sample value.

Bad:

```css
.title {
  width: 180px;
}
```

because the sample title happened to fit.

Prefer flexible constraints.

---

## Long Content

Test:

- Long titles.
- Long button labels.
- Long usernames.
- Large numbers.
- Multi-line descriptions.

Do not assume production content resembles placeholder text.

---

## Localization

Expect translated strings to be longer or differently structured.

Avoid fixed-width labels and controls where text must fit.

Use logical properties where appropriate.

---

## User Zoom

Layouts should remain usable at increased browser zoom.

Avoid:

- Fixed tiny heights.
- Clipped text.
- Absolute positioning tied to exact pixel measurements.
- Disabling overflow that users need.

---

## CSS Reset Buttons

If buttons are reset:

```css
button {
  font: inherit;
}
```

ensure essential native affordances are intentionally restored through component styles.

Do not remove all native styling without rebuilding:

- Focus.
- Disabled state.
- Hover.
- Active state.
- Cursor/interaction clarity.

---

## Anchor Styling

Links should remain recognizable as links when context requires it.

Do not globally remove underlines from all anchors if doing so makes inline links indistinguishable from text.

Navigation links may have a different visual language.

---

## Hover States

Hover effects should be subtle and informative.

Avoid dramatic:

```css
transform: scale(1.1);
```

on dense grids where surrounding content jumps or overlaps.

Prefer stable effects such as:

- Color changes.
- Border changes.
- Small transforms.
- Shadow changes.

depending on the design.

---

## Active States

Buttons and controls should have meaningful active/pressed feedback where appropriate.

Do not create excessive animations for a basic click.

Simple feedback is usually enough.

---

## Disabled Styling

Disabled controls should look unavailable without becoming unreadable.

Do not reduce opacity so far that text becomes illegible.

Remember disabled is a semantic state, not just a visual style.

---

## Error Styling

Errors should be distinguishable and readable.

Do not rely solely on a red border.

Combine styling with visible error text or icons where appropriate.

---

## Success Styling

Similarly, do not rely only on green.

Use labels or icons when state matters.

---

## Skeleton Loaders

If skeletons are used:

- Keep them structurally simple.
- Avoid replicating every detail.
- Respect reduced motion.
- Do not use expensive animations on large lists.

Skeletons should reduce perceived waiting, not create GPU-heavy decoration.

---

## Spinners

Use a spinner when progress is indeterminate.

Do not show several competing spinners on the same screen unless different independent operations genuinely require them.

Prefer established component styles.

---

## Loading Animations

Avoid unnecessarily elaborate keyframes.

A simple rotation or pulse is often enough.

Do not animate large page sections continuously.

---

## CSS Comments

Use comments sparingly.

Good:

```css
/* Keep this above the player so native iframe controls remain reachable. */
.player-toolbar {
  z-index: var(--z-player-toolbar);
}
```

Bad:

```css
/* Set the background color */
.card {
  background: black;
}
```

Comments should explain unusual decisions, not narrate declarations.

---

## Avoid Decorative Comments

Do not add:

```css
/* ========================== */
/*       CARD STYLES          */
/* ========================== */
```

unless the repository intentionally uses that format.

Good file structure and component naming should usually provide enough organization.

---

## Avoid AI-Looking Comments

Avoid prose such as:

```text
This section is responsible for...
The following styles ensure...
This provides a modern and polished appearance...
This creates a robust responsive layout...
```

Write concise technical explanations only when needed.

---

## File Organization

Organize styles according to the project's architecture.

Possible structures include:

```text
base/
components/
utilities/
pages/
themes/
```

or colocated component styles.

Do not introduce a new folder structure for one feature.

---

## Component Styles

Keep component-specific styles near the component when the architecture supports it.

Avoid one giant global stylesheet containing every component if the project is already modular.

Likewise, do not split a small static site into dozens of tiny CSS files without reason.

---

## Global vs Local

Ask whether a rule truly belongs globally.

A component bug should usually be fixed in the component.

Avoid global overrides like:

```css
button {
  margin-top: 12px;
}
```

because one button needed spacing.

---

## Avoid Duplicate Rules

Before adding a rule, search for existing styles that already solve the problem.

Do not create:

```css
.center-content
.flex-center
.centered-flex
.content-center
```

that all do the same thing.

Consolidate only when the abstraction is genuinely shared and consistent with the architecture.

---

## Avoid Premature DRY

Do not combine selectors merely because two components currently share three declarations.

Bad:

```css
.movie-card,
.user-card,
.settings-panel,
.dropdown,
.toast {
  border-radius: 12px;
  background: #111;
}
```

if those components are unrelated and may evolve independently.

Shared design tokens are often better than coupling selectors.

---

## Shared Tokens Over Shared Selectors

Prefer:

```css
.movie-card {
  border-radius: var(--surface-radius);
}

.user-card {
  border-radius: var(--surface-radius);
}
```

when the relationship is a design-system value rather than shared component behavior.

This reduces accidental coupling.

---

## Avoid Giant Utility Classes

Do not create classes that attempt to style an entire page:

```css
.super-container {
  display: flex;
  width: 100%;
  min-height: 100vh;
  padding: 20px;
  margin: 0 auto;
  background: ...;
  color: ...;
  border: ...;
  box-shadow: ...;
  overflow: ...;
  position: ...;
}
```

Break styles by actual responsibility when the component genuinely contains distinct concerns.

Do not fragment simple rules unnecessarily either.

---

## Shorthand Properties

Use shorthands when they are clear.

Good:

```css
margin: 1rem 0;
```

Be careful when shorthands unintentionally reset subproperties.

For example:

```css
background:
font:
border:
animation:
```

can reset values not explicitly specified.

Use longhands when partial control matters.

---

## `background`

Do not replace:

```css
background-color:
```

with:

```css
background:
```

unless you intend to reset background image, position, repeat, and related properties.

This can create subtle regressions.

---

## `border`

Likewise:

```css
border:
```

resets several border subproperties.

Use it intentionally.

---

## Transition Shorthand

Be explicit about transitioned properties.

Avoid broad shorthand that accidentally captures future style changes.

---

## `calc()`

Use `calc()` when values genuinely combine units or variables.

Good:

```css
height: calc(100dvh - var(--header-height));
```

Do not use `calc()` for arithmetic that can be simplified statically.

Bad:

```css
width: calc(100% - 0px);
```

---

## `min()`, `max()`, and `clamp()`

Use these for responsive sizing when they simplify breakpoints.

Example:

```css
font-size: clamp(1.5rem, 2vw + 1rem, 3rem);
```

Do not use complicated formulas merely to appear sophisticated.

Keep responsive math understandable.

---

## Fluid Typography

Fluid typography can be useful.

Do not make every font size fluid.

Headlines or major responsive spacing may benefit; body text often works well with stable accessible sizes.

---

## `ch`

Use `ch` for text-oriented measures such as line length.

Do not assume `1ch` is a precise width for arbitrary layout dimensions.

It reflects the width of the "0" glyph.

---

## `em` vs `rem`

Use `rem` for values that should track the root font size.

Use `em` when scaling relative to the component's own font size is desirable.

Do not mix units randomly.

---

## Percentage Heights

Remember percentage heights often require a definite parent height.

Do not repeatedly add:

```css
height: 100%;
```

through ancestor chains without understanding the containing block.

Use flex/grid sizing where appropriate.

---

## `100vw`

Be cautious with:

```css
width: 100vw;
```

because it may include scrollbar width and create horizontal overflow.

For normal block content:

```css
width: 100%;
```

is often more appropriate.

---

## `box-sizing`

Understand whether borders and padding are included in sizing calculations.

A global `border-box` setup simplifies most component sizing.

Do not compensate with mysterious `calc()` expressions if box sizing is the actual issue.

---

## Floats

Use floats primarily for their intended text-wrapping behavior around media.

Do not build modern page layouts with floats unless maintaining legacy code.

---

## `clear`

Likewise, avoid clearfix hacks in new layout code when Flexbox/Grid solve the problem.

Do not refactor legacy float layout without a reason if it already works and is outside the task.

---

## `display: contents`

Use carefully.

It can simplify layout wrappers, but historically had accessibility inconsistencies.

Check browser requirements before relying on it for semantically meaningful elements.

Do not use it simply to erase every wrapper.

---

## `display: none`

Use it when content should genuinely be removed from layout and accessibility.

Do not toggle essential content visibility with CSS alone when application state also needs semantic updates.

---

## Visibility Transitions

You cannot smoothly transition `display`.

For animated visibility, use appropriate combinations of:

```css
opacity
visibility
transform
```

and application state.

Do not build fragile hacks when a simple immediate state change is acceptable.

---

## Modals and Overlays

CSS for dialogs should account for:

- Viewport sizing.
- Scrolling.
- Layering.
- Mobile layout.
- Safe margins.
- Content overflow.

Avoid fixed modal dimensions that fail on smaller screens.

Example:

```css
.dialog {
  width: min(32rem, calc(100vw - 2rem));
  max-height: calc(100dvh - 2rem);
  overflow: auto;
}
```

Use project tokens where available.

---

## Sticky Positioning

Use:

```css
position: sticky;
```

when an element should remain within its scroll container.

Do not replace sticky behavior with JavaScript unless requirements exceed what CSS can provide.

Remember sticky can fail because of ancestor overflow or layout constraints.

Diagnose those before abandoning it.

---

## Fixed Positioning

Use fixed positioning for true viewport-level UI such as:

- Modals.
- Floating action controls.
- Toasts.

Do not use fixed positioning to hold ordinary headers if sticky behavior is more appropriate.

---

## Safe Areas

For fullscreen/mobile interfaces that may run on notched devices, consider:

```css
env(safe-area-inset-top)
```

and related values when relevant.

Do not add safe-area padding to ordinary desktop pages without need.

---

## CSS and JavaScript

Prefer CSS for presentation and layout.

Do not use JavaScript to calculate styles CSS can express through:

- Media queries.
- Container queries.
- Flexbox.
- Grid.
- `clamp()`.
- `min()`.
- `max()`.
- `aspect-ratio`.

Use JavaScript when behavior depends on information CSS cannot reasonably express.

---

## Avoid Inline Style Mutation for State

Bad:

```js
element.style.backgroundColor = "red";
element.style.opacity = "0.5";
element.style.transform = "scale(0.95)";
```

when a class or data state could represent the state.

Prefer:

```js
element.dataset.state = "error";
```

with CSS:

```css
[data-state="error"] {
  ...
}
```

when that fits the architecture.

---

## Avoid CSS That Depends on JavaScript Timing

Do not design critical layout around arbitrary timeout-based class changes.

Use real component state and transitions.

Avoid brittle sequences such as:

```text
add class
wait 300ms
remove element
```

unless animation lifecycle is intentionally coordinated.

---

## `@property`

Use custom property registration when typed or animatable custom properties provide genuine benefit.

Do not introduce it for ordinary variables.

It is an advanced tool, not a default requirement.

---

## `@scope`

Use CSS scoping features when supported by the project's browser target and when they simplify component boundaries.

Do not use cutting-edge syntax without checking support.

---

## Subgrid

Use `subgrid` when child alignment genuinely needs to inherit parent grid tracks and browser support is acceptable.

Do not force it into simple layouts.

---

## Masonry

Do not rely on experimental CSS masonry features without verifying production support.

Use established layout solutions appropriate to the project's target browsers.

---

## Avoid Browser-Specific Hacks

Do not add selectors like:

```css
@supports (-webkit-touch-callout: none) {
  ...
}
```

unless fixing a verified browser issue.

Document why the workaround exists.

Remove hacks when they are no longer necessary.

---

## Workarounds

When unusual CSS exists to handle a browser bug, explain the reason briefly.

Good:

```css
/* Safari needs a definite min-height here for the nested scroller to shrink. */
.panel-body {
  min-height: 0;
}
```

This is useful because the declaration otherwise appears arbitrary.

---

## Avoid Screenshot-Driven CSS

Do not position everything using exact coordinates to match one reference image.

CSS should reproduce the design while remaining responsive to real content.

Avoid:

```css
left: 147px;
top: 83px;
width: 426px;
```

for ordinary page structure.

Use layout systems first.

---

## Pixel Perfection

Pixel accuracy can matter, but maintainability and responsiveness still matter.

Match intentional values such as:

- Radius.
- Spacing.
- Typography.
- Alignment.

Do not reproduce accidental screenshot artifacts.

---

## Design Consistency

Before inventing a new visual treatment, inspect neighboring components.

Reuse:

- Existing button styles.
- Existing card surfaces.
- Existing border treatments.
- Existing hover states.
- Existing typography.
- Existing spacing.

Do not create a unique visual language for every new feature.

---

## Avoid Generic "Modern UI" Styling

Do not automatically add:

- Large gradients.
- Glassmorphism.
- Excessive blur.
- Pill-shaped everything.
- Huge shadows.
- Floating cards.
- Animated glows.

These are common AI-generated defaults.

Match the product's actual design language instead.

---

## Avoid Over-Decoration

Every:

- Border.
- Shadow.
- Gradient.
- Blur.
- Background.
- Animation.

should serve hierarchy or interaction.

Do not add decoration simply because a component looks "plain."

Simple interfaces can be professional.

---

## Avoid Unnecessary `transition`

Do not add transitions to properties that never change.

Avoid copying:

```css
transition: all 0.2s ease;
```

onto every component.

Transitions should correspond to actual interactive states.

---

## Avoid Repeated Overrides

If a stylesheet repeatedly overrides itself:

```css
.card { padding: 16px; }
.card { padding: 20px; }
.card { padding: 18px; }
```

consolidate the rule when safe.

Do not leave historical override chains after finishing a focused refactor.

---

## Avoid Dead CSS

Remove styles that are no longer used when you can verify they are dead.

Do not delete classes simply because they are not present in one HTML file.

Check:

- JavaScript.
- Templates.
- Dynamic class construction.
- Framework components.
- Tests.
- Other pages.

Be careful with dynamically generated selectors.

---

## Avoid Global Fixes for Local Problems

Bad:

```css
img {
  max-height: 300px;
}
```

because one poster image was too tall.

Prefer:

```css
.movie-card__poster {
  ...
}
```

Keep fixes scoped to the component that owns the problem.

---

## Avoid `overflow: hidden` as a Universal Fix

Do not solve:

- Layout bugs.
- Border radius problems.
- Float clearing.
- Scrolling.
- Animation clipping.

all with `overflow: hidden`.

It can create unintended clipping and new formatting contexts.

Use the property for intentional overflow behavior.

---

## Avoid Negative Margins by Default

Negative margins can be legitimate.

Do not use them as the first solution to ordinary alignment problems.

Check:

- Parent padding.
- Grid tracks.
- Gap.
- Container structure.

before introducing offset hacks.

---

## Avoid Transform-Based Layout

Do not use:

```css
transform: translateX(...)
```

to position static layout elements that should be handled by normal layout.

Transforms change visual position without changing document flow.

Use them for animation or deliberate visual offset.

---

## Avoid Magic Z-Indexes

Do not solve overlap by repeatedly increasing values:

```css
z-index: 10;
z-index: 100;
z-index: 999;
z-index: 99999;
```

Establish a clear stacking model.

---

## Avoid Huge Negative Z-Index

Be cautious with:

```css
z-index: -999;
```

Negative stacking can place elements behind ancestor backgrounds and make behavior confusing.

Use proper stacking contexts.

---

## Avoid `height: 100%` Chains

Do not apply `height: 100%` to multiple ancestors hoping one eventually creates the desired layout.

Use explicit flex/grid structure or viewport sizing when appropriate.

---

## Avoid Width Calculations Based on Sibling Guessing

Bad:

```css
width: calc(100% - 237px);
```

because a sidebar happens to be 237px.

Prefer Grid:

```css
.layout {
  display: grid;
  grid-template-columns: 15rem minmax(0, 1fr);
}
```

or another explicit layout system.

---

## Avoid `calc()` for Known Grid Relationships

If a layout is naturally columns, use Grid/Flex instead of manual subtraction.

CSS layout primitives are more robust.

---

## Avoid Fixed Card Counts

Do not hard-code:

```css
grid-template-columns: repeat(5, 1fr);
```

for all screen widths unless the design intentionally requires exactly five columns.

For responsive content, consider:

```css
repeat(auto-fill, minmax(...))
```

or breakpoint-based layouts.

Use the design requirements, not a generic formula.

---

## `auto-fill` vs `auto-fit`

Understand the difference.

Use whichever better matches the intended empty-track behavior.

Do not switch between them randomly.

---

## Grid Minimums

Use:

```css
minmax(0, 1fr)
```

when grid children need to shrink below intrinsic content width.

This can prevent unexpected overflow.

Do not add it everywhere without understanding why.

---

## Flex Shrinking

Be deliberate with:

```css
flex: 1;
flex-shrink: 0;
flex-basis:
```

Do not blindly apply:

```css
flex: 1 1 0;
```

to every child.

Choose the sizing model the layout requires.

---

## `flex: 1`

Understand that `flex: 1` commonly expands to behavior equivalent to:

```css
flex: 1 1 0%;
```

This may differ from:

```css
flex: 1 1 auto;
```

Use the one matching desired intrinsic sizing.

---

## `margin-inline: auto`

Use auto margins for alignment when appropriate.

Example:

```css
.actions {
  margin-inline-start: auto;
}
```

This is often simpler than adding extra wrapper elements.

---

## Order Property

Avoid using:

```css
order:
```

to substantially reorder semantic content.

Visual order can diverge from keyboard and reading order.

Use DOM order that makes sense first.

---

## CSS Generated Counters

Use counters when numbering is a presentational concern tied to document structure.

Do not use CSS counters for meaningful data that should exist in HTML or application state.

---

## List Markers

Use:

```css
::marker
```

for styling list markers when appropriate.

Do not replace semantic lists with custom spans merely to style bullets.

---

## Hyphenation

Use:

```css
hyphens: auto;
```

where language and browser behavior make it useful.

Do not globally enable it without testing readability.

---

## Text Balance

Modern properties such as:

```css
text-wrap: balance;
text-wrap: pretty;
```

can improve headings and prose where supported.

Use them as progressive enhancement.

Do not rely on them for critical layout.

---

## Color Functions

Use modern color functions when they simplify the design system and browser support permits.

Examples:

```css
color-mix()
oklch()
```

Do not rewrite the entire palette into a newer color space during an unrelated task.

---

## Opacity Tokens

Prefer semantic translucent color tokens when commonly reused.

Do not repeatedly invent:

```css
rgba(255, 255, 255, 0.07)
rgba(255, 255, 255, 0.08)
rgba(255, 255, 255, 0.09)
```

without a design reason.

---

## `filter`

Use filters deliberately.

Avoid:

```css
filter: brightness(...)
```

as a generic substitute for proper hover colors when explicit colors are part of the design system.

Filters can alter images and child content unexpectedly.

---

## Blend Modes

Use blend modes only when intentional visual composition requires them.

They can create difficult-to-predict results across backgrounds.

Do not use them as decoration by default.

---

## CSS Masks

Masks can be useful for monochrome icons or effects.

Do not replace straightforward SVG or background images with complex masking without a reason.

---

## Browser Defaults

Keep useful browser defaults when they work.

Do not reset every element and then rebuild all behavior manually.

Native controls and typography defaults can provide solid accessibility baselines.

---

## Progressive Enhancement

Base styling should remain usable.

Layer newer visual enhancements on top where practical.

Do not make basic readability depend on a cutting-edge CSS feature.

---

## Print and Reduced Motion

Respect environment-specific user needs without turning every stylesheet into dozens of special-mode overrides.

Add modes where the application genuinely benefits.

---

## Stylelint

If the project uses Stylelint, respect its configuration.

Do not disable rules merely to make generated CSS pass.

Avoid broad:

```css
/* stylelint-disable */
```

unless there is a well-understood reason.

Prefer narrowly scoped disables when necessary.

---

## Prettier

If Prettier formats CSS, let it.

Do not manually align declarations or selectors in ways the formatter will undo.

Avoid formatting churn.

---

## Ordering Rules

If the project uses a property-ordering convention, follow it.

Do not impose alphabetical ordering or another method on files that use a different established style.

Consistency matters more than the ordering philosophy.

---

## Vendor Prefixes

Let the build pipeline manage prefixes when possible.

Do not manually add old prefixes everywhere.

Keep only prefixes required by actual browser targets.

---

## Minification

Do not write source CSS as minified code.

Minification belongs in production build output.

Source should remain readable.

---

## Source Maps

Do not disable source maps casually when they are useful for debugging and the build supports them.

Likewise, do not expose them in production environments where policy forbids it.

This is a build concern, not a component styling shortcut.

---

## Refactoring CSS

When explicitly asked to refactor:

- Preserve visual behavior unless changes are requested.
- Preserve class names used by markup or JavaScript.
- Preserve responsive behavior.
- Preserve theme behavior.
- Preserve accessibility states.
- Remove real duplication.
- Lower specificity where safe.
- Consolidate duplicate overrides.
- Avoid simultaneous design changes.

A refactor should make the stylesheet easier to reason about, not merely different.

---

## Preserve JavaScript Hooks

Before renaming or removing classes, check whether JavaScript or tests use them.

CSS classes sometimes serve multiple roles.

If the project uses `data-*` for behavior, keep styling and behavior hooks separate.

Do not assume a selector is purely presentational.

---

## Preserve Dynamic Classes

Search for classes constructed dynamically.

For example:

```js
`status-${state}`
```

may produce CSS selectors that never appear literally in markup.

Do not delete them as dead code without verifying runtime usage.

---

## Do Not Rewrite Working CSS Unnecessarily

When modifying an existing component:

1. Identify the smallest responsible rule.
2. Understand the cascade.
3. Check surrounding design tokens.
4. Preserve unrelated styles.
5. Avoid global changes.
6. Test relevant responsive states.
7. Verify hover, focus, active, disabled, and dark-mode behavior where applicable.

Do not rewrite an entire stylesheet because one spacing value is wrong.

---

## Avoid Premature Abstraction

Do not create a generic design framework for one page.

Avoid unnecessary:

```text
utility systems
mixins
token layers
component generators
layout frameworks
helper classes
```

when existing CSS already handles the project cleanly.

Add abstractions after real patterns appear.

---

## Avoid Fake Reusability

Do not create generic classes like:

```css
.flex-centered-rounded-shadowed-box {}
```

merely because several properties occur together once.

Reusable CSS should represent stable design or layout concepts.

---

## Avoid Excessive Utility Classes in Plain CSS

Bad:

```css
.mt-7 {}
.p-13 {}
.w-427 {}
.fs-17 {}
```

in a project that does not use a generated utility framework.

If utility classes are desired, use a coherent utility system rather than manually inventing dozens of one-off classes.

---

## Avoid AI-Looking CSS

Common generated-code smells include:

```text
transition: all
z-index: 9999
overflow: hidden on body
position: relative everywhere
display: flex everywhere
justify-content: center everywhere
backdrop-filter on every card
large gradients
large shadows
random border radii
random spacing values
duplicate media queries
!important chains
```

None of these properties are inherently wrong.

The problem is using them by default without connection to actual layout or design requirements.

---

## Avoid "Modernize Everything" Changes

Do not rewrite:

- Flexbox into Grid.
- Pixels into rem.
- Hex colors into OKLCH.
- Media queries into container queries.
- Sass into native CSS.
- Existing naming conventions into BEM.

unless the task calls for such migration.

Modern features are tools, not automatic upgrades.

---

## Avoid Screenshot Hacks

Do not solve visual mismatch through:

```css
top: -3px;
left: 2px;
transform: translateY(1px);
```

before checking the actual typography, line height, alignment, and container layout.

Tiny offsets are sometimes legitimate.

They should not compensate for a misunderstood layout system.

---

## Avoid Pixel Drift

When neighboring elements should align, derive them from shared layout constraints rather than manually assigning slightly different positions.

Use:

- Shared grid tracks.
- Common padding.
- Shared line heights.
- Alignment properties.

Do not align by eye with unrelated margins.

---

## Before Finishing

Review the change and remove or correct:

- Unnecessary `!important`.
- Over-specific selectors.
- Deep selector nesting.
- Duplicate rules.
- Dead declarations.
- Arbitrary values that should use tokens.
- Generic AI-style class names.
- Unnecessary wrappers required only by poor CSS.
- `transition: all`.
- Random z-index values.
- Global overflow suppression.
- Fixed dimensions that break responsive content.
- Unnecessary absolute positioning.
- Excessive blur or shadows.
- Decorative gradients that do not fit the design.
- Missing hover/focus/active states where required.
- Focus suppression.
- Low-contrast text.
- Duplicate breakpoint logic.
- Unnecessary media queries.
- Browser hacks without explanation.
- Debugging outlines.
- Placeholder styles.
- Unrelated formatting churn.

Then run the project's existing checks where available.

Typical checks may include:

```bash
npm run lint
npm run build
```

or tools such as:

```bash
stylelint "**/*.css"
prettier --check .
```

Do not assume these exact commands exist.

Inspect:

```text
package.json
stylelint config
build scripts
CI configuration
```

and use the project's established workflow.

The final CSS should look like it naturally belongs in the repository rather than like a generic AI-generated styling pass.

It should feel like CSS written by an experienced frontend developer: restrained, responsive, low-specificity, design-system-aware, accessible, and built around the cascade rather than constantly fighting it.