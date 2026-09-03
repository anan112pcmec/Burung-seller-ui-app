<script lang="ts">
	import { goto } from "$app/navigation";

    
	const bayangkanajaadaimagefotobarangdidiskon = 5;

    const {id, path, fun}: {id:number, path:string, fun?:() => void} = $props()
</script>

<div
    role="button"
    onclick={() => {
    if (fun){
        fun()
    }

    goto(path)
 }}
 class="relative w-[17rem] shrink-0 bg-white border border-zinc-950/10 flex flex-col p-5 group hover:border-slate-950 transition-colors cursor-pointer">
		
		<!-- ─── TOP BAR: LABEL & PERSENTASE ─── -->
		<div class="flex justify-between items-start mb-6">
			<div class="flex flex-col">
				<span class="font-mono text-[10px] text-zinc-500 uppercase tracking-widest font-bold">
					Promo Bundel
				</span>
				<h2 class="font-sans font-extrabold text-slate-950 text-4xl tracking-tighter mt-1 flex items-baseline gap-1">
					40% <span class="text-lg text-zinc-400 font-bold tracking-normal">OFF</span>
				</h2>
			</div>

			<!-- Ikon Boks Hitam Solid -->
			<div class="w-8 h-8 bg-slate-950 text-white flex items-center justify-center shrink-0">
				<!-- Menyesuaikan stroke-linecap jadi square agar senada -->
				<svg class="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="square" stroke-linejoin="miter">
					<path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/>
					<polyline points="3.27 6.96 12 12.01 20.73 6.96"/>
					<line x1="12" y1="22.08" x2="12" y2="12"/>
				</svg>
			</div>
		</div>

		<!-- ─── BODY: IMAGE STACK (TUMPUKAN RAPI) ─── -->
		<!-- Menggunakan perhitungan matematika yang lebih kalem agar kartu tidak terlihat berantakan -->
		<div class="relative h-32 w-full mt-2">
			{#each Array(bayangkanajaadaimagefotobarangdidiskon) as _, idx}
				<div 
					style="z-index: {10 - idx}; transform: translateX({idx * 16}px) rotate({(idx * 5) - 10}deg);" 
					class="absolute top-0 left-4 w-24 h-28 border border-zinc-950/20 bg-zinc-50 transition-all duration-300 origin-bottom"
				>
					<!-- Efek grayscale bawaanmu saya pertahankan, sangat cocok untuk style minimalis -->
					<img 
						src="https://picsum.photos/200/300?random={id * 10 + idx}" 
						alt="Foto Produk Diskon" 
						class="w-full h-full object-cover grayscale opacity-80 group-hover:grayscale-0 group-hover:opacity-100 transition-all duration-500" 
					/>
				</div>
			{/each}
		</div>

		<!-- ─── FOOTER: INFO TAMBAHAN ─── -->
		<div class="mt-6 border-t border-zinc-950/10 pt-3 flex items-center justify-between">
			<span class="font-mono text-[9px] text-zinc-400 uppercase tracking-widest">
				Berakhir 3 Hari
			</span>
			<span class="font-mono text-[10px] font-bold text-slate-950 flex items-center gap-1 group-hover:translate-x-1 transition-transform">
				KLAIM 
				<svg class="w-3 h-3" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
					<path stroke-linecap="square" stroke-linejoin="miter" d="M5 12h14M12 5l7 7-7 7"/>
				</svg>
			</span>
		</div>

	</div>