[![npm version](https://img.shields.io/npm/v/@itrocks/contained-auto-width?logo=npm)](https://www.npmjs.org/package/@itrocks/contained-auto-width)
[![npm downloads](https://img.shields.io/npm/dm/@itrocks/contained-auto-width)](https://www.npmjs.org/package/@itrocks/contained-auto-width)
[![GitHub](https://img.shields.io/github/last-commit/itrocks-ts/contained-auto-width?color=2dba4e&label=commit&logo=github)](https://github.com/itrocks-ts/contained-auto-width)
[![issues](https://img.shields.io/github/issues/itrocks-ts/contained-auto-width)](https://github.com/itrocks-ts/contained-auto-width/issues)
[![discord](https://img.shields.io/discord/1314141024020467782?color=7289da&label=discord&logo=discord&logoColor=white)](https://25.re/ditr)

# contained-auto-width

Automatically adjusts the width of <input> elements based on their text content within a list.

*This documentation was written by an artificial intelligence and may contain errors or approximations.
It has not yet been fully reviewed by a human. If anything seems unclear or incomplete,
please feel free to contact the author of this package.*

## Installation

```bash
npm i @itrocks/contained-auto-width
```

The package is published as an ES module. You can import it from any modern bundler or directly in the browser (with an ES module–aware loader).

## Usage

`@itrocks/contained-auto-width` exposes a single helper:

- `containedAutoWidth(container)` — keeps the width of an `<input>` in sync with its current text (or placeholder) by mirroring the text inside the surrounding container.

The helper does **not** depend on the rest of the `@itrocks` ecosystem and can be used on its own. In the `@itrocks/framework`, it is wired automatically using data‑attributes.

### Minimal example (stand‑alone)

HTML:

```html
<ul id="tags">
	<li>
		<input type="text" placeholder="+">
	</li>
</ul>
```

TypeScript / JavaScript:

```ts
import { containedAutoWidth } from '@itrocks/contained-auto-width'

const list = document.getElementById('tags') as HTMLUListElement

// Apply to each <li> so that the <input> expands/shrinks with its content
for (const li of Array.from(list.children)) {
	if (li instanceof HTMLElement) {
		containedAutoWidth(li)
	}
}
```

Now, as the user types in the input, the text content mirrored in the `<li>` grows or shrinks, and the list item width follows.

### Integrated example with `@itrocks/framework`

When you use `@itrocks/contained-auto-width` through `@itrocks/framework`, you typically do **not** call `containedAutoWidth` yourself. Instead, you use data‑attributes that the framework’s front‑end builder recognises.

#### Multiple inputs within a list

Generated HTML (simplified) for a collection of related objects might look like:

```html
<ul data-multiple-contained-auto-width data-fetch="/movies/summary" data-type="objects">
	<li>
		<input name="actors.1" value="Keanu Reeves">
		<input id="actors-id.1" name="actors_id.1" type="hidden" value="1">
	</li>
	<li>
		<input name="actors.2" value="Carrie-Anne Moss">
		<input id="actors-id.2" name="actors_id.2" type="hidden" value="2">
	</li>
	<li>
		<input name="actors" placeholder="+">
		<input id="actors-id" name="actors_id" type="hidden">
	</li>
</ul>
```

The framework initialisation contains the following line:

```ts
selector = '[data-contained-auto-width], [data-multiple-contained-auto-width] > li'
build<HTMLLIElement>(selector, async container => containedAutoWidth(container))
```

- On elements with `data-contained-auto-width`, the helper is applied directly.
- On elements inside a `data-multiple-contained-auto-width` list, each `<li>` gets `containedAutoWidth`, which keeps the width of the `<li>` aligned with the content of its first `<input>`.

This provides a compact, spreadsheet‑like editing experience where each list item grows just enough to display its current value.

## API

### `containedAutoWidth(container: HTMLElement): void`

Applies automatic width adjustment to a container that holds an editable `<input>`.

#### Parameters

- `container` — the HTML element that contains the input whose content should control the width. The function expects the **first element child** of `container` to be an `<input>` element.

#### Behaviour

- If `container` is an `<ol>` or `<ul>` element, `containedAutoWidth` recursively applies itself to all child elements that are `HTMLElement` instances, then returns. This allows you to call it once on a list and have it configured on each list item.
- For any other element:
	- Existing text‑node children of `container` are removed (only DOM nodes of type `Node.TEXT_NODE`). This avoids conflicting text when the mirrored content is added.
	- The **first element child** is treated as the target `<input>`.
	- A new `Text` node is created and appended to `container`. Its content is kept in sync with:
		- the current `value` of the input, when non‑empty, or
		- the `placeholder` of the input, when the value is empty, or
		- a single space character if both value and placeholder are empty.
	- `change` and `input` event listeners are attached to the `<input>` to update the mirrored text node whenever the user edits the field.

Because the text node sits next to the input inside the container, usual CSS layout rules (inline‑block, flex, etc.) can make the container’s width follow the text content.

#### Return value

The function returns `void` and is used for its side effects on the DOM.

#### Usage notes and constraints

- Call `containedAutoWidth` **after** the container and its input are present in the DOM (for example, on `DOMContentLoaded` or when a component is mounted).
- Ensure that the container’s **first element child** is the `<input>` you want to track; other structures are not supported.
- It is usually enough to call `containedAutoWidth` **once** per container; the attached event listeners keep it up to date.
- The helper assumes a browser environment with `document` and `Node` available.

## Typical use cases

- Editable lists of related entities, where each item is identified by a single text field (e.g. actors of a movie, tags, categories).
- Tag or chip editors where the “add new item” row shows a `+` placeholder and should expand while typing.
- Inline editing of simple collections in back‑office forms where you want compact rows that resize with user input.
- Custom list‑based widgets that need their item width to follow the text of their primary input without manual recalculation.
