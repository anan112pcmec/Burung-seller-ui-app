<script lang="ts">
  import { onMount } from "svelte";
    import "../layout.css"
  import NavHeader from "$lib/page/main/general/NavHeader.svelte";

    let { children } = $props();
	let mounted: boolean = $state<boolean>(false);

	onMount(() => {
		mounted = true;
	})
</script>


{#if mounted}
<div class="h-screen">
<NavHeader/>
{@render children?.()}
</div>
{/if}