<script lang="ts">
	import EtalaseCard from "../general/EtalaseCard.svelte";

  const ulang = 9;
</script>


<!-- ==========================================
     SECTION: GRID ETALASE
=========================================== -->
<section id="etalase-section" class="w-full bg-white p-4 md:p-6">
  <!-- Grid responsif: 2 kolom di HP, 3 kolom di tablet, 4 kolom di desktop -->
  <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
    {#each Array(ulang) as _, i}
      <EtalaseCard id={i} path={"/details/etalase"} fun={() => {}}/>
    {/each}
  </div>
</section>