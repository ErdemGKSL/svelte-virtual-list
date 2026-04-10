<script lang="ts">
	import { VirtualList } from '../lib/index.js';

	type DemoItem = {
		id: number;
		title: string;
		description: string;
		accent: 'amber' | 'cyan' | 'emerald' | 'rose';
		height: number;
	};

	let total = $state(1000);
	let fixedHeight = $state(false);
	let compact = $state(false);
	let start = $state(0);
	let end = $state(0);

	let viewportHeight = $derived(compact ? '24rem' : '36rem');
	let itemHeight = $derived(fixedHeight ? (compact ? 44 : 56) : undefined);

	let items = $derived.by<DemoItem[]>(() => {
		const baseHeight = compact ? 44 : 56;
		const accents = ['amber', 'cyan', 'emerald', 'rose'] as const;

		return Array.from({ length: total }, (_, index) => {
			const accent = accents[index % accents.length];
			const variableHeight = baseHeight + (index % 5) * 14;

			return {
				id: index,
				title: `Row ${index + 1}`,
				description: `Virtualized item ${index + 1} of ${total}`,
				accent,
				height: fixedHeight ? baseHeight : variableHeight
			};
		});
	});
</script>

<svelte:head>
	<title>Svelte Virtual List Demo</title>
</svelte:head>

<div class="page">
	<section class="intro">
		<p class="eyebrow">Svelte 5 migration demo</p>
		<h1>Virtual list with rune-based internals</h1>
		<p class="lede">
			The scrolling logic is unchanged. This page exercises the Svelte 5 component through
			bindable visible indices and optional fixed row heights.
		</p>
	</section>

	<section class="panel controls">
		<label>
			<span>Total items</span>
			<input type="range" min="100" max="5000" step="100" bind:value={total} />
			<strong>{total}</strong>
		</label>

		<label class="toggle">
			<input type="checkbox" bind:checked={fixedHeight} />
			<span>Use fixed item height</span>
		</label>

		<label class="toggle">
			<input type="checkbox" bind:checked={compact} />
			<span>Compact viewport</span>
		</label>
	</section>

	<section class="panel stats">
		<div>
			<span>Visible range</span>
			<strong>{start} - {Math.max(start, end - 1)}</strong>
		</div>
		<div>
			<span>Rendered items</span>
			<strong>{Math.max(end - start, 0)}</strong>
		</div>
		<div>
			<span>Mode</span>
			<strong>{fixedHeight ? `fixed ${itemHeight}px` : 'measured rows'}</strong>
		</div>
	</section>

	<section class="panel list-shell">
		<VirtualList items={items} height={viewportHeight} {itemHeight} bind:start bind:end>
			{#snippet row(item: DemoItem)}
				<article class:fixed={fixedHeight} class="row row-{item.accent}" style:height="{item.height}px">
					<div>
						<p class="title">{item.title}</p>
						<p class="description">{item.description}</p>
					</div>
					<span>{item.height}px</span>
				</article>
			{/snippet}
		</VirtualList>
	</section>
</div>

<style>
	:global(body) {
		margin: 0;
		font-family: 'Space Grotesk', 'Segoe UI', sans-serif;
		background:
			radial-gradient(circle at top left, rgba(15, 118, 110, 0.18), transparent 24rem),
			linear-gradient(180deg, #f3efe6 0%, #fbf8f2 45%, #f4efe7 100%);
		color: #1f2937;
	}

	.page {
		max-width: 72rem;
		margin: 0 auto;
		padding: 3rem 1.5rem 4rem;
		display: grid;
		gap: 1rem;
	}

	.intro {
		display: grid;
		gap: 0.75rem;
	}

	.eyebrow {
		margin: 0;
		text-transform: uppercase;
		letter-spacing: 0.18em;
		font-size: 0.78rem;
		color: #0f766e;
	}

	h1,
	.lede {
		margin: 0;
	}

	h1 {
		font-size: clamp(2.5rem, 6vw, 4.75rem);
		line-height: 0.95;
		max-width: 12ch;
	}

	.lede {
		max-width: 56rem;
		font-size: 1.05rem;
		line-height: 1.6;
		color: #475569;
	}

	.panel {
		background: rgba(255, 255, 255, 0.78);
		backdrop-filter: blur(10px);
		border: 1px solid rgba(148, 163, 184, 0.22);
		border-radius: 1.5rem;
		box-shadow: 0 20px 45px rgba(15, 23, 42, 0.08);
	}

	.controls {
		display: flex;
		flex-wrap: wrap;
		gap: 1rem 1.5rem;
		padding: 1.25rem;
		align-items: center;
	}

	.controls label {
		display: flex;
		gap: 0.75rem;
		align-items: center;
		color: #334155;
	}

	.controls input[type='range'] {
		width: min(18rem, 45vw);
	}

	.toggle {
		padding: 0.7rem 1rem;
		border-radius: 999px;
		background: rgba(226, 232, 240, 0.75);
	}

	.stats {
		display: grid;
		grid-template-columns: repeat(3, minmax(0, 1fr));
		gap: 1rem;
		padding: 1.25rem;
	}

	.stats div {
		display: grid;
		gap: 0.35rem;
	}

	.stats span {
		font-size: 0.82rem;
		text-transform: uppercase;
		letter-spacing: 0.08em;
		color: #64748b;
	}

	.stats strong {
		font-size: 1.4rem;
		font-weight: 600;
	}

	.list-shell {
		padding: 1rem;
	}

	.row {
		display: flex;
		justify-content: space-between;
		align-items: center;
		gap: 1rem;
		padding: 1rem 1.1rem;
		border-radius: 1rem;
		margin-bottom: 0.75rem;
		border: 1px solid rgba(255, 255, 255, 0.9);
		box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.65);
	}

	.row.fixed {
		align-items: center;
	}

	.row-amber {
		background: linear-gradient(135deg, rgba(251, 191, 36, 0.18), rgba(255, 251, 235, 0.92));
	}

	.row-cyan {
		background: linear-gradient(135deg, rgba(34, 211, 238, 0.2), rgba(236, 254, 255, 0.92));
	}

	.row-emerald {
		background: linear-gradient(135deg, rgba(16, 185, 129, 0.18), rgba(236, 253, 245, 0.92));
	}

	.row-rose {
		background: linear-gradient(135deg, rgba(244, 63, 94, 0.16), rgba(255, 241, 242, 0.92));
	}

	.title,
	.description {
		margin: 0;
	}

	.title {
		font-size: 1.05rem;
		font-weight: 700;
	}

	.description {
		margin-top: 0.2rem;
		color: #475569;
	}

	@media (max-width: 720px) {
		.page {
			padding-inline: 1rem;
		}

		.stats {
			grid-template-columns: 1fr;
		}

		.controls {
			align-items: stretch;
		}

		.controls label {
			flex-wrap: wrap;
		}

		.controls input[type='range'] {
			width: 100%;
		}

		.row {
			align-items: flex-start;
			flex-direction: column;
		}
	}
</style>
