<script lang="ts">
	// ///////////////////////////////////////////////////////////////////////
	// Types
	// ///////////////////////////////////////////////////////////////////////

	interface KategoriRingkas {
		nama: string;
		terjual: number; // unit — agregat dari Transaksi.KuantitasBarang, join id_kategori_barang
		omset: number; // Rp — agregat dari Transaksi.Total
	}

	interface Point {
		x: number;
		y: number;
	}

	interface KategoriBarangCard {
		id: number;
		namaKategori: string; // KategoriBarang.Nama
		namaBarangInduk: string; // BarangInduk.NamaBarang
		deskripsi: string; // KategoriBarang.Deskripsi
		sku: string; // KategoriBarang.Sku
		isOriginal: boolean; // KategoriBarang.IsOriginal
		harga: number; // KategoriBarang.Harga
		fotoUrl: string; // TODO: ganti dgn URL asli dari media_services, ini placeholder
		riwayatLikes: number[]; // TODO: butuh service engagement terpisah, belum ada di struct manapun
		riwayatPenjualan: number[]; // agregat harian dari Transaksi.KuantitasBarang
		riwayatWishlist: number[]; // TODO: belum ada service wishlist yang kebaca sama sekali
		riwayatDilihat: number[]; // TODO: sama seperti likes, kemungkinan nempel di service lain
	}

	// ///////////////////////////////////////////////////////////////////////
	// Mock data — TOP METRICS
	// ganti dengan hasil query agregasi barang_induk + kategori_barang + transaksi
	// ///////////////////////////////////////////////////////////////////////

	const kategoriMaster: KategoriRingkas[] = [
		{ nama: 'Elektronik & Gadget', terjual: 482, omset: 186_400_000 },
		{ nama: 'Fashion Wanita', terjual: 356, omset: 71_200_000 },
		{ nama: 'Fashion Pria', terjual: 298, omset: 59_600_000 },
		{ nama: 'Kecantikan & Perawatan', terjual: 241, omset: 36_150_000 },
		{ nama: 'Rumah Tangga', terjual: 164, omset: 24_600_000 },
		{ nama: 'Olahraga & Outdoor', terjual: 97, omset: 19_400_000 }
	];

	const totalVarianAktif = 148; // count KategoriBarang, deleted_at IS NULL
	const ambangStokKritis = 10;

	const stokKritisList = [
		{ nama: 'Kemeja Linen Oversize - L', stok: 3 },
		{ nama: 'Case iPhone 15 Pro - Bening', stok: 5 },
		{ nama: 'Serum Niacinamide 20ml', stok: 7 },
		{ nama: 'Tumbler Stainless 500ml', stok: 9 }
	];

	// ///////////////////////////////////////////////////////////////////////
	// Derived — angka ringkasan 4 metrik utama
	// ///////////////////////////////////////////////////////////////////////

	let kategoriTerurut = $derived([...kategoriMaster].sort((a, b) => b.terjual - a.terjual));
	let totalOmset = $derived(kategoriMaster.reduce((a, k) => a + k.omset, 0));
	let totalTerjual = $derived(kategoriMaster.reduce((a, k) => a + k.terjual, 0));
	let stokKritisCount = $derived(stokKritisList.length);

	function formatRupiah(n: number): string {
		if (n >= 1_000_000_000) return `Rp${(n / 1_000_000_000).toFixed(1)}M`;
		if (n >= 1_000_000) return `Rp${(n / 1_000_000).toFixed(1)}jt`;
		if (n >= 1_000) return `Rp${(n / 1_000).toFixed(0)}rb`;
		return `Rp${n}`;
	}

	// ///////////////////////////////////////////////////////////////////////
	// SVG helper — sparkline mini buat tiap metrik di card bawah
	// ///////////////////////////////////////////////////////////////////////

	const SPARK_W = 100;
	const SPARK_H = 28;
	const SPARK_PAD = 2;

	function toPoints(values: number[], w: number, h: number, pad: number): Point[] {
		const max = Math.max(...values, 1);
		const usableW = w - pad * 2;
		const usableH = h - pad * 2;
		const step = values.length > 1 ? usableW / (values.length - 1) : 0;
		return values.map((v, i) => ({
			x: pad + step * i,
			y: pad + usableH - (v / max) * usableH
		}));
	}

	function linePath(points: Point[]): string {
		return points.map((p, i) => `${i === 0 ? 'M' : 'L'} ${p.x},${p.y}`).join(' ');
	}

	function areaPath(points: Point[], h: number, pad: number): string {
		if (points.length === 0) return '';
		const first = points[0];
		const last = points[points.length - 1];
		return `${linePath(points)} L ${last.x},${h - pad} L ${first.x},${h - pad} Z`;
	}

	// ///////////////////////////////////////////////////////////////////////
	// Mock data — LIST KATEGORI BARANG (card bawah)
	// foto pakai placeholder seeded, ganti dengan URL asli dari media_services
	// ///////////////////////////////////////////////////////////////////////

	const kategoriBarangList: KategoriBarangCard[] = [
		{
			id: 1,
			namaKategori: 'Kemeja Linen Oversize - Biru Dongker',
			namaBarangInduk: 'Kemeja Linen Oversize',
			deskripsi: 'Bahan linen premium adem, potongan oversize kekinian, cocok buat harian maupun semi formal.',
			sku: 'KLO-BD-L',
			isOriginal: true,
			harga: 189_000,
			fotoUrl: 'https://picsum.photos/seed/kemeja-linen/300/300',
			riwayatLikes: [40, 52, 48, 61, 70, 66, 82],
			riwayatPenjualan: [8, 12, 10, 15, 18, 14, 22],
			riwayatWishlist: [20, 24, 22, 30, 33, 29, 38],
			riwayatDilihat: [420, 480, 510, 560, 590, 610, 680]
		},
		{
			id: 2,
			namaKategori: 'Case iPhone 15 Pro - Bening',
			namaBarangInduk: 'Case iPhone 15 Pro',
			deskripsi: 'Case bening anti kuning, presisi cutout kamera, dilengkapi bantalan sudut anti benturan.',
			sku: 'CIP-15P-BN',
			isOriginal: false,
			harga: 45_000,
			fotoUrl: 'https://picsum.photos/seed/case-iphone/300/300',
			riwayatLikes: [90, 95, 88, 102, 110, 98, 120],
			riwayatPenjualan: [25, 30, 28, 34, 40, 36, 48],
			riwayatWishlist: [15, 18, 16, 20, 19, 22, 25],
			riwayatDilihat: [900, 940, 980, 1020, 1100, 1080, 1200]
		},
		{
			id: 3,
			namaKategori: 'Serum Niacinamide 20ml',
			namaBarangInduk: 'Serum Wajah Brightening',
			deskripsi: 'Serum niacinamide 5% + zinc, membantu mencerahkan & mengontrol minyak berlebih di wajah.',
			sku: 'SRM-NC-20',
			isOriginal: true,
			harga: 79_000,
			fotoUrl: 'https://picsum.photos/seed/serum-wajah/300/300',
			riwayatLikes: [60, 58, 65, 70, 68, 75, 80],
			riwayatPenjualan: [14, 12, 16, 18, 17, 20, 24],
			riwayatWishlist: [35, 38, 36, 40, 42, 39, 45],
			riwayatDilihat: [610, 590, 630, 660, 700, 690, 740]
		},
		{
			id: 4,
			namaKategori: 'Tumbler Stainless 500ml - Sage Green',
			namaBarangInduk: 'Tumbler Stainless Vacuum',
			deskripsi: 'Vacuum flask 2 lapis, tahan panas 6 jam & dingin 12 jam, muat di cup holder mobil.',
			sku: 'TMB-SS-500',
			isOriginal: false,
			harga: 65_000,
			fotoUrl: 'https://picsum.photos/seed/tumbler/300/300',
			riwayatLikes: [30, 34, 32, 36, 40, 38, 44],
			riwayatPenjualan: [6, 8, 7, 9, 11, 10, 13],
			riwayatWishlist: [12, 14, 13, 16, 18, 17, 20],
			riwayatDilihat: [280, 300, 310, 330, 350, 340, 380]
		},
		{
			id: 5,
			namaKategori: 'Matras Yoga 6mm - Ungu',
			namaBarangInduk: 'Matras Yoga Anti Slip',
			deskripsi: 'Permukaan anti slip dua sisi, ketebalan 6mm nyaman untuk lantai keras, dilengkapi tali jinjing.',
			sku: 'MYG-6-UN',
			isOriginal: true,
			harga: 112_000,
			fotoUrl: 'https://picsum.photos/seed/matras-yoga/300/300',
			riwayatLikes: [18, 20, 19, 22, 24, 21, 26],
			riwayatPenjualan: [4, 5, 4, 6, 7, 6, 8],
			riwayatWishlist: [9, 10, 9, 11, 12, 11, 14],
			riwayatDilihat: [150, 160, 155, 170, 180, 175, 190]
		}
	];
