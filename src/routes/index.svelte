<canvas id="problemTypesHypergraph"></canvas>

<script lang="TypeScript">
    import { onMount } from 'svelte';
    import preset from 'canvas-preset';
  
    onMount(() => {
        const {
            canvas,
            draw,
            render,
            size
        } = preset(null, "#problemTypesHypergraph");

        size();
        draw(() => {
            render({
                x: 0, y: 0, width: 100, height: 100, color: "#0f0"
            }).rect();
        });
    });
</script>