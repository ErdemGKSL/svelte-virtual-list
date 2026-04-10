<script lang="ts" generics="Item">
	import { onMount, tick, type Snippet } from 'svelte';

	let {
		items,
		height = '100%',
		itemHeight = undefined,
		start = $bindable(0),
		end = $bindable(0),
		row
	}: {
		items: Item[];
		height?: string;
		itemHeight?: number;
		start?: number;
		end?: number;
		row?: Snippet<[Item]>;
	} = $props();

	let height_map = $state<number[]>([]);
	let rows: HTMLCollectionOf<HTMLElement> | undefined;
	let viewport: HTMLElement | undefined;
	let contents: HTMLElement | undefined;
	let viewport_height = $state(0);
	let mounted = $state(false);

	let top = $state(0);
	let bottom = $state(0);
	let average_height = $state<number | undefined>(undefined);

	let visible = $derived(items.slice(start, end).map((data, i) => {
		return { index: i + start, data };
	}));

	$effect.pre(() => {
		if (mounted) refresh(items, viewport_height, itemHeight);
	});

	async function refresh(items: Item[], viewport_height: number, itemHeight: number | undefined) {
		if (!viewport || !rows) return;

		const { scrollTop } = viewport;

		await tick(); // wait until the DOM is up to date

		let content_height = top - scrollTop;
		let i = start;

		while (content_height < viewport_height && i < items.length) {
			let row = rows[i - start];

			if (!row) {
				end = i + 1;
				await tick(); // render the newly visible row
				row = rows[i - start];
			}

			const row_height = height_map[i] = itemHeight || row.offsetHeight;
			content_height += row_height;
			i += 1;
		}

		end = i;

		const remaining = items.length - end;
		average_height = (top + content_height) / end;

		bottom = remaining * average_height;
		height_map.length = items.length;

	}

	async function handle_scroll() {
		if (!viewport || !rows) return;

		const { scrollTop } = viewport;

		const old_start = start;

		for (let v = 0; v < rows.length; v += 1) {
			height_map[start + v] = itemHeight || rows[v].offsetHeight;
		}

		let i = 0;
		let y = 0;

		while (i < items.length) {
			const row_height = height_map[i] ?? average_height ?? 0;
			if (y + row_height > scrollTop) {
				start = i;
				top = y;

				break;
			}

			y += row_height;
			i += 1;
		}

		while (i < items.length) {
			y += height_map[i] ?? average_height ?? 0;
			i += 1;

			if (y > scrollTop + viewport_height) break;
		}

		end = i;

		const remaining = items.length - end;
		average_height = y / end;

		while (i < items.length) height_map[i++] = average_height;
		bottom = remaining * average_height;

		// prevent jumping if we scrolled up into unknown territory
		if (start < old_start) {
			await tick();

			let expected_height = 0;
			let actual_height = 0;

			for (let i = start; i < old_start; i +=1) {
				if (rows[i - start]) {
					expected_height += height_map[i];
					actual_height += itemHeight || rows[i - start].offsetHeight;
				}
			}

			const d = actual_height - expected_height;
			viewport.scrollTo(0, scrollTop + d);
		}

		// TODO if we overestimated the space these
		// rows would occupy we may need to add some
		// more. maybe we can just call handle_scroll again?
	}

	// trigger initial refresh
	onMount(() => {
		if (!contents) return;

		rows = contents.getElementsByTagName('svelte-virtual-list-row') as HTMLCollectionOf<HTMLElement>;
		mounted = true;
	});
</script>

<style>
	svelte-virtual-list-viewport {
		position: relative;
		overflow-y: auto;
		-webkit-overflow-scrolling:touch;
		display: block;
	}

	svelte-virtual-list-contents, svelte-virtual-list-row {
		display: block;
	}

	svelte-virtual-list-row {
		overflow: hidden;
	}
</style>

<svelte-virtual-list-viewport
	bind:this={viewport}
	bind:offsetHeight={viewport_height}
	onscroll={handle_scroll}
	style="height: {height};"
>
	<svelte-virtual-list-contents
		bind:this={contents}
		style="padding-top: {top}px; padding-bottom: {bottom}px;"
	>
		{#each visible as visibleRow (visibleRow.index)}
			<svelte-virtual-list-row>
				{#if row}
					{@render row(visibleRow.data)}
				{:else}
					Missing template
				{/if}
			</svelte-virtual-list-row>
		{/each}
	</svelte-virtual-list-contents>
</svelte-virtual-list-viewport>
