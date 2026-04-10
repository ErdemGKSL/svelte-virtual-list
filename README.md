# svelte-virtual-list

A virtual list component for Svelte 5. Instead of rendering every item in a large array, `VirtualList` only renders the visible rows and pads the rest of the scroll area.

The current package is built as a Svelte 5 library and uses runes internally. The virtualization logic is unchanged from the original component, but the rendering API now uses Svelte 5 snippets.

## Installation

```bash
pnpm add svelte-virtual-list
```

You need Svelte 5 in the consuming app.

## Usage

```svelte
<script lang="ts">
  import { VirtualList } from 'svelte-virtual-list';

  let things = [
    { name: 'one', number: 1 },
    { name: 'two', number: 2 },
    { name: 'three', number: 3 }
  ];

  let start = $state(0);
  let end = $state(0);
</script>

<VirtualList items={things} bind:start bind:end>
  {#snippet row(item)}
    <p>{item.number}: {item.name}</p>
  {/snippet}
</VirtualList>

<p>showing {start}-{Math.max(start, end - 1)} of {things.length} rows</p>
```

## Props

### `items`

The array to virtualize.

### `height`

Defaults to `100%`. Pass any CSS length to control the viewport height.

```svelte
<VirtualList items={things} height="500px">
  {#snippet row(item)}
    <p>{item.number}: {item.name}</p>
  {/snippet}
</VirtualList>
```

### `itemHeight`

Optional fixed row height in pixels. If this is provided, initial measurement and scrolling can be more efficient because the component does not need to measure each rendered row.

```svelte
<VirtualList items={things} itemHeight={48}>
  {#snippet row(item)}
    <p>{item.number}: {item.name}</p>
  {/snippet}
</VirtualList>
```

### `start` and `end`

These values are bindable and reflect the currently rendered slice.

```svelte
<script>
  let start = $state(0);
  let end = $state(0);
</script>

<VirtualList items={things} bind:start bind:end>
  {#snippet row(item)}
    <p>{item.name}</p>
  {/snippet}
</VirtualList>
```

`start` is the index of the first visible row. `end` is the exclusive upper bound of the rendered range.

## Rendering rows

Rows are provided with a `row` snippet:

```svelte
<VirtualList items={things}>
  {#snippet row(item)}
    <article>
      <h2>{item.name}</h2>
      <p>#{item.number}</p>
    </article>
  {/snippet}
</VirtualList>
```

If no `row` snippet is supplied, the component renders `Missing template`.

## Package entry

The library exports a named component:

```ts
import { VirtualList } from 'svelte-virtual-list';
```

## Demo

The repository includes a SvelteKit showcase page in [src/routes/+page.svelte](src/routes/+page.svelte) that demonstrates:

- bindable `start` and `end`
- dynamic item counts
- fixed and measured row heights
- a snippet-based row renderer
