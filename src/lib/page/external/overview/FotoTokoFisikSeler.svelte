<script lang="ts">
	// Data statis untuk preview UI
	const storeInfo = {
		address: "Jl. Jend. Sudirman Kav. 45, Jakarta Pusat, 10210",
		coordinates: "6.2215° S, 106.8044° E",
		openHours: "08:00 - 17:00 WIB"
	}

	// Kumpulan foto untuk carousel showroom (bisa diisi berapa saja)
	const showroomPhotos = [
		{ id: 1, url: "https://picsum.photos/400/300?random=102", label: "Area Display Utama" },
		{ id: 2, url: "https://picsum.photos/400/300?random=105", label: "Kasir & Layanan Pelanggan" },
		{ id: 3, url: "https://picsum.photos/400/300?random=108", label: "Sudut Gudang & Stok" },
		{ id: 4, url: "https://picsum.photos/400/300?random=111", label: "Area Packing Barang" }
	];

	// State untuk index aktif carousel
	let currentSlide = $state(0);

	function nextSlide() {
		currentSlide = (currentSlide + 1) % showroomPhotos.length;
	}

	function prevSlide() {
		currentSlide = (currentSlide - 1 + showroomPhotos.length) % showroomPhotos.length;
	}
</script>

<section id="foto-toko-fisik" class="w-full bg-white border border-zinc-950/10 flex flex-col mt-6">
	
	<!-- ─── HEADER SECTION ─── -->
	<div class="flex flex-col md:flex-row md:items-center justify-between border-b border-zinc-950/10 p-4 md:px-6 bg-zinc-50/50">
		<h2 class="font-sans font-bold text-slate-950 text-sm md:text-base tracking-tight uppercase">
			Lokasi & Fasilitas Fisik
		</h2>
		
		<!-- Koordinat bergaya navigasi satelit -->
		<div class="font-mono text-[10px] text-zinc-500 uppercase tracking-widest mt-1 md:mt-0 font-bold flex items-center gap-3">
			<span>{storeInfo.coordinates}</span>
			<span class="w-1 h-1 bg-zinc-300"></span>
			<span class="text-slate-950 flex items-center gap-1">
				<svg class="w-3 h-3 text-green-600" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="3">
					<path stroke-linecap="square" stroke-linejoin="miter" d="M5 13l4 4L19 7" />
				</svg>
				VERIFIED
			</span>
		</div>
	</div>

	<!-- ─── STRUKTURAL GRID LAYOUT ─── -->
	<div class="grid grid-cols-1 md:grid-cols-3 md:min-h-[25rem]">
		
		<!-- 1. Main Image (Kiri - Lebar 2 Kolom) -->
		<div class="md:col-span-2 relative border-b md:border-b-0 md:border-r border-zinc-950/10 group overflow-hidden bg-zinc-100">
			<img 
				src="https://picsum.photos/800/600?random=101" 
				alt="Toko Tampak Depan" 
				class="w-full h-full object-cover grayscale opacity-90 group-hover:grayscale-0 group-hover:opacity-100 transition-all duration-700"
			>
			<!-- Label Posisi Absolut (Kaku & Persegi) -->
			<div class="absolute bottom-4 left-4 bg-slate-950 text-white font-mono text-[10px] px-3 py-1.5 uppercase tracking-widest font-bold">
				Fasad Bangunan
			</div>
		</div>

		<!-- 2. Sidebar Kanan (Dibelah Atas-Bawah) -->
		<div class="flex flex-col">
			
			<!-- Image Carousel (Area Dalam/Interior) -->
			<div class="flex-1 relative border-b border-zinc-950/10 group overflow-hidden bg-zinc-100 min-h-[14rem] flex flex-col">
				
				<!-- Slide Container -->
				<div class="relative w-full h-full flex-1 overflow-hidden">
					{#each showroomPhotos as photo, idx}
						<div 
							class="absolute inset-0 w-full h-full transition-opacity duration-500 {idx === currentSlide ? 'opacity-100 z-10' : 'opacity-0 z-0'}"
						>
							<img 
								src={photo.url} 
								alt={photo.label} 
								class="w-full h-full object-cover grayscale opacity-90 group-hover:grayscale-0 group-hover:opacity-100 transition-all duration-700"
							>
							<!-- Label Foto Aktif -->
							<div class="absolute bottom-3 left-3 bg-white text-slate-950 border border-zinc-950/10 font-mono text-[9px] px-2 py-1 uppercase tracking-widest font-bold z-20">
								{photo.label}
							</div>
						</div>
					{/each}
				</div>

				<!-- Tombol Navigasi Carousel (Kiri & Kanan) -->
				<div class="absolute top-3 right-3 z-30 flex items-center gap-1">
					<button 
						onclick={prevSlide}
						class="w-7 h-7 bg-slate-950/80 hover:bg-slate-950 text-white border border-white/20 flex items-center justify-center transition-colors"
						aria-label="Previous Slide"
					>
						<svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2.5">
							<path stroke-linecap="square" stroke-linejoin="miter" d="M15 19l-7-7 7-7" />
						</svg>
					</button>
					<button 
						onclick={nextSlide}
						class="w-7 h-7 bg-slate-950/80 hover:bg-slate-950 text-white border border-white/20 flex items-center justify-center transition-colors"
						aria-label="Next Slide"
					>
						<svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2.5">
							<path stroke-linecap="square" stroke-linejoin="miter" d="M9 5l7 7-7 7" />
						</svg>
					</button>
				</div>

				<!-- Indikator Angka Carousel (Monospace) -->
				<div class="absolute top-3 left-3 z-20 bg-slate-950 text-white font-mono text-[9px] px-2 py-1 tracking-widest font-bold">
					0{currentSlide + 1} / 0{showroomPhotos.length}
				</div>

			</div>
			
			<!-- Panel Data Kunjungan (Bawah) -->
			<div class="p-5 md:p-6 bg-zinc-50 flex flex-col justify-center flex-1">
				<h3 class="font-sans font-bold text-slate-950 text-sm mb-4 uppercase tracking-tight">
					Data Kunjungan
				</h3>
				
				<div class="flex flex-col gap-3">
					<div>
						<p class="font-mono text-[9px] text-zinc-400 uppercase tracking-widest mb-1">
							Alamat Lengkap
						</p>
						<p class="font-sans text-xs text-slate-800 leading-relaxed font-medium">
							{storeInfo.address}
						</p>
					</div>
					
					<!-- Garis Pemisah (Divider) -->
					<div class="w-full h-px bg-zinc-950/10 my-1"></div>
					
					<div>
						<p class="font-mono text-[9px] text-zinc-400 uppercase tracking-widest mb-1">
							Jam Operasional
						</p>
						<p class="font-mono text-xs text-slate-950 font-bold tracking-tight">
							{storeInfo.openHours}
						</p>
					</div>
				</div>

				<!-- Tombol Aksi Kaku / Monolithic -->
				<button class="mt-5 w-full bg-white border border-slate-950 text-slate-950 font-mono font-bold text-[10px] uppercase tracking-widest py-2.5 hover:bg-slate-950 hover:text-white transition-colors flex items-center justify-center gap-2 group">
					<svg class="w-3 h-3 group-hover:scale-110 transition-transform" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
						<path stroke-linecap="square" stroke-linejoin="miter" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z" />
						<path stroke-linecap="square" stroke-linejoin="miter" d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
					</svg>
					Buka di Maps
				</button>
			</div>

		</div>

	</div>
</section>