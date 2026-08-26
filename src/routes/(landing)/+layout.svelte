<script lang="ts">
  import LandingFooter from "$lib/page/landing/general/LandingFooter.svelte";
  import LandingHeader from "$lib/page/landing/general/LandingHeader.svelte";
    import "../layout.css"
    let {children} = $props()
</script>

<LandingHeader/>
{@render children()}
<LandingFooter/>
