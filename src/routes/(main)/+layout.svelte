<script lang="ts">
  import OuterHeader from '$lib/page/general/OuterHeader.svelte';
  import OuterSide from '$lib/page/general/OuterSide.svelte';
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
<div class="grid grid-rows-[14%_86%]" in:fade={{duration: 800}}>
	<div>
		<OuterHeader/>
	</div>
	<div class="flex">
		<div class="flex-1">
			<OuterSide/>
		</div>
		<div class="flex-1">{@render children()}</div>
	</div>
	
</div>
{/if}


