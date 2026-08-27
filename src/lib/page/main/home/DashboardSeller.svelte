<script lang="ts">
	// ///////////////////////////////////////////////////////////////////////
	// Types
	// ///////////////////////////////////////////////////////////////////////

	interface StageOrder {
		label: string;
		jumlah: number;
	}

	interface RekeningDefault {
		namaBank: string;
		nomorRekening: string;
		pemilik: string;
		adaRekening: boolean;
	}

	interface KinerjaPengiriman {
		reguler: number;
		ekspedisi: number;
		ratingRataKurir: number;
		jumlahRating: number;
	}

	interface BarangKritis {
		id: number;
		nama: string;
		stok: number;
		status: 'Stok Menipis' | 'Nonaktif';
	}

	interface KomentarBelumDibalas {
		id: number;
		namaBarang: string;
		komentar: string;
		waktu: string;
	}

	interface DiskonAktif {
		id: number;
		nama: string;
		persen: number;
		jumlahBarang: number;
		sisaPersenWaktu: number; // 0-100, dipakai buat progress bar sisa waktu
		sisaWaktuLabel: string;
	}

	// ///////////////////////////////////////////////////////////////////////
	// Mock Data — ganti dengan hasil fetch dari service masing-masing
	// ///////////////////////////////////////////////////////////////////////

	let namaSeller = $state('Siti Rahmawati');
	let namaToko = $state('Toko Baju Modis');

	let orderPipeline: StageOrder[] = $state([
		{ label: 'Menunggu Approve', jumlah: 8 },
		{ label: 'Diproses', jumlah: 14 },
		{ label: 'Menunggu Kurir', jumlah: 5 },
		{ label: 'Selesai (7 Hari)', jumlah: 62 }
	]);

	let rekening: RekeningDefault = $state({
		namaBank: 'BCA',
		nomorRekening: '5280 •••• 1123',
		pemilik: 'Siti Rahmawati',
		adaRekening: true
	});

	let pengiriman: KinerjaPengiriman = $state({
		reguler: 46,
		ekspedisi: 18,
		ratingRataKurir: 4.6,
		jumlahRating: 32
	});

	let barangKritis: BarangKritis[] = $state([
		{ id: 1, nama: 'Kemeja Linen Oversize', stok: 2, status: 'Stok Menipis' },
		{ id: 2, nama: 'Rok Plisket Katun', stok: 0, status: 'Nonaktif' },
		{ id: 3, nama: 'Blouse Rajut Kancing', stok: 3, status: 'Stok Menipis' }
	]);

	let komentarBelumDibalas: KomentarBelumDibalas[] = $state([
		{ id: 1, namaBarang: 'Kemeja Linen Oversize', komentar: 'Bahannya adem gak kak?', waktu: '12m lalu' },
		{ id: 2, namaBarang: 'Rok Plisket Katun', komentar: 'Ready size L kak?', waktu: '48m lalu' },
		{ id: 3, namaBarang: 'Blouse Rajut Kancing', komentar: 'Real pic dong kak', waktu: '2j lalu' }
	]);

	let diskonAktif: DiskonAktif[] = $state([
		{ id: 1, nama: 'Migrasi Kilat', persen: 20, jumlahBarang: 12, sisaPersenWaktu: 72, sisaWaktuLabel: '2h 6j lagi' },
		{ id: 2, nama: 'Diskon Kategori Atasan', persen: 15, jumlahBarang: 24, sisaPersenWaktu: 24, sisaWaktuLabel: '9j lagi' }
	]);

	let kelengkapanProfil = $state(72); // persen
	let statusJenisSeller: 'Seller Biasa' | 'Pengajuan Diproses' | 'Distributor Resmi' = $state('Pengajuan Diproses');

	// ///////////////////////////////////////////////////////////////////////
	// Derived
	// ///////////////////////////////////////////////////////////////////////

	let totalOrderAktif = $derived(
		orderPipeline.slice(0, 3).reduce((acc, s) => acc + s.jumlah, 0)
	);

	let maxStage = $derived(Math.max(...orderPipeline.map((s) => s.jumlah), 1));

	let totalPengiriman = $derived(pengiriman.reguler + pengiriman.ekspedisi);
	let persenReguler = $derived(Math.round((pengiriman.reguler / totalPengiriman) * 100));
	let persenEkspedisi = $derived(100 - persenReguler);
