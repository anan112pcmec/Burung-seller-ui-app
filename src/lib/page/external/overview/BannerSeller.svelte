<script lang="ts">
	// Data statis untuk preview UI
	const seller = {
		nama: "TechHaven Official",
		username: "techhaven_id",
		lokasi: "Jakarta Pusat",
		rating: "4.9",
		pengikut: "24.5K",
		produk_aktif: 128,
		is_official: true
	};
</script>

<section id="banner-seller" class="w-full bg-white border border-zinc-950/10 flex flex-col">
	
	<!-- ─── BAGIAN 1: COVER IMAGE ─── -->
	<div class="relative w-full h-48 md:h-64 bg-zinc-100 border-b border-zinc-950/10 overflow-hidden group">
		<!-- Gambar Cover -->
		<img 
			src="https://picsum.photos/1200/400?random=99" 
			alt="Cover Seller" 
			class="w-full h-full object-cover grayscale opacity-90 group-hover:grayscale-0 group-hover:opacity-100 transition-all duration-700" 
		/>
		
		<!-- Badge Official / Status di Kanan Atas -->
		{#if seller.is_official}
			<div class="absolute top-4 right-4 bg-slate-950 text-white font-mono text-[10px] px-3 py-1.5 uppercase tracking-widest font-bold flex items-center gap-1.5 shadow-sm">
				<svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
					<path stroke-linecap="square" stroke-linejoin="miter" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
				</svg>
				Official Store
			</div>
		{/if}
	</div>

	<!-- ─── BAGIAN 2: INFO BAR STRUKTURAL ─── -->
	<!-- Dibagi menggunakan flex/grid agar seperti tabel data -->
	<div class="flex flex-col md:flex-row items-stretch">
		
		<!-- Identitas Seller (Kiri) -->
		<div class="flex-1 p-4 md:p-6 flex items-center gap-5 border-b md:border-b-0 md:border-r border-zinc-950/10">
			<!-- Avatar Solid Kotak -->
			<div class="w-16 h-16 bg-slate-950 text-white flex items-center justify-center font-mono text-xl font-bold shrink-0 shadow-sm border-2 border-white outline outline-1 outline-zinc-950/10 -mt-10 md:-mt-12 relative z-10">
				{seller.nama.slice(0, 2).toUpperCase()}
			</div>
			
			<div class="flex flex-col pt-1">
				<h1 class="font-sans font-extrabold text-xl md:text-2xl text-slate-950 tracking-tight flex items-center gap-2">
					{seller.nama}
				</h1>
				<div class="flex items-center gap-3 mt-1">
					<span class="font-mono text-[10px] text-zinc-500 uppercase tracking-widest">
						@{seller.username}
					</span>
					<span class="w-1 h-1 bg-zinc-300"></span>
					<span class="font-mono text-[10px] text-zinc-500 uppercase tracking-widest flex items-center gap-1">
						<svg class="w-3 h-3" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
							<path stroke-linecap="square" stroke-linejoin="miter" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z" />
							<path stroke-linecap="square" stroke-linejoin="miter" d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
						</svg>
						{seller.lokasi}
					</span>
				</div>
			</div>
		</div>

		<!-- Panel Metrik / Statistik (Tengah) -->
		<div class="grid grid-cols-3 shrink-0 border-b md:border-b-0 md:border-r border-zinc-950/10 bg-zinc-50/30">
			<!-- Metrik 1 -->
			<div class="p-4 md:px-6 flex flex-col justify-center border-r border-zinc-950/10">
				<span class="font-mono text-[9px] text-zinc-400 uppercase tracking-widest">Rating</span>
				<span class="font-mono font-bold text-slate-950 text-lg flex items-center gap-1">
					<svg class="w-3.5 h-3.5 text-slate-950" viewBox="0 0 24 24" fill="currentColor">
						<path d="M12 17.27L18.18 21l-1.64-7.03L22 9.24l-7.19-.61L12 2 9.19 8.63 2 9.24l5.46 4.73L5.82 21z"/>
					</svg>
					{seller.rating}
				</span>
			</div>
			<!-- Metrik 2 -->
			<div class="p-4 md:px-6 flex flex-col justify-center border-r border-zinc-950/10">
				<span class="font-mono text-[9px] text-zinc-400 uppercase tracking-widest">Pengikut</span>
				<span class="font-mono font-bold text-slate-950 text-lg">{seller.pengikut}</span>
			</div>
			<!-- Metrik 3 -->
			<div class="p-4 md:px-6 flex flex-col justify-center">
				<span class="font-mono text-[9px] text-zinc-400 uppercase tracking-widest">Produk</span>
				<span class="font-mono font-bold text-slate-950 text-lg">{seller.produk_aktif}</span>
			</div>
		</div>

		<!-- Aksi Seller (Kanan) -->
		<div class="flex items-stretch shrink-0 bg-slate-950 hover:bg-slate-800 transition-colors cursor-pointer group">
			<button class="px-6 md:px-8 flex items-center justify-center gap-2 w-full outline-none">
				<svg class="w-4 h-4 text-white group-hover:scale-110 transition-transform" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
					<path stroke-linecap="square" stroke-linejoin="miter" d="M12 4v16m8-8H4" />
				</svg>
				<span class="font-mono font-bold text-[11px] uppercase tracking-widest text-white">
					Ikuti Toko
				</span>
			</button>
		</div>

	</div>
</section>