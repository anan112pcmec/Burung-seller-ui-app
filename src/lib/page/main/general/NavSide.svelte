<script lang="ts">
  import type { Snippet } from "svelte";

  // Tangkap prop 'children' dengan tipe Snippet
  let { children }: { children?: Snippet } = $props();
</script>

<section class="grid grid-cols-[6%_94%] h-full">
    <aside class="bg-slate-600 h-full">
        Bhapp
    </aside>
    <div>
        {@render children?.()}
    </div>
</section>