</script>

<section id="dashboard-seller" class="w-full min-h-screen bg-white p-4 sm:p-6 lg:p-8 text-slate-950 scrollbar-none">
	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- HEADER -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<header class="flex flex-col sm:flex-row sm:items-end sm:justify-between gap-3 sm:gap-0 pb-6 mb-6 border-b border-zinc-800/10">
		<div>
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
				DASHBOARD SELLER
			</span>
			<h1 class="mt-1 text-xl sm:text-2xl lg:text-3xl font-bold uppercase tracking-tight leading-none">
				Halo, {namaSeller}
			</h1>
			<p class="mt-1 text-[11px] sm:text-xs text-slate-500 font-light">
				{namaToko}
			</p>
		</div>

		<div class="flex items-center gap-2">
			<span class="inline-flex items-center gap-1.5 px-2.5 py-1.5 border border-zinc-800/20 rounded-xs text-[9px] sm:text-[10px] font-medium uppercase tracking-wider">
				<span class="w-1.5 h-1.5 rounded-full bg-emerald-500"></span>
				Toko Aktif
			</span>
		</div>
	</header>

	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- GRID -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="grid grid-cols-1 lg:grid-cols-12 gap-3 sm:gap-4">

		<!-- 1. ORDER PIPELINE — hero card -->
		<div class="lg:col-span-8 border border-zinc-800/20 rounded-sm p-4 sm:p-6 flex flex-col justify-between">
			<div class="flex items-start justify-between">
				<div>
					<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
						PIPELINE ORDER
					</span>
					<h2 class="mt-1 text-base sm:text-lg font-bold uppercase tracking-tight">
						{totalOrderAktif} Order Butuh Perhatian
					</h2>
				</div>
				<button
					type="button"
					class="hidden sm:inline-flex border border-zinc-800/30 px-3 py-1.5 text-[9px] font-medium tracking-[0.15em] uppercase rounded-xs hover:bg-slate-950 hover:text-white transition duration-300"
				>
					Kelola Order →
				</button>
			</div>

			<div class="mt-5 sm:mt-6 grid grid-cols-2 sm:grid-cols-4 gap-2.5 sm:gap-3">
				{#each orderPipeline as stage, i}
					<div class="border border-zinc-800/20 rounded-xs p-2.5 sm:p-3 flex flex-col justify-between h-[5.5rem] sm:h-[6.5rem] {i < 3 ? 'bg-white' : 'bg-slate-50'}">
						<span class="text-[8px] sm:text-[9px] font-bold text-slate-950/50 uppercase tracking-wider leading-tight">
							{stage.label}
						</span>
						<div>
							<span class="text-xl sm:text-2xl font-bold font-mono tracking-tight block">
								{stage.jumlah}
							</span>
							<div class="mt-1 w-full h-0.5 bg-slate-100 rounded-full overflow-hidden">
								<div
									class="h-full bg-slate-950 rounded-full"
									style:width="{(stage.jumlah / maxStage) * 100}%"
								></div>
							</div>
						</div>
					</div>
				{/each}
			</div>
		</div>

		<!-- 7. KELENGKAPAN PROFIL & LEVEL TOKO -->
		<div class="lg:col-span-4 border border-zinc-800/20 rounded-sm p-4 sm:p-6 flex flex-col justify-between bg-slate-950 text-white">
			<div class="flex items-start justify-between">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-white/40 uppercase font-mono">
					PROFIL TOKO
				</span>
				<span class="text-[8px] sm:text-[9px] font-mono font-bold uppercase border border-white/20 px-1.5 py-0.5 rounded-xs">
					{statusJenisSeller}
				</span>
			</div>

			<div class="mt-4">
				<div class="flex items-baseline justify-between">
					<span class="text-3xl sm:text-4xl font-bold font-mono tracking-tight">
						{kelengkapanProfil}%
					</span>
					<span class="text-[9px] sm:text-[10px] text-white/50 font-light">Lengkap</span>
				</div>
				<div class="mt-2 w-full h-1 bg-white/10 rounded-full overflow-hidden">
					<div class="h-full bg-white rounded-full" style:width="{kelengkapanProfil}%"></div>
				</div>
			</div>

			<button
				type="button"
				class="mt-4 sm:mt-5 w-full border border-white/20 py-2 text-[9px] font-medium tracking-[0.15em] uppercase rounded-xs hover:bg-white hover:text-slate-950 transition duration-300"
			>
				Lengkapi Profil →
			</button>
		</div>

		<!-- 2. REKENING & PENCAIRAN -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
				PENCAIRAN DANA
			</span>

			{#if rekening.adaRekening}
				<div class="mt-3">
					<p class="text-sm sm:text-base font-bold uppercase tracking-tight">{rekening.namaBank}</p>
					<p class="text-[10px] sm:text-[11px] font-mono text-slate-500 mt-0.5">{rekening.nomorRekening}</p>
					<span class="inline-block mt-2 text-[8px] sm:text-[9px] font-bold uppercase tracking-wider border border-zinc-800/20 px-1.5 py-0.5 rounded-xs">
						Rekening Default
					</span>
				</div>
			{:else}
				<div class="mt-3">
					<p class="text-[11px] sm:text-xs text-slate-600 font-light leading-relaxed">
						Belum ada rekening default. Dana hasil penjualan tidak bisa dicairkan.
					</p>
				</div>
			{/if}

			<button
				type="button"
				class="mt-4 w-full border border-zinc-800/30 py-2 text-[9px] font-medium tracking-[0.15em] uppercase rounded-xs hover:bg-slate-950 hover:text-white transition duration-300"
			>
				{rekening.adaRekening ? 'Kelola Rekening' : 'Tambah Rekening'}
			</button>
		</div>

		<!-- 3. KINERJA PENGIRIMAN -->
		<div class="lg:col-span-5 border border-zinc-800/20 rounded-sm p-4 sm:p-5">
			<div class="flex items-center justify-between">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
					KINERJA PENGIRIMAN
				</span>
				<span class="flex items-center gap-1 text-[10px] sm:text-[11px] font-mono font-bold">
					<svg width="11" height="11" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
						<path class="fill-slate-950" d="M12.6722 2.04308C12.5459 1.78707 12.2851 1.625 11.9996 1.625C11.7142 1.625 11.4534 1.78707 11.3271 2.04308L8.67288 7.4211L2.73788 8.28351C2.45536 8.32456 2.22065 8.52244 2.13243 8.79395C2.04421 9.06546 2.11779 9.36351 2.32222 9.56278L6.61682 13.749L5.603 19.66C5.55475 19.9414 5.67041 20.2257 5.90137 20.3936C6.13233 20.5614 6.43853 20.5835 6.69122 20.4506L11.9996 17.6598L17.3081 20.4506C17.5608 20.5835 17.867 20.5614 18.0979 20.3936C18.3289 20.2257 18.4445 19.9414 18.3963 19.66L17.3825 13.749L21.6771 9.56278C21.8815 9.36351 21.9551 9.06546 21.8669 8.79395C21.7786 8.52244 21.5439 8.32456 21.2614 8.28351L15.3264 7.4211L12.6722 2.04308Z"/>
					</svg>
					{pengiriman.ratingRataKurir.toFixed(1)}
					<span class="text-slate-400 font-light">({pengiriman.jumlahRating})</span>
				</span>
			</div>

			<div class="mt-4 flex w-full h-2 rounded-full overflow-hidden bg-slate-100">
				<div class="h-full bg-slate-950" style:width="{persenReguler}%"></div>
				<div class="h-full bg-slate-300" style:width="{persenEkspedisi}%"></div>
			</div>

			<div class="mt-3 grid grid-cols-2 gap-3">
				<div class="flex items-center gap-2">
					<span class="w-2 h-2 rounded-full bg-slate-950 flex-shrink-0"></span>
					<div>
						<p class="text-[9px] text-slate-500 uppercase tracking-wider font-medium">Reguler</p>
						<p class="text-sm font-bold font-mono">{pengiriman.reguler}</p>
					</div>
				</div>
				<div class="flex items-center gap-2">
					<span class="w-2 h-2 rounded-full bg-slate-300 flex-shrink-0"></span>
					<div>
						<p class="text-[9px] text-slate-500 uppercase tracking-wider font-medium">Ekspedisi</p>
						<p class="text-sm font-bold font-mono">{pengiriman.ekspedisi}</p>
					</div>
				</div>
			</div>
		</div>

		<!-- 5. KOMENTAR BELUM DIBALAS -->
		<div class="lg:col-span-4 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col">
			<div class="flex items-center justify-between">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
					KOMENTAR BELUM DIBALAS
				</span>
				<span class="text-sm font-bold font-mono">{komentarBelumDibalas.length}</span>
			</div>

			<div class="mt-3 space-y-2 flex-1">
				{#each komentarBelumDibalas as k (k.id)}
					<div class="border border-zinc-800/10 rounded-xs p-2 sm:p-2.5 hover:bg-slate-50 transition duration-200 cursor-pointer">
						<div class="flex items-center justify-between">
							<p class="text-[10px] sm:text-[11px] font-bold uppercase tracking-wide truncate">
								{k.namaBarang}
							</p>
							<span class="text-[8px] sm:text-[9px] text-slate-400 font-mono flex-shrink-0 ml-2">
								{k.waktu}
							</span>
						</div>
						<p class="text-[10px] sm:text-[11px] text-slate-600 font-light mt-0.5 truncate">
							"{k.komentar}"
						</p>
					</div>
				{/each}
			</div>

			<button
				type="button"
				class="mt-3 w-full border border-zinc-800/30 py-2 text-[9px] font-medium tracking-[0.15em] uppercase rounded-xs hover:bg-slate-950 hover:text-white transition duration-300"
			>
				Balas Semua →
			</button>
		</div>

		<!-- 4. STOK & STATUS BARANG KRITIS -->
		<div class="lg:col-span-6 border border-zinc-800/20 rounded-sm p-4 sm:p-5">
			<div class="flex items-center justify-between">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
					BARANG PERLU PERHATIAN
				</span>
				<span class="text-sm font-bold font-mono">{barangKritis.length}</span>
			</div>

			<div class="mt-3 space-y-2">
				{#each barangKritis as b (b.id)}
					<div class="flex items-center justify-between p-2 sm:p-2.5 border border-zinc-800/10 rounded-xs">
						<div class="min-w-0">
							<p class="text-[10px] sm:text-[11px] font-bold uppercase tracking-wide truncate">
								{b.nama}
							</p>
							<p class="text-[9px] sm:text-[10px] text-slate-500 font-mono mt-0.5">
								Stok: {b.stok}
							</p>
						</div>
						<span
							class="text-[8px] sm:text-[9px] font-mono font-bold uppercase px-1.5 py-0.5 rounded-xs flex-shrink-0 ml-2 {b.status === 'Nonaktif'
								? 'bg-slate-950 text-white'
								: 'border border-zinc-800/30 text-slate-950'}"
						>
							{b.status}
						</span>
					</div>
				{/each}
			</div>
		</div>

		<!-- 6. DISKON AKTIF -->
		<div class="lg:col-span-6 border border-zinc-800/20 rounded-sm p-4 sm:p-5">
			<div class="flex items-center justify-between">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
					DISKON AKTIF
				</span>
				<span class="text-sm font-bold font-mono">{diskonAktif.length}</span>
			</div>

			<div class="mt-3 space-y-2.5">
				{#each diskonAktif as d (d.id)}
					<div class="p-2.5 sm:p-3 border border-zinc-800/10 rounded-xs">
						<div class="flex items-center justify-between">
							<p class="text-[10px] sm:text-[11px] font-bold uppercase tracking-wide">
								{d.nama} — {d.persen}%
							</p>
							<span class="text-[8px] sm:text-[9px] text-slate-400 font-mono flex-shrink-0">
								{d.sisaWaktuLabel}
							</span>
						</div>
						<p class="text-[9px] sm:text-[10px] text-slate-500 font-light mt-0.5">
							{d.jumlahBarang} barang terdampak
						</p>
						<div class="mt-1.5 w-full h-1 bg-slate-100 rounded-full overflow-hidden">
							<div class="h-full bg-slate-950 rounded-full" style:width="{d.sisaPersenWaktu}%"></div>
						</div>
					</div>
				{/each}
			</div>
		</div>

	</div>
</section>