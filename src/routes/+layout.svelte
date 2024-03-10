<script lang="ts">
	import '../app.postcss';
</script>

<svelte:head>
	<title>s-visual-g</title>

	<meta content="s-visual-g" property="og:title" />
	<meta content="Simple SVG path visualizer & editor" property="og:description" />
	<meta name="theme-color" content="#faa33fd8" />
</svelte:head>

<div class="grid grid-cols-3 *:h-full *:overflow-y-scroll">
	<slot />
</div>
