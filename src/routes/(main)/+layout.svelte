<script lang="ts">
  import NavHeader from '$lib/page/main/general/NavHeader.svelte';
  import NavSide from '$lib/page/main/general/NavSide.svelte';
	import '../layout.css';
  import { onMount } from 'svelte';
  import { fade } from 'svelte/transition';

	let { children } = $props();
	let mounted: boolean = $state<boolean>(false);

	onMount(() => {
		mounted = true;
	})
</script>

{#if mounted}
<div class="h-screen">
<NavHeader/>
<NavSide>{@render children?.()}</NavSide>
</div>
{/if}