</script>

<section id="metric-barang" class="w-full bg-white p-4 sm:p-6 lg:p-8 text-slate-950">
	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- HEADER -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="pb-6 mb-6 border-b border-zinc-800/10">
		<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
			ANALITIK BARANG INDUK
		</span>
		<h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
			NIKE A1 
		</h1>
	</div>

	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- 4 METRIK UTAMA -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="grid grid-cols-1 lg:grid-cols-12 gap-4">

		<!-- 2. TOTAL OMSET -->
		<div class="lg:col-span-6 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				TOTAL OMSET DARI BARANG INI
			</span>
			<span class="mt-2 text-2xl sm:text-3xl font-bold font-mono tracking-tight text-teal-700">
				{formatRupiah(totalOmset)}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">
				dari {totalTerjual.toLocaleString('id-ID')} unit terjual, gabungan semua kategori
			</p>
		</div>

		<!-- 4. STOK KRITIS — actionable -->
		<div class="lg:col-span-6 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<div class="flex items-center justify-between gap-2">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					STOK KRITIS
				</span>
				{#if stokKritisCount > 0}
					<span class="w-1.5 h-1.5 rounded-full bg-teal-600 flex-shrink-0"></span>
				{/if}
			</div>
			<span class="mt-2 text-2xl sm:text-3xl font-bold font-mono tracking-tight">
				{stokKritisCount}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">
				varian dengan stok di bawah {ambangStokKritis} unit
			</p>
		</div>

		<!-- 1. KATEGORI PALING DIMINATI — ranking -->
		<div class="lg:col-span-7 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				KATEGORI PALING DIMINATI
			</span>
			<span class="text-[9px] text-slate-400 font-light mt-0.5">diurutkan berdasarkan unit terjual</span>

			<div class="mt-4 space-y-3 flex-1 flex flex-col justify-center">
				{#each kategoriTerurut as k, i}
					{@const maxTerjual = Math.max(kategoriTerurut[0].terjual, 1)}
					<div>
						<div class="flex items-center justify-between mb-1">
							<span class="text-[10px] sm:text-[11px] font-medium truncate flex items-center gap-1.5">
								<span class="text-teal-700 font-mono">0{i + 1}</span>
								{k.nama}
							</span>
							<span class="text-[10px] sm:text-[11px] font-bold font-mono flex-shrink-0 ml-2">
								{k.terjual} unit
							</span>
						</div>
						<div class="w-full h-1 bg-slate-100 rounded-full overflow-hidden">
							<div class="h-full bg-teal-600 rounded-full" style:width="{(k.terjual / maxTerjual) * 100}%"></div>
						</div>
					</div>
				{/each}
			</div>
		</div>

		<!-- 3. TOTAL VARIAN AKTIF -->
		<div class="lg:col-span-5 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				TOTAL VARIAN AKTIF
			</span>
			<div class="mt-4 flex-1 flex flex-col items-center justify-center gap-1">
				<span class="text-3xl sm:text-4xl font-bold font-mono tracking-tight">{totalVarianAktif}</span>
				<span class="text-[10px] text-slate-400">kategori/varian barang aktif</span>
			</div>
			<div class="mt-3 pt-3 border-t border-dashed border-zinc-200">
				<p class="text-[9px] text-zinc-500 leading-relaxed">
					Rata-rata <strong class="text-zinc-700">{Math.round(totalVarianAktif / kategoriMaster.length)}</strong>
					varian per kelompok kategori dari {kategoriMaster.length} kategori yang tercatat.
				</p>
			</div>
		</div>

	</div>

	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- LIST KATEGORI BARANG — card horizontal, scroll vertikal -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="mt-8">
		<div class="pb-3 mb-4 border-b border-zinc-800/10">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
				DETAIL PER KATEGORI BARANG
			</span>
			<h2 class="mt-1 text-base sm:text-lg font-bold uppercase tracking-tight leading-none">
				Performa Tiap Varian
			</h2>
		</div>

		<div class="flex flex-col gap-3 max-h-[42rem] overflow-y-auto scrollbar-none pr-1">
			{#each kategoriBarangList as item (item.id)}
				{@const likesPts = toPoints(item.riwayatLikes, SPARK_W, SPARK_H, SPARK_PAD)}
				{@const penjualanPts = toPoints(item.riwayatPenjualan, SPARK_W, SPARK_H, SPARK_PAD)}
				{@const wishlistPts = toPoints(item.riwayatWishlist, SPARK_W, SPARK_H, SPARK_PAD)}
				{@const dilihatPts = toPoints(item.riwayatDilihat, SPARK_W, SPARK_H, SPARK_PAD)}

				<div class="border border-zinc-200 hover:border-zinc-400 rounded-lg bg-white shadow-sm overflow-hidden transition-colors duration-150">
					<div class="grid grid-cols-1 sm:grid-cols-[9rem_1fr] lg:grid-cols-[9rem_1fr_auto]">

						<!-- FOTO -->
						<div class="h-36 sm:h-full w-full bg-zinc-100 relative overflow-hidden">
							<img src={item.fotoUrl} alt={item.namaKategori} class="w-full h-full object-cover" />
							{#if item.isOriginal}
								<span class="absolute top-1.5 left-1.5 px-1.5 py-0.5 bg-teal-600/90 text-white text-[8px] font-bold uppercase tracking-wider rounded">
									Original
								</span>
							{/if}
						</div>

						<!-- INFO -->
						<div class="p-3 sm:p-4 flex flex-col justify-center min-w-0">
							<div class="flex items-center gap-1.5 flex-wrap mb-1">
								<span class="text-[9px] text-zinc-400 font-mono uppercase tracking-wider">{item.namaBarangInduk}</span>
								<span class="text-zinc-300">·</span>
								<span class="text-[9px] text-zinc-400 font-mono">SKU {item.sku}</span>
							</div>
							<h3 class="text-xs sm:text-sm font-semibold text-zinc-800 leading-tight truncate">
								{item.namaKategori}
							</h3>
							<p class="text-[10px] text-zinc-400 line-clamp-2 mt-1 leading-tight max-w-md">
								{item.deskripsi}
							</p>
							<span class="text-[11px] font-bold font-mono text-zinc-800 mt-1.5">
								{formatRupiah(item.harga)}
							</span>
						</div>

						<!-- METRIK + KURVA -->
						<div class="grid grid-cols-2 gap-x-4 gap-y-3 p-3 sm:p-4 border-t sm:border-t-0 sm:border-l border-zinc-100 bg-zinc-50/40">
							<!-- Likes -->
							<div class="flex items-center gap-2">
								<svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-rose-400 flex-shrink-0" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
									<path d="M19 14c1.49-1.46 3-3.21 3-5.5A5.5 5.5 0 0 0 16.5 3c-1.76 0-3 .5-4.5 2-1.5-1.5-2.74-2-4.5-2A5.5 5.5 0 0 0 2 8.5c0 2.3 1.5 4.05 3 5.5l7 7Z"/>
								</svg>
								<div>
									<p class="text-[10px] font-bold font-mono text-zinc-800 leading-none">{item.riwayatLikes[item.riwayatLikes.length - 1]}</p>
									<p class="text-[8px] text-zinc-400 uppercase tracking-wider">Likes</p>
								</div>
								<svg viewBox="0 0 {SPARK_W} {SPARK_H}" class="w-12 h-5 ml-auto flex-shrink-0" preserveAspectRatio="none">
									<path d={areaPath(likesPts, SPARK_H, SPARK_PAD)} class="fill-rose-400/10" />
									<path d={linePath(likesPts)} fill="none" class="stroke-rose-400" stroke-width="1.5" />
								</svg>
							</div>

							<!-- Penjualan -->
							<div class="flex items-center gap-2">
								<svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-teal-600 flex-shrink-0" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
									<path d="M6 2 3 6v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2V6l-3-4Z"/><path d="M3 6h18"/><path d="M16 10a4 4 0 0 1-8 0"/>
								</svg>
								<div>
									<p class="text-[10px] font-bold font-mono text-zinc-800 leading-none">{item.riwayatPenjualan[item.riwayatPenjualan.length - 1]}</p>
									<p class="text-[8px] text-zinc-400 uppercase tracking-wider">Penjualan</p>
								</div>
								<svg viewBox="0 0 {SPARK_W} {SPARK_H}" class="w-12 h-5 ml-auto flex-shrink-0" preserveAspectRatio="none">
									<path d={areaPath(penjualanPts, SPARK_H, SPARK_PAD)} class="fill-teal-600/10" />
									<path d={linePath(penjualanPts)} fill="none" class="stroke-teal-600" stroke-width="1.5" />
								</svg>
							</div>

							<!-- Wishlist -->
							<div class="flex items-center gap-2">
								<svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-amber-500 flex-shrink-0" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
									<path d="m19 21-7-4-7 4V5a2 2 0 0 1 2-2h10a2 2 0 0 1 2 2v16z"/>
								</svg>
								<div>
									<p class="text-[10px] font-bold font-mono text-zinc-800 leading-none">{item.riwayatWishlist[item.riwayatWishlist.length - 1]}</p>
									<p class="text-[8px] text-zinc-400 uppercase tracking-wider">Wishlist</p>
								</div>
								<svg viewBox="0 0 {SPARK_W} {SPARK_H}" class="w-12 h-5 ml-auto flex-shrink-0" preserveAspectRatio="none">
									<path d={areaPath(wishlistPts, SPARK_H, SPARK_PAD)} class="fill-amber-500/10" />
									<path d={linePath(wishlistPts)} fill="none" class="stroke-amber-500" stroke-width="1.5" />
								</svg>
							</div>

							<!-- Dilihat -->
							<div class="flex items-center gap-2">
								<svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-slate-500 flex-shrink-0" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
									<path d="M2.062 12.348a1 1 0 0 1 0-.696 10.75 10.75 0 0 1 19.876 0 1 1 0 0 1 0 .696 10.75 10.75 0 0 1-19.876 0"/><circle cx="12" cy="12" r="3"/>
								</svg>
								<div>
									<p class="text-[10px] font-bold font-mono text-zinc-800 leading-none">{item.riwayatDilihat[item.riwayatDilihat.length - 1]}</p>
									<p class="text-[8px] text-zinc-400 uppercase tracking-wider">Dilihat</p>
								</div>
								<svg viewBox="0 0 {SPARK_W} {SPARK_H}" class="w-12 h-5 ml-auto flex-shrink-0" preserveAspectRatio="none">
									<path d={areaPath(dilihatPts, SPARK_H, SPARK_PAD)} class="fill-slate-400/10" />
									<path d={linePath(dilihatPts)} fill="none" class="stroke-slate-400" stroke-width="1.5" />
								</svg>
							</div>
						</div>

					</div>
				</div>
			{/each}
		</div>
	</div>
</section>