<script lang="ts">
	import { onMount } from 'svelte';
	import preset from 'canvas-preset';
	let circles: { [index: string ]: { x: number, y: number, connections: string[] } } = {
		a: { x: 300, y: 300, connections: ['c', 'b'] },
		b: { x: 100, y: 100, connections: ['a', 'd'] },
		c: { x: 100, y: 300, connections: ['d', 'b'] },
		d: { x: 300, y: 100, connections: ['a', 'c'] }
    };
    const connections = Object.entries(circles).map(([key, { connections, x, y }]) => {
        return connections.map(name => {
            return {
                thickness: 1,
                color: "red",
                group: [
                    { x, y },
                    { x: circles[name].x, y: circles[name].y }
                ],
                name: key
            }
        });
    }).flat()
    const flattenedCircles = Object.values(circles);

	onMount(() => {
		const { c, draw, renderGroup, size } = preset(null, '#problemTypesHypergraph');

		size();
		draw(() => {
            renderGroup('lines', connections);
			renderGroup('arc', flattenedCircles, ({ x, y }: { x: number; y: number }) => {
				return {
					x,
					y,
					color: 'black',
					radius: 50
				};
			});
		});
	});
</script>

<canvas id="problemTypesHypergraph" />
