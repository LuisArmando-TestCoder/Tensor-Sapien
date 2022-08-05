<script lang="ts">
	import { onMount } from 'svelte';
	interface Position {
		x: number;
		y: number;
	}

	let elements: { [index: string]: { connections: string[] } } = {
		a: { connections: ['c', 'b'] },
		b: { connections: ['a', 'd'] },
		c: { connections: ['d', 'b'] },
		d: { connections: ['a', 'c'] }
	};
	let nodes: {
		key: string;
		position: Position;
	}[] = [Object.entries(elements)[0]].map(([key]) => ({
		key,
		position: { x: 0, y: 0 }
	}));
	const lines: { _1: Position; _2: Position }[] = [];
	const center: Position = { x: 0, y: 0 };
	let lastPosition: Position | void;
	let lastCenter = {...center};
	let nodesCanvas: HTMLUnknownElement;

	onMount(() => {
		// nodesCanvas
		window.addEventListener('mousedown', (event) => {
			lastPosition = {
				x: event.clientX - lastCenter.x,
				y: event.clientY - lastCenter.y,
			};
		}, 
		// false
		);
		window.addEventListener('mousemove', (event) => {
			if (!lastPosition) return;

			center.x = event.clientX - (lastPosition as Position).x;
			center.y = event.clientY - (lastPosition as Position).y;
			lastCenter.x = center.x;
			lastCenter.y = center.y;
		});
		window.addEventListener('mouseup', () => {
			lastPosition = undefined;
		});
	});
</script>

<section class="nodes-hypergraph">
	<div class="lines">
		<svg>
			{#each lines as line}
				<line
					x1={line._1.x}
					y1={line._1.y}
					x2={line._2.x}
					y2={line._2.y}
					style="stroke: rgb(255, 255, 255); stroke-width: 2;"
				/>
			{/each}
		</svg>
	</div>
	<div class="nodes" bind:this={nodesCanvas}>
		{#each nodes as node}
			<div
				class="node"
				style={`
					left: calc(${node.position.x + (center?.x ?? 0)}px + 50vw - 25px);
					top: calc(${node.position.y + (center?.y ?? 0)}px + 50vh - 25px);
				`}
			>
				<p>{node.key}</p>
			</div>
		{/each}
	</div>
</section>

<style>
	.nodes-hypergraph {
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		position: fixed;
		background: #333;
	}
	.nodes {
		position: absolute;
		top: 0;
		left: 0;
		cursor: pointer;
	}
	.node {
		position: absolute;
		border-radius: 50%;
		display: grid;
		width: 50px;
		height: 50px;
		place-items: center;
		text-align: center;
		background: #111;
		color: #bbb;
		font-family: monospace;
		cursor: normal;
		user-select: none;
	}
</style>
