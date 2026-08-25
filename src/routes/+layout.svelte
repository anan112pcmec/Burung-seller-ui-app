<script lang="ts">
	import './layout.css';
  import { onMount } from 'svelte';
  import { fade } from 'svelte/transition';

	let { children } = $props();
	let mounted: boolean = $state<boolean>(false);

	onMount(() => {
		mounted = true;
	})
</script>

<svelte:head>
	<link rel="icon"  />
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="">
    <link href="https://fonts.googleapis.com/css2?family=Jost:ital,wght@0,100..900;1,100..900&family=Zen+Dots&display=swap" rel="stylesheet">
</svelte:head>

{#if mounted}
<div in:fade={{duration: 800}}>
{@render children()}	
</div>
{/if}


