<script lang="ts">
	import './layout.css';
	import favicon from '$lib/favicon/favicon.ico';
	import PenguinImage from '$lib/Penguins_Windows_7.jpg';

	let { children } = $props();
</script>

<svelte:head><link rel="icon" href={favicon} /></svelte:head>
<main
	style:background-image={`url("${PenguinImage}")`}
	class="h-[dvh] w-full bg-cover bg-no-repeat"
>
	{@render children()}
</main>
