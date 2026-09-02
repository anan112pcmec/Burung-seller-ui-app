<script lang="ts">
	// ///////////////////////////////////////////////////////////////////////
	// Types
	// ///////////////////////////////////////////////////////////////////////

	type RangeMode = 'hari' | 'minggu' | 'bulan' | 'tahun' | 'selamanya' | 'custom';

	interface BucketData {
		label: string;
		pemasukan: number;
	}

	interface KontribusiKategori {
		nama: string;
		omset: number;
	}

	interface BarangTerhubung {
		idKategoriBarang: number;
		namaBarangInduk: string;
		namaVarian: string;
		sku: string;
		harga: number;
		stok: number;
		kontribusiOmset: number;
	}

	interface Point {
		x: number;
		y: number;
	}

	// ///////////////////////////////////////////////////////////////////////
	// Mock data — Rekening yang sedang dibuka
	// ///////////////////////////////////////////////////////////////////////

	const rekening = {
		id_rekening_seller: 1,
		nama_bank: 'BCA',
		nomor_rekening: '4520019283',
		pemilik_rekening: 'Ahmad Fauzan',
		is_default: true
	};

	function maskNorek(nomor: string): string {
		if (nomor.length <= 4) return nomor;
		return `${'•'.repeat(Math.max(nomor.length - 4, 0))}${nomor.slice(-4)}`;
	}

	// ///////////////////////////////////////////////////////////////////////
	// State — filter periode tren pemasukan
	// ///////////////////////////////////////////////////////////////////////

	let rangeMode = $state<RangeMode>('bulan');
	let customStart = $state('2026-08-01');
	let customEnd = $state('2026-08-27');

	const rangeOptions: { value: RangeMode; label: string }[] = [
		{ value: 'hari', label: 'Per Hari' },
		{ value: 'minggu', label: 'Per Minggu' },
		{ value: 'bulan', label: 'Per Bulan' },
		{ value: 'tahun', label: 'Per Tahun' },
		{ value: 'selamanya', label: 'Selama Berjalan' },
		{ value: 'custom', label: 'Custom' }
	];

	function seedNumber(seed: number, min: number, max: number): number {
		const x = Math.sin(seed * 999.77) * 10000;
		const frac = x - Math.floor(x);
		return Math.round(min + frac * (max - min));
	}

	function bucketsForMode(mode: RangeMode, start: string, end: string): BucketData[] {
		if (mode === 'hari') {
			const jam = ['00:00', '03:00', '06:00', '09:00', '12:00', '15:00', '18:00', '21:00'];
			return jam.map((j, i) => ({ label: j, pemasukan: seedNumber(i + 1, 200_000, 1_800_000) }));
		}
		if (mode === 'minggu') {
			const hari = ['Sen', 'Sel', 'Rab', 'Kam', 'Jum', 'Sab', 'Min'];
			return hari.map((h, i) => ({ label: h, pemasukan: seedNumber(i + 40, 1_500_000, 9_000_000) }));
		}
		if (mode === 'bulan') {
			return Array.from({ length: 4 }, (_, i) => ({
				label: `Minggu ${i + 1}`,
				pemasukan: seedNumber(i + 80, 8_000_000, 32_000_000)
			}));
		}
		if (mode === 'tahun' || mode === 'selamanya') {
			const bulan = ['Jan', 'Feb', 'Mar', 'Apr', 'Mei', 'Jun', 'Jul', 'Agu', 'Sep', 'Okt', 'Nov', 'Des'];
			return bulan.map((b, i) => ({
				label: b,
				pemasukan: seedNumber(i + 120, 30_000_000, 110_000_000)
			}));
		}
		const tanggalMulai = new Date(start);
		const tanggalSelesai = new Date(end);
		const jumlahHari = Math.max(1, Math.min(31, Math.round((tanggalSelesai.getTime() - tanggalMulai.getTime()) / 86_400_000) + 1));
		return Array.from({ length: jumlahHari }, (_, i) => {
			const tgl = new Date(tanggalMulai);
			tgl.setDate(tgl.getDate() + i);
			return {
				label: tgl.toLocaleDateString('id-ID', { day: '2-digit', month: 'short' }),
				pemasukan: seedNumber(i + 160, 1_000_000, 7_000_000)
			};
		});
	}

	let dataTren = $derived(bucketsForMode(rangeMode, customStart, customEnd));
	let totalPemasukanPeriode = $derived(dataTren.reduce((a, d) => a + d.pemasukan, 0));
	let deltaPemasukan = $derived(seedNumber(totalPemasukanPeriode % 97, -8, 22));

	function formatRupiah(n: number): string {
		if (n >= 1_000_000_000) return `Rp${(n / 1_000_000_000).toFixed(1)}M`;
		if (n >= 1_000_000) return `Rp${(n / 1_000_000).toFixed(1)}jt`;
		if (n >= 1_000) return `Rp${(n / 1_000).toFixed(0)}rb`;
		return `Rp${n}`;
	}

	function formatRupiahPenuh(n: number): string {
		return `Rp${n.toLocaleString('id-ID')}`;
	}

	// ///////////////////////////////////////////////////////////////////////
	// SVG chart helper — tren garis
	// ///////////////////////////////////////////////////////////////////////

	const CHART_W = 640;
	const CHART_H = 180;
	const PAD_X = 8;
	const PAD_Y = 14;

	function toPoints(values: number[], w: number, h: number, pad: number): Point[] {
		const max = Math.max(...values, 1);
		const usableW = w - pad * 2;
		const usableH = h - pad * 2;
		const step = values.length > 1 ? usableW / (values.length - 1) : 0;
		return values.map((v, i) => ({ x: pad + step * i, y: pad + usableH - (v / max) * usableH }));
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

	let trenPoints = $derived(toPoints(dataTren.map((d) => d.pemasukan), CHART_W, CHART_H, PAD_X));
	let visibleLabelIdx = $derived(dataTren.map((_, i) => i).filter((i) => dataTren.length <= 8 || i % Math.ceil(dataTren.length / 8) === 0));

	// ///////////////////////////////////////////////////////////////////////
	// Mock data — kontribusi per kategori (buat donut top 10 + lainnya)
	// ///////////////////////////////////////////////////////////////////////

	const semuaKontribusi: KontribusiKategori[] = [
		{ nama: 'Kaos Polos Combed 30s - Hitam M', omset: 42_500_000 },
		{ nama: 'Kemeja Flanel Kotak-kotak - Merah L', omset: 31_200_000 },
		{ nama: 'Celana Chino Slimfit - Khaki 32', omset: 24_800_000 },
		{ nama: 'Jaket Bomber Varsity - Navy L', omset: 21_100_000 },
		{ nama: 'Tumbler Stainless 500ml - Sage', omset: 15_600_000 },
		{ nama: 'Matras Yoga 6mm - Ungu', omset: 12_300_000 },
		{ nama: 'Sepatu Sneakers Canvas - Putih 42', omset: 10_900_000 },
		{ nama: 'Serum Niacinamide 20ml', omset: 8_700_000 },
		{ nama: 'Case iPhone 15 Pro - Bening', omset: 7_400_000 },
		{ nama: 'Topi Bucket Denim', omset: 6_100_000 },
		{ nama: 'Kaos Kaki Rajut 3-Pack', omset: 4_200_000 },
		{ nama: 'Gantungan Kunci Kulit', omset: 2_800_000 },
		{ nama: 'Sticker Pack Vinyl', omset: 1_900_000 }
	];

	let kontribusiTerurut = $derived([...semuaKontribusi].sort((a, b) => b.omset - a.omset));
	let top10Kontribusi = $derived(kontribusiTerurut.slice(0, 10));
	let sisaLainnya = $derived(kontribusiTerurut.slice(10).reduce((a, k) => a + k.omset, 0));
	let totalSemuaOmset = $derived(kontribusiTerurut.reduce((a, k) => a + k.omset, 0));

	let segmenDonut = $derived(() => {
		const items = [...top10Kontribusi];
		if (sisaLainnya > 0) items.push({ nama: 'Lainnya', omset: sisaLainnya });
		const total = totalSemuaOmset || 1;
		let offsetKumulatif = 0;
		return items.map((item, i) => {
			const persen = (item.omset / total) * 100;
			const segmen = { ...item, persen, offset: offsetKumulatif, warna: i === items.length - 1 && item.nama === 'Lainnya' ? '#d4d4d8' : `hsl(${168 + i * 14}, 55%, ${42 - i * 1.5}%)` };
			offsetKumulatif += persen;
			return segmen;
		});
	});

	// ///////////////////////////////////////////////////////////////////////
	// KPI turunan
	// ///////////////////////////////////////////////////////////////////////

	const totalTransaksiSepanjangWaktu = 428;
	let rataRataPerTransaksi = $derived(Math.round(totalSemuaOmset / Math.max(totalTransaksiSepanjangWaktu, 1)));

	// ///////////////////////////////////////////////////////////////////////
	// Mock data — SECTION 2: full list barang terhubung ke rekening ini
	// ///////////////////////////////////////////////////////////////////////

	let pencarian = $state('');

	const barangTerhubung: BarangTerhubung[] = [
		{ idKategoriBarang: 1, namaBarangInduk: 'Kaos Polos Combed 30s', namaVarian: 'Hitam, size M', sku: 'KPC-HT-M', harga: 65_000, stok: 128, kontribusiOmset: 42_500_000 },
		{ idKategoriBarang: 2, namaBarangInduk: 'Kemeja Flanel Kotak-kotak', namaVarian: 'Merah, size L', sku: 'KFK-MR-L', harga: 159_000, stok: 42, kontribusiOmset: 31_200_000 },
		{ idKategoriBarang: 3, namaBarangInduk: 'Celana Chino Slimfit', namaVarian: 'Khaki, size 32', sku: 'CCS-KH-32', harga: 189_000, stok: 7, kontribusiOmset: 24_800_000 },
		{ idKategoriBarang: 4, namaBarangInduk: 'Jaket Bomber Varsity', namaVarian: 'Navy, size L', sku: 'JBV-NV-L', harga: 245_000, stok: 5, kontribusiOmset: 21_100_000 },
		{ idKategoriBarang: 5, namaBarangInduk: 'Tumbler Stainless 500ml', namaVarian: 'Sage Green', sku: 'TMB-SS-500', harga: 65_000, stok: 210, kontribusiOmset: 15_600_000 },
		{ idKategoriBarang: 6, namaBarangInduk: 'Matras Yoga Anti Slip', namaVarian: '6mm, Ungu', sku: 'MYG-6-UN', harga: 112_000, stok: 36, kontribusiOmset: 12_300_000 },
		{ idKategoriBarang: 7, namaBarangInduk: 'Sepatu Sneakers Canvas', namaVarian: 'Putih, size 42', sku: 'SSC-PT-42', harga: 210_000, stok: 19, kontribusiOmset: 10_900_000 },
		{ idKategoriBarang: 8, namaBarangInduk: 'Serum Wajah Brightening', namaVarian: 'Niacinamide 20ml', sku: 'SRM-NC-20', harga: 79_000, stok: 7, kontribusiOmset: 8_700_000 },
		{ idKategoriBarang: 9, namaBarangInduk: 'Case iPhone 15 Pro', namaVarian: 'Bening', sku: 'CIP-15P-BN', harga: 45_000, stok: 5, kontribusiOmset: 7_400_000 },
		{ idKategoriBarang: 10, namaBarangInduk: 'Topi Bucket Denim', namaVarian: 'Biru Washed', sku: 'TBD-BW', harga: 89_000, stok: 24, kontribusiOmset: 6_100_000 },
		{ idKategoriBarang: 11, namaBarangInduk: 'Kaos Kaki Rajut', namaVarian: '3-Pack, Mix Warna', sku: 'KKR-3PK', harga: 39_000, stok: 65, kontribusiOmset: 4_200_000 },
		{ idKategoriBarang: 12, namaBarangInduk: 'Gantungan Kunci Kulit', namaVarian: 'Cokelat Tua', sku: 'GKK-CT', harga: 25_000, stok: 88, kontribusiOmset: 2_800_000 },
		{ idKategoriBarang: 13, namaBarangInduk: 'Sticker Pack Vinyl', namaVarian: 'Isi 10', sku: 'STK-V10', harga: 15_000, stok: 140, kontribusiOmset: 1_900_000 }
	];

	let barangTersaring = $derived(
		barangTerhubung
			.filter((b) => (b.namaBarangInduk + b.namaVarian).toLowerCase().includes(pencarian.toLowerCase()))
			.sort((a, b) => b.kontribusiOmset - a.kontribusiOmset)
	);
</script>

<section id="details-rekening" class="w-full bg-white p-4 sm:p-6 lg:p-8 text-slate-950">
	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- BREADCRUMB + IDENTITAS REKENING -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-3 pb-5 mb-6 border-b border-zinc-800/10">
		<div>
			<span class="text-[9px] sm:text-[10px] font-mono text-slate-400 uppercase tracking-wider">
				Rekening Toko / {rekening.nama_bank}
			</span>
			<h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
				{rekening.nama_bank} · {maskNorek(rekening.nomor_rekening)}
			</h1>
			<p class="text-[11px] text-zinc-500 mt-1">a.n. {rekening.pemilik_rekening}</p>
		</div>
		{#if rekening.is_default}
			<span class="px-2 py-1 rounded text-[9px] font-medium uppercase tracking-wider bg-teal-50 text-teal-700 self-start">
				Rekening Default
			</span>
		{/if}
	</div>

	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- SECTION 1 — DATA VISUALISASI -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="pb-8 mb-8 border-b border-zinc-800/10">
		<div class="grid grid-cols-1 lg:grid-cols-12 gap-4">
			<!-- 3. TOTAL PEMASUKAN SEPANJANG WAKTU -->
			<div class="lg:col-span-4 border border-zinc-800/20 rounded-sm p-4 flex flex-col justify-between">
				<span class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">Total Pemasukan Sepanjang Waktu</span>
				<span class="mt-2 text-2xl font-bold font-mono tracking-tight text-teal-700">{formatRupiah(totalSemuaOmset)}</span>
				<p class="text-[9px] text-slate-400 font-light mt-2">dari {totalTransaksiSepanjangWaktu} transaksi</p>
			</div>

			<!-- 4. RATA-RATA PEMASUKAN PER TRANSAKSI -->
			<div class="lg:col-span-4 border border-zinc-800/20 rounded-sm p-4 flex flex-col justify-between">
				<span class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">Rata-rata / Transaksi</span>
				<span class="mt-2 text-2xl font-bold font-mono tracking-tight">{formatRupiah(rataRataPerTransaksi)}</span>
				<p class="text-[9px] text-slate-400 font-light mt-2">total pemasukan ÷ jumlah transaksi</p>
			</div>

			<!-- 5. TOTAL BARANG TERHUBUNG -->
			<div class="lg:col-span-4 border border-zinc-800/20 rounded-sm p-4 flex flex-col justify-between">
				<span class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">Total Barang Terhubung</span>
				<span class="mt-2 text-2xl font-bold font-mono tracking-tight">{barangTerhubung.length}</span>
				<p class="text-[9px] text-slate-400 font-light mt-2">varian yang mengarah ke rekening ini</p>
			</div>

			<!-- 2. TREN PEMASUKAN -->
			<div class="lg:col-span-12 border border-zinc-800/20 rounded-sm p-4 sm:p-5">
				<div class="flex flex-col lg:flex-row lg:items-end lg:justify-between gap-3 mb-3">
					<div>
						<span class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">Tren Pemasukan</span>
						<div class="flex items-baseline gap-2 mt-1">
							<span class="text-lg font-bold font-mono">{formatRupiah(totalPemasukanPeriode)}</span>
							<span class="text-[10px] font-mono font-bold {deltaPemasukan >= 0 ? 'text-teal-600' : 'text-rose-600'}">
								{deltaPemasukan >= 0 ? '+' : ''}{deltaPemasukan}%
							</span>
						</div>
					</div>
					<div class="inline-flex border border-zinc-800/20 rounded-xs overflow-hidden self-start">
						{#each rangeOptions as opt}
							<button
								type="button"
								onclick={() => (rangeMode = opt.value)}
								class="px-2.5 py-1.5 text-[9px] font-medium uppercase tracking-wider transition duration-200 {rangeMode === opt.value
									? 'bg-slate-950 text-white'
									: 'text-slate-600 hover:bg-slate-50'}"
							>
								{opt.label}
							</button>
						{/each}
					</div>
				</div>

				{#if rangeMode === 'custom'}
					<div class="flex items-center gap-1.5 text-[10px] font-mono mb-3">
						<input type="date" bind:value={customStart} class="border border-zinc-800/20 rounded-xs px-2 py-1.5 text-slate-950 focus:outline-none focus:border-slate-950" />
						<span class="text-slate-400">—</span>
						<input type="date" bind:value={customEnd} class="border border-zinc-800/20 rounded-xs px-2 py-1.5 text-slate-950 focus:outline-none focus:border-slate-950" />
					</div>
				{/if}

				<svg viewBox="0 0 {CHART_W} {CHART_H}" class="w-full h-[9rem]" preserveAspectRatio="none">
					{#each [0.25, 0.5, 0.75] as g}
						<line x1={PAD_X} x2={CHART_W - PAD_X} y1={PAD_Y + (CHART_H - PAD_Y * 2) * g} y2={PAD_Y + (CHART_H - PAD_Y * 2) * g} stroke="currentColor" class="text-zinc-800/10" stroke-width="1" />
					{/each}
					<path d={areaPath(trenPoints, CHART_H, PAD_X)} class="fill-teal-600/10" />
					<path d={linePath(trenPoints)} fill="none" class="stroke-teal-600" stroke-width="2" />
				</svg>
				<div class="relative h-4 mt-1">
					{#each visibleLabelIdx as i}
						<span class="absolute -translate-x-1/2 text-[8px] font-mono text-slate-400 uppercase tracking-wider whitespace-nowrap" style:left="{(trenPoints[i].x / CHART_W) * 100}%">
							{dataTren[i].label}
						</span>
					{/each}
				</div>
			</div>

			<!-- 1. DISTRIBUSI TOP 10 KATEGORI + LAINNYA -->
			<div class="lg:col-span-12 border border-zinc-800/20 rounded-sm p-4 sm:p-5">
				<span class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					Top 10 Kontributor Pemasukan
				</span>
				<span class="block text-[9px] text-slate-400 font-light mt-0.5">sisanya digabung jadi "Lainnya"</span>

				<div class="mt-4 flex flex-col lg:flex-row items-center gap-6">
					<!-- Donut -->
					<div class="relative w-40 h-40 sm:w-48 sm:h-48 flex-shrink-0">
						<svg viewBox="0 0 36 36" class="w-full h-full -rotate-90">
							<circle cx="18" cy="18" r="15.915" fill="none" class="stroke-zinc-100" stroke-width="4" />
							{#each segmenDonut() as seg}
								<circle
									cx="18" cy="18" r="15.915" fill="none"
									stroke={seg.warna}
									stroke-width="4"
									stroke-dasharray="{seg.persen} {100 - seg.persen}"
									stroke-dashoffset={-seg.offset}
								/>
							{/each}
						</svg>
						<div class="absolute inset-0 flex flex-col items-center justify-center">
							<span class="text-[9px] text-zinc-400 uppercase tracking-wider">Total</span>
							<span class="text-sm font-bold font-mono text-zinc-800">{formatRupiah(totalSemuaOmset)}</span>
						</div>
					</div>

					<!-- Legend -->
					<div class="flex-1 w-full grid grid-cols-1 sm:grid-cols-2 gap-x-6 gap-y-1.5">
						{#each segmenDonut() as seg, i}
							<div class="flex items-center justify-between gap-2 text-[10px]">
								<span class="flex items-center gap-1.5 min-w-0">
									<span class="w-2 h-2 rounded-full flex-shrink-0" style:background-color={seg.warna}></span>
									<span class="truncate text-zinc-600">{i + 1}. {seg.nama}</span>
								</span>
								<span class="font-mono font-semibold text-zinc-800 flex-shrink-0">{seg.persen.toFixed(1)}%</span>
							</div>
						{/each}
					</div>
				</div>
			</div>
		</div>
	</div>

	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- SECTION 2 — LIST BARANG TERHUBUNG (FULL) -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div>
		<div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-3 pb-3 mb-4 border-b border-zinc-800/10">
			<div>
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
					SEMUA BARANG TERHUBUNG
				</span>
				<h2 class="mt-1 text-base sm:text-lg font-bold uppercase tracking-tight leading-none">
					{barangTerhubung.length} Varian
				</h2>
			</div>
			<div class="relative w-full sm:w-64">
				<svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4 absolute left-3 top-1/2 -translate-y-1/2 text-zinc-400" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.75">
					<circle cx="11" cy="11" r="8"/><path d="m21 21-4.3-4.3"/>
				</svg>
				<input
					type="text"
					bind:value={pencarian}
					placeholder="Cari barang..."
					class="w-full bg-white border border-zinc-300 rounded-md pl-9 pr-3 py-2 text-xs text-zinc-800 placeholder-zinc-400 focus:outline-none focus:border-zinc-800 transition-colors"
				/>
			</div>
		</div>

		<div class="overflow-x-auto">
			<table class="w-full text-left border-collapse min-w-[720px]">
				<thead>
					<tr class="text-[9px] text-zinc-400 uppercase tracking-wider font-mono border-b border-zinc-200">
						<th class="py-2 pr-3 font-medium">Barang</th>
						<th class="py-2 pr-3 font-medium">SKU</th>
						<th class="py-2 pr-3 font-medium text-right">Harga</th>
						<th class="py-2 pr-3 font-medium text-right">Stok</th>
						<th class="py-2 pr-3 font-medium text-right">Kontribusi Pemasukan</th>
						<th class="py-2 pr-0 font-medium text-right">% dari Total</th>
					</tr>
				</thead>
				<tbody>
					{#each barangTersaring as b (b.idKategoriBarang)}
						<tr class="border-b border-zinc-100 hover:bg-zinc-50/60 transition-colors">
							<td class="py-2.5 pr-3">
								<p class="text-[11px] font-semibold text-zinc-800">{b.namaBarangInduk}</p>
								<p class="text-[9px] text-zinc-400">{b.namaVarian}</p>
							</td>
							<td class="py-2.5 pr-3 text-[10px] font-mono text-zinc-500">{b.sku}</td>
							<td class="py-2.5 pr-3 text-[10px] font-mono text-zinc-700 text-right">{formatRupiahPenuh(b.harga)}</td>
							<td class="py-2.5 pr-3 text-[10px] font-mono text-zinc-700 text-right">{b.stok}</td>
							<td class="py-2.5 pr-3 text-[11px] font-mono font-bold text-zinc-900 text-right">{formatRupiahPenuh(b.kontribusiOmset)}</td>
							<td class="py-2.5 pr-0 text-[10px] font-mono text-teal-700 text-right">
								{((b.kontribusiOmset / totalSemuaOmset) * 100).toFixed(1)}%
							</td>
						</tr>
					{/each}
				</tbody>
			</table>

			{#if barangTersaring.length === 0}
				<p class="text-[11px] text-zinc-400 text-center py-10">Tidak ada barang yang cocok dengan pencarian.</p>
			{/if}
		</div>
	</div>
</section>