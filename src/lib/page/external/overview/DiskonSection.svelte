<script lang="ts">
	import DiskonCard from "../general/DiskonCard.svelte";

	const bayangkanajasisellerbikindiskonsebanyak = 6;
</script>

<!-- ==========================================
     SNIPPET: ELEMEN DISKON KARTU
=========================================== -->

<!-- ==========================================
     SECTION: CONTAINER HORIZONTAL SCROLL
=========================================== -->
<section id="diskon-section" class="w-full bg-white border border-zinc-950/10 flex flex-col">
	
	<!-- Header Section (Senada dengan 'Barang Unggulan') -->
	<div class="flex items-center justify-between border-b border-zinc-950/10 p-4 md:px-6 bg-zinc-50/50">
		<h2 class="font-sans font-bold text-slate-950 text-sm md:text-base tracking-tight uppercase">
			Diskon & Penawaran Toko
		</h2>
		
		<div class="flex items-center gap-2 font-mono text-[10px] text-zinc-500 uppercase tracking-widest font-bold">
			<span>{bayangkanajasisellerbikindiskonsebanyak} PROMO AKTIF</span>
		</div>
	</div>

	<!-- Kontainer Gulir -->
	<div class="flex gap-4 overflow-x-auto scrollbar-none items-stretch px-4 py-6 md:px-6 w-full">
		{#each Array(bayangkanajasisellerbikindiskonsebanyak) as _, i}
			<DiskonCard id={i} path={"details/diskon"} fun={() => {}}/>
		{/each}
	</div>
	
</section>