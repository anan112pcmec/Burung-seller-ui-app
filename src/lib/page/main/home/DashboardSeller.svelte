<script lang="ts">
	// ///////////////////////////////////////////////////////////////////////
	// Types
	// ///////////////////////////////////////////////////////////////////////

	interface StageOrder {
		label: string;
		jumlah: number;
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

	interface Komplain {
		foto_profil: string,
		nama: string, 
		pesan: string,
		created_at: string
	}

	interface Pesanan {
		foto_barang: string
		nama_barang: string,
		kategori_barang: string,
		harga_barang: number,
		kuantitas: number,
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

	let listKomplain: Komplain[] = [
		{
			foto_profil: "free_image",
			nama: "Faiz",
			pesan: "bagaimana ini pengiriman nya bermasalah",
			created_at: "20-12-2026"
		},
		{
			foto_profil: "free_image",
			nama: "iconk",
			pesan: "ini gimana ya kok barang saya tidak lengkap",
			created_at: "20-11-2026"
		},
		{
			foto_profil: "free_image",
			nama: "ical",
			pesan: "gabaik nih seller pelayanan nya",
			created_at: "20-01-2026"
		},
		{
			foto_profil: "free_image",
			nama: "bambang",
			pesan: "gak sesuai produk yang dikirim dengan produk yang diberikan",
			created_at: "20-02-2026"
		},
	]

	let listPesanan: Pesanan[] = [
    {
        foto_barang: "https://images.unsplash.com/photo-1542291026-7eec264c27ff?w=100&auto=format&fit=crop&q=60",
        nama_barang: "Sepatu Nike Air Jordan",
        kategori_barang: "Nike Jordan A1",
        harga_barang: 2999999,
        kuantitas: 2
    },
    {
        foto_barang: "https://images.unsplash.com/photo-1523275335684-37898b6baf30?w=100&auto=format&fit=crop&q=60",
        nama_barang: "Smartwatch Series 8",
        kategori_barang: "Apple Gadget",
        harga_barang: 5499000,
        kuantitas: 1
    },
    {
        foto_barang: "https://images.unsplash.com/photo-1505740420928-5e560c06d30e?w=100&auto=format&fit=crop&q=60",
        nama_barang: "Headphone Wireless ANC",
        kategori_barang: "Audio & Music",
        harga_barang: 1250000,
        kuantitas: 3
    },
    {
        foto_barang: "https://images.unsplash.com/photo-1583394838336-acd977736f90?w=100&auto=format&fit=crop&q=60",
        nama_barang: "Mechanical Keyboard RGB",
        kategori_barang: "Computer Gear",
        harga_barang: 850000,
        kuantitas: 1
    },
    {
        foto_barang: "https://images.unsplash.com/photo-1491553895911-0055eca6402d?w=100&auto=format&fit=crop&q=60",
        nama_barang: "Sneakers Adidas Ultraboost",
        kategori_barang: "Adidas Running",
        harga_barang: 2100000,
        kuantitas: 2
    }
];

function totalkanPesanan(pesanan: Pesanan[]): number {
	let total: number = 0;
	for (const p of pesanan) {
		total = total + p.kuantitas;
	}

	return total;
}
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
	<div class="lg:col-span-3 border border-zinc-200 bg-white rounded-md p-4 sm:p-5 flex flex-col justify-around shadow-xs">
    <!-- Header -->
    <div class="flex items-center justify-between mb-3">
        <span class="text-[10px] font-bold tracking-[0.15em] text-zinc-500 uppercase font-mono flex items-center gap-2">
            Informasi Komplain 
            <span class="bg-zinc-900 text-white text-[9px] px-2 py-0.5 rounded-full font-sans">
                {listKomplain.length}
            </span>
        </span>
    </div>

    <!-- List Komplain Container -->
    <div class="w-full space-y-3 overflow-y-auto scrollbar-none rounded h-48 sm:h-56">
        {#each listKomplain as komplain}
            <div class="flex items-start space-x-3 p-2 rounded hover:bg-white transition-colors border border-transparent hover:border-zinc-200/60">
                <!-- Avatar -->
                <div class="bg-zinc-200 h-7 w-7 rounded-full overflow-hidden shrink-0"> 
                    <img src="{komplain.foto_profil}" alt="{komplain.nama}" class="h-full w-full object-cover">
                </div>
                <!-- Text Content (min-w-0 wajib agar truncate berfungsi dalam flex) -->
                <div class="flex flex-col min-w-0 flex-1">
                    <span class="text-zinc-800 text-[11px] font-medium leading-tight">{komplain.nama}</span>
                    <span class="text-zinc-400 text-[10px] truncate mt-0.5">{komplain.pesan}</span>
                </div>
            </div>
        {/each}
    </div>

    <!-- Action Button -->
    <button
        type="button"
        class="mt-4 w-full border border-zinc-300 bg-white py-2 text-[10px] font-semibold tracking-[0.15em] uppercase rounded text-zinc-700 hover:bg-zinc-900 hover:border-zinc-900 hover:text-white transition-all duration-200"
    >
        Kelola Komplain
    </button>
</div>

		<!-- 3. Pesanan Masuk -->
		<div class="lg:col-span-5 border border-zinc-800/20 rounded-sm p-4 sm:p-5">
			<div class="flex items-center justify-between">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
					Pesanan Masuk Hari Ini
				</span>
				<span class="flex items-center gap-1 p-2 text-[10px] sm:text-[11px] font-mono font-bold">
					<svg xmlns="http://www.w3.org/2000/svg" width="11" height="11" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-package-icon lucide-package"><path d="M11 21.73a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73z"/><path d="M12 22V12"/><polyline points="3.29 7 12 12 20.71 7"/><path d="m7.5 4.27 9 5.15"/></svg>
					{totalkanPesanan(listPesanan)} Barang
					
				</span>
			</div>
			<div class="w-full space-y-2.5 overflow-y-auto scrollbar-none rounded h-50 sm:h-60 pt-2 pr-1">
				{#each listPesanan as pesanan}
					<div class="flex items-center justify-between p-2.5 rounded border border-zinc-200/80 bg-white hover:border-zinc-300 hover:shadow-xs transition-all">
						<!-- Bagian Kiri: Foto & Detail Barang -->
						<div class="flex items-center space-x-3 min-w-0 flex-1 mr-3">
							<!-- Foto Barang -->
							<div class="h-10 w-10 rounded bg-zinc-100 overflow-hidden shrink-0 border border-zinc-200/50">
								<img src="{pesanan.foto_barang}" alt="{pesanan.nama_barang}" class="h-full w-full object-cover">
							</div>
							
							<!-- Teks (Nama & Kategori) -->
							<div class="flex flex-col min-w-0 flex-1">
								<span class="text-zinc-800 text-[11px] font-semibold truncate leading-tight">
									{pesanan.nama_barang}
								</span>
								<span class="text-zinc-400 text-[9px] truncate mt-0.5 font-mono">
									Kategori -- {pesanan.kategori_barang}
								</span>
							</div>
						</div>

						<!-- Bagian Kanan: Harga & Kuantitas -->
						<div class="flex flex-col items-end shrink-0 text-right">
							<span class="text-zinc-900 text-[11px] font-bold font-mono">
								Rp {(pesanan.kuantitas * pesanan.harga_barang).toLocaleString('id-ID')}
							</span>
							<span class="text-zinc-500 text-[9px] bg-zinc-100 px-1.5 py-0.5 rounded mt-0.5 font-mono">
								{pesanan.kuantitas}x
							</span>
						</div>
					</div>
				{/each}
			</div>
			
			 <button
				type="button"
				class="mt-4 w-full border border-zinc-300 bg-white py-2 text-[10px] font-semibold tracking-[0.15em] uppercase rounded text-zinc-700 hover:bg-zinc-900 hover:border-zinc-900 hover:text-white transition-all duration-200"
			>
				Kelola Pesanan
			</button>
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