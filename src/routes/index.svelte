<script lang="ts">
	import { onMount } from 'svelte';
	interface Position {
		x: number;
		y: number;
	}
	interface Node {
		state: State;
		key: string;
		position: Position;
	}

	type State = 'unseen' | 'seen' | 'visible';

	let elements: { [index: string]: { connections: string[] } } = {
		a: { connections: ['c', 'b'] },
		b: { connections: ['a', 'd'] },
		c: { connections: ['d', 'b'] },
		d: { connections: ['a', 'c'] }
	};
	let nodes: Node[] = [Object.entries(elements)[0]].map(([key]) => ({
		state: 'unseen',
		key,
		position: { x: 0, y: 0 }
	}));
	const lines: { _1: Position; _2: Position }[] = [];
	const center: Position = { x: 0, y: 0 };
	let lastPosition: Position | void;
	let lastCenter = { ...center };
	let nodesCanvas: HTMLUnknownElement;

	onMount(() => {
		// nodesCanvas
		window.addEventListener(
			'mousedown',
			(event) => {
				lastPosition = {
					x: event.clientX - lastCenter.x,
					y: event.clientY - lastCenter.y
				};
			}
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
		window.addEventListener('click', (event: PointerEvent) => {
			const element = event?.target as HTMLElement;
			if (element.classList.contains('node')) {
				const node = nodes.find(({ key }) => {
					return key === element.dataset.key;
				});

				if (node) node.key = 'seen';
			}
		});
	});
</script>

<section class="nodes-hypergraph">
	<div
		class="wrapper"
		style={`
		left: calc(${center?.x ?? 0}px + 50vw - 25px);
		top: calc(${center?.y ?? 0}px + 50vh - 25px);
	`}
	>
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
					data-key={node.key}
					class={'node ' + node.state}
					style={`
						left: ${node.position.x}px;
						top: ${node.position.y}px;
					`}
				>
					<p>{node.key}</p>
				</div>
			{/each}
		</div>
	</div>
</section>

<style>
	.nodes-hypergraph {
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background: #333;
		cursor: pointer;
	}
	.wrapper,
	.nodes-hypergraph {
		position: fixed;
	}
	.nodes,
	.node,
	.lines {
		position: absolute;
	}
	.node {
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
		transition: background 0.35s, color 0.35s, box-shadow 0.35s;
		z-index: 1;
		box-shadow: 0 0 20px;
	}
	.node:hover {
		background-color: #bbb;
		color: #111;
		box-shadow: 0 0 2px #bbb;
	}
	.node p {
		pointer-events: none;
	}
	.seen {
		opacity: 0.5;
	}
	.unseen {
		opacity: 1;
	}
	.visible {
		opacity: 1;
		background: #8f8f00;
	}
</style>
