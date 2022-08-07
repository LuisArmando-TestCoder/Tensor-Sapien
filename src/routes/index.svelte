<script lang="ts">
	import SvelteMarkdown from 'svelte-markdown';

	const source = `# a`;

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
		abbbbccccccccc: { connections: ['c', 'b'] },
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
		window.addEventListener('mousedown', function setLastPostion(event) {
			lastPosition = {
				x: event.clientX - lastCenter.x,
				y: event.clientY - lastCenter.y
			};
		});
		window.addEventListener('mousemove', function repositionCenter(event) {
			if (!lastPosition) return;

			center.x = event.clientX - (lastPosition as Position).x;
			center.y = event.clientY - (lastPosition as Position).y;
			lastCenter.x = center.x;
			lastCenter.y = center.y;
		});
		window.addEventListener('mouseup', function resetPosition() {
			lastPosition = undefined;
		});
		window.addEventListener('click', function setNodeAsVisible(event: MouseEvent) {
			const element = event?.target as HTMLElement;
			if (element.classList.contains('node')) {
				const node = nodes.find(({ key }) => {
					return key === element.dataset.key;
				});

				if (node) node.state = 'visible';

				nodes = [...nodes];
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

<section class="node-info">
	<div class="node-info__wrapper">
		<SvelteMarkdown {source}/>
	</div>
</section>

<style>
	:root {
		--color-silver: #bbb;
		--color-mine-shaft: #333;
		--color-cod-gray: #111;
		--color-white: #fff;
		--color-gorse: #fffa73;
	}
	.nodes-hypergraph {
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background: var(--color-mine-shaft);
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
		background: var(--color-mine-shaft);
		color: var(--color-silver);
		font-family: monospace;
		cursor: normal;
		user-select: none;
		transition: background 0.35s, color 0.35s, box-shadow 0.35s;
		z-index: 1;
		box-shadow: 0 0 20px var(--color-cod-gray);
	}
	.node:not(.visible):hover {
		background-color: var(--color-silver);
		color: var(--color-mine-shaft);
		box-shadow: 0 0 10px var(--color-cod-gray);
	}
	.node:not(.visible):active {
		background-color: var(--color-white);
		box-shadow: 0 0 40px var(--color-white);
	}
	.node.visible:hover {
		box-shadow: 0 0 10px var(--color-gorse);
	}
	.node.visible:active {
		box-shadow: 0 0 40px var(--color-gorse);
	}
	.node p {
		pointer-events: none;
		max-width: 40px;
		text-overflow: ellipsis;
		overflow: hidden;
	}
	.seen {
		background: var(--color-silver);
		color: var(--color-mine-shaft);
	}
	.unseen {
		background: var(--color-mine-shaft);
		color: var(--color-silver);
	}
	.visible {
		background: var(--color-gorse);
		color: var(--color-mine-shaft);
		box-shadow: 0 0 20px var(--color-visible);
	}
	.node-info {
		width: clamp(100px, 50vw, 400px);
		height: 100vh;
		position: fixed;
		z-index: 1;
		top: 0;
		right: 0;
		background: var(--color-cod-gray);
		overflow: auto;
		padding: 25px;
		box-sizing: border-box;
		box-shadow: 0 0 20px var(--color-cod-gray);
	}
	.node-info__wrapper {
		padding: 25px;
		color: var(--color-cod-ray);
		background: var(--color-silver);
		border-radius: 5px;
		height: 100%;
		box-sizing: border-box;
		font-family: monospace;
		box-shadow: inset 0 0 20px var(--color-white);
	}
</style>
