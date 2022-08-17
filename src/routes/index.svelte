<script lang="ts">
	import SvelteMarkdown from 'svelte-markdown';

	let source = '';
	let shouldCloseNodeInfo = true;

	import { onMount } from 'svelte';

	interface Position {
		x: number;
		y: number;
	}
	interface Node {
		state: State;
		key: string;
		content: string;
		connections: string[];
		position: Position;
	}

	type State = 'unseen' | 'seen' | 'visible';

	let elements: { [index: string]: { connections: string[]; content: string } } = {
		a: { connections: ['c', 'b', 'd'], content: 'hey' },
		b: { connections: [], content: 'a que' },
		c: { connections: [], content: 'no' },
		d: { connections: ['e', 'f'], content: 'creías que' },
		e: { connections: [], content: 'esto era' },
		f: { connections: ['g'], content: 'posible' },
		g: { connections: [], content: '# Bienvenido' },
	};
	let nodes: Node[] = [Object.entries(elements)[0]].map(([key, { content, connections }]) => ({
		state: 'unseen',
		key,
		content,
		connections,
		position: { x: 0, y: 0 }
	}));
	let lines: { _1: Position; _2: Position }[] = [];
	const center: Position = { x: 0, y: 0 };
	let lastPosition: Position | void;
	let lastCenter = { ...center };
	let nodesCanvas: HTMLUnknownElement;
	const falseWindow = {
		innerWidth: 0,
		innerHeight: 0
	};

	onMount(() => {
		falseWindow.innerWidth = window.innerWidth;
		falseWindow.innerHeight = window.innerHeight;

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
		(document.querySelector('.nodes-hypergraph') as HTMLElement).addEventListener(
			'mouseup',
			(event: MouseEvent) =>
				[
					function resetPosition() {
						lastPosition = undefined;
					},
					function closeNodeInfo() {
						const element = event?.target as HTMLElement;
						if (!element.classList.contains('node') || element.tagName === 'SVG') {
							shouldCloseNodeInfo = true;
						}
					}
				].forEach((functionValue) => functionValue())
		);
		window.addEventListener('click', function setNodeAsVisible(event: MouseEvent) {
			const element = event?.target as HTMLElement;
			if (element.classList.contains('node')) {
				const node = nodes.find(({ key }) => {
					return key === element.dataset.key;
				});

				if (node) node.state = 'visible';

				nodes = [...nodes];

				shouldCloseNodeInfo = false;

				if (typeof node?.key === 'string') {
					source = node?.content;

					const lastLinesLength = lines.length;

					populateNodeChildren(node);

					if (lastLinesLength != lines.length) {
						lines = [...lines];
					}
				}
			}
		});
	});

	function populateNodeChildren(node: Node) {
		node.connections.forEach((key, index) => {
			const { content, connections } = elements[key];

			if (!nodes.map(({ key }) => key).includes(key)) {
				const step = (index / node.connections.length) * Math.PI * 2;

				nodes.push({
					state: 'unseen',
					key,
					content,
					connections,
					position: {
						x: node.position.x + Math.sin(step) * 100,
						y: node.position.y + Math.cos(step) * 100
					}
				});

				lines.push({
					_1: {
						x: node.position.x,
						y: node.position.y
					},
					_2: {
						x: nodes[nodes.length - 1].position.x,
						y: nodes[nodes.length - 1].position.y
					}
				});
			}
		});
	}
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
				<g
					transform={`translate(${(center?.x ?? 0) + falseWindow.innerWidth / 2}, ${
						(center?.y ?? 0) + falseWindow.innerHeight / 2
					})`}
				>
					{#each lines as line}
						<line
							x1={line._1.x}
							y1={line._1.y}
							x2={line._2.x}
							y2={line._2.y}
							stroke="#f44"
							stroke-width="2"
						/>
					{/each}
				</g>
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

<section class={`node-info ${shouldCloseNodeInfo && 'right-hidden'}`}>
	<div class="node-info__wrapper">
		<SvelteMarkdown {source} />
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
		box-shadow: inset 0 0 40px var(--color-cod-gray);
		cursor: pointer;
	}
	.wrapper,
	.nodes-hypergraph {
		position: fixed;
	}
	.nodes {
		z-index: 1;
	}
	.nodes,
	.node {
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
		z-index: 2;
		top: 0;
		right: 0;
		background: var(--color-mine-shaft);
		padding: 25px;
		box-sizing: border-box;
		box-shadow: 0 0 20px var(--color-cod-gray);
		transition: right 1s;
	}
	.right-hidden {
		right: calc(clamp(100px, 50vw, 400px) * -1);
	}
	.node-info__wrapper {
		padding: 25px;
		color: var(--color-cod-ray);
		background: var(--color-white);
		border-radius: 5px;
		height: 100%;
		box-sizing: border-box;
		font-family: monospace;
		box-shadow: inset 0 0 20px var(--color-silver);
		overflow: auto;
	}
	.lines svg {
		width: 100vw;
		height: 100vh;
		top: 0;
		left: 0;
		position: fixed;
		z-index: 0;
	}
</style>
