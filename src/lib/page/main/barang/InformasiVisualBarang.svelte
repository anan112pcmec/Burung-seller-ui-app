<script lang="ts">
	// ///////////////////////////////////////////////////////////////////////
	// Types
	// ///////////////////////////////////////////////////////////////////////

	type RangeMode = 'minggu' | 'bulan' | 'tahun' | 'custom';

	interface BucketData {
		label: string;
		penjualan: number; // nominal Rp
		pembelian: number; // jumlah unit terjual
		barangBaru: number; // jumlah barang/varian baru ditambahkan
	}

	interface ProdukTerlaris {
		nama: string;
		unitTerjual: number;
	}

	interface Point {
		x: number;
		y: number;
	}

	// ///////////////////////////////////////////////////////////////////////
	// State — filter periode
	// ///////////////////////////////////////////////////////////////////////

	let rangeMode: RangeMode = $state('bulan');
	let customStart = $state('2026-08-01');
	let customEnd = $state('2026-08-27');

	const rangeOptions: { value: RangeMode; label: string }[] = [
		{ value: 'minggu', label: 'Per Minggu' },
		{ value: 'bulan', label: 'Per Bulan' },
		{ value: 'tahun', label: 'Per Tahun' },
		{ value: 'custom', label: 'Custom' }
	];

	// ///////////////////////////////////////////////////////////////////////
	// Mock data generator — ganti pemanggilan ini dengan fetch ke service asli
	// (transaksi_services buat penjualan/pembelian, barang_services buat
	// penambahan barang) berdasarkan rangeMode & rentang tanggal terpilih.
	// ///////////////////////////////////////////////////////////////////////

	function seedNumber(seed: number, min: number, max: number): number {
		const x = Math.sin(seed * 999.77) * 10000;
		const frac = x - Math.floor(x);
		return Math.round(min + frac * (max - min));
	}

	function bucketsForMode(mode: RangeMode, start: string, end: string): BucketData[] {
		if (mode === 'minggu') {
			const hari = ['Sen', 'Sel', 'Rab', 'Kam', 'Jum', 'Sab', 'Min'];
			return hari.map((h, i) => ({
				label: h,
				penjualan: seedNumber(i + 1, 900_000, 4_200_000),
				pembelian: seedNumber(i + 20, 8, 42),
				barangBaru: seedNumber(i + 40, 0, 5)
			}));
		}

		if (mode === 'bulan') {
			return Array.from({ length: 4 }, (_, i) => ({
				label: `Minggu ${i + 1}`,
				penjualan: seedNumber(i + 60, 5_500_000, 18_000_000),
				pembelian: seedNumber(i + 80, 60, 160),
				barangBaru: seedNumber(i + 100, 1, 10)
			}));
		}

		if (mode === 'tahun') {
			const bulan = ['Jan', 'Feb', 'Mar', 'Apr', 'Mei', 'Jun', 'Jul', 'Agu', 'Sep', 'Okt', 'Nov', 'Des'];
			return bulan.map((b, i) => ({
				label: b,
				penjualan: seedNumber(i + 120, 22_000_000, 68_000_000),
				pembelian: seedNumber(i + 140, 240, 620),
				barangBaru: seedNumber(i + 160, 3, 24)
			}));
		}

		// custom — bucket per hari di antara dua tanggal terpilih
		const tanggalMulai = new Date(start);
		const tanggalSelesai = new Date(end);
		const jumlahHari = Math.max(
			1,
			Math.min(31, Math.round((tanggalSelesai.getTime() - tanggalMulai.getTime()) / 86_400_000) + 1)
		);

		return Array.from({ length: jumlahHari }, (_, i) => {
			const tgl = new Date(tanggalMulai);
			tgl.setDate(tgl.getDate() + i);
			return {
				label: tgl.toLocaleDateString('id-ID', { day: '2-digit', month: 'short' }),
				penjualan: seedNumber(i + 200, 700_000, 3_800_000),
				pembelian: seedNumber(i + 220, 6, 38),
				barangBaru: seedNumber(i + 240, 0, 4)
			};
		});
	}

	const produkTerlarisMaster: ProdukTerlaris[] = [
		{ nama: 'Kemeja Linen Oversize', unitTerjual: 184 },
		{ nama: 'Rok Plisket Katun', unitTerjual: 152 },
		{ nama: 'Blouse Rajut Kancing', unitTerjual: 121 },
		{ nama: 'Celana Cargo Utility', unitTerjual: 96 },
		{ nama: 'Tote Bag Kanvas', unitTerjual: 74 }
	];

	// ///////////////////////////////////////////////////////////////////////
	// Derived — angka ringkasan
	// ///////////////////////////////////////////////////////////////////////

	let data = $derived(bucketsForMode(rangeMode, customStart, customEnd));

	let totalPenjualan = $derived(data.reduce((a, d) => a + d.penjualan, 0));
	let totalPembelian = $derived(data.reduce((a, d) => a + d.pembelian, 0));
	let totalBarangBaru = $derived(data.reduce((a, d) => a + d.barangBaru, 0));

	// delta dummy vs periode sebelumnya, buat kasih konteks naik/turun
	let deltaPenjualan = $derived(seedNumber(totalPenjualan % 97, -18, 26));
	let deltaPembelian = $derived(seedNumber(totalPembelian % 61, -12, 22));
	let deltaBarangBaru = $derived(seedNumber(totalBarangBaru % 13, -10, 30));

	function formatRupiah(n: number): string {
		if (n >= 1_000_000_000) return `Rp${(n / 1_000_000_000).toFixed(1)}M`;
		if (n >= 1_000_000) return `Rp${(n / 1_000_000).toFixed(1)}jt`;
		if (n >= 1_000) return `Rp${(n / 1_000).toFixed(0)}rb`;
		return `Rp${n}`;
	}

	// ///////////////////////////////////////////////////////////////////////
	// SVG chart helpers — dipakai bareng buat chart besar & sparkline kecil
	// ///////////////////////////////////////////////////////////////////////

	const CHART_W = 640;
	const CHART_H = 200;
	const PAD_X = 8;
	const PAD_Y = 16;

	const SPARK_W = 160;
	const SPARK_H = 40;
	const SPARK_PAD = 3;

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

	// chart utama — tren penjualan
	let penjualanPoints = $derived(toPoints(data.map((d) => d.penjualan), CHART_W, CHART_H, PAD_X));

	// label sumbu-x chart utama, diposisikan presisi pas di bawah tiap titik
	let visibleLabelIdx = $derived(
		data.map((_, i) => i).filter((i) => data.length <= 8 || i % Math.ceil(data.length / 8) === 0)
	);

	// sparkline kecil di tiap kartu KPI
	let sparkPenjualan = $derived(toPoints(data.map((d) => d.penjualan), SPARK_W, SPARK_H, SPARK_PAD));
	let sparkPembelian = $derived(data.map((d) => d.pembelian));
	let sparkBarangBaru = $derived(data.map((d) => d.barangBaru));
	let maxSparkPembelian = $derived(Math.max(...sparkPembelian, 1));
	let maxSparkBarangBaru = $derived(Math.max(...sparkBarangBaru, 1));

	// bar chart pembelian vs barang baru — dua skala independen biar pola tiap
	// metrik tetap kebaca meski rentang angkanya jauh berbeda
	let maxPembelian = $derived(Math.max(...data.map((d) => d.pembelian), 1));
	let maxBarangBaru = $derived(Math.max(...data.map((d) => d.barangBaru), 1));
</script>

<section id="informasi-visual-barang" class="w-full bg-white p-4 sm:p-6 lg:p-8 text-slate-950">
	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- HEADER + FILTER PERIODE -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="flex flex-col lg:flex-row lg:items-end lg:justify-between gap-4 pb-6 mb-6 border-b border-zinc-800/10">
		<div>
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
				ANALITIK BARANG
			</span>
			<h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
				Informasi Penjualan &amp; Barang
			</h1>
		</div>

		<div class="flex flex-col sm:flex-row sm:items-center gap-2.5">
			<div class="inline-flex border border-zinc-800/20 rounded-xs overflow-hidden self-start">
				{#each rangeOptions as opt}
					<button
						type="button"
						onclick={() => (rangeMode = opt.value)}
						class="px-3 py-1.5 text-[9px] sm:text-[10px] font-medium uppercase tracking-wider transition duration-200 {rangeMode === opt.value
							? 'bg-slate-950 text-white'
							: 'text-slate-600 hover:bg-slate-50'}"
					>
						{opt.label}
					</button>
				{/each}
			</div>

			{#if rangeMode === 'custom'}
				<div class="flex items-center gap-1.5 text-[10px] sm:text-[11px] font-mono">
					<input
						type="date"
						bind:value={customStart}
						class="border border-zinc-800/20 rounded-xs px-2 py-1.5 text-slate-950 focus:outline-none focus:border-slate-950"
					/>
					<span class="text-slate-400">—</span>
					<input
						type="date"
						bind:value={customEnd}
						class="border border-zinc-800/20 rounded-xs px-2 py-1.5 text-slate-950 focus:outline-none focus:border-slate-950"
					/>
				</div>
			{/if}
		</div>
	</div>

	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- SATU GRID BUAT SEMUA CARD — biar spacing & alignment konsisten -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="grid grid-cols-1 lg:grid-cols-12 gap-4">

		<!-- 1. TOTAL PENJUALAN — angka + sparkline tren -->
		<div class="lg:col-span-4 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<div class="flex items-start justify-between gap-3">
				<div class="min-w-0">
					<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
						TOTAL PENJUALAN
					</span>
					<div class="mt-2 flex items-baseline gap-2 flex-wrap">
						<span class="text-xl sm:text-2xl font-bold font-mono tracking-tight">
							{formatRupiah(totalPenjualan)}
						</span>
						<span class="text-[10px] sm:text-[11px] font-mono font-bold {deltaPenjualan >= 0 ? 'text-emerald-600' : 'text-rose-600'}">
							{deltaPenjualan >= 0 ? '+' : ''}{deltaPenjualan}%
						</span>
					</div>
				</div>
				<svg viewBox="0 0 {SPARK_W} {SPARK_H}" class="w-16 sm:w-20 h-8 sm:h-10 flex-shrink-0" preserveAspectRatio="none">
					<path d={areaPath(sparkPenjualan, SPARK_H, SPARK_PAD)} class="fill-slate-950/5" />
					<path d={linePath(sparkPenjualan)} fill="none" class="stroke-slate-950" stroke-width="1.5" />
				</svg>
			</div>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">vs periode sebelumnya</p>
		</div>

		<!-- 2. TOTAL PEMBELIAN — angka + mini bar sparkline -->
		<div class="lg:col-span-4 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<div class="flex items-start justify-between gap-3">
				<div class="min-w-0">
					<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
						TOTAL PEMBELIAN
					</span>
					<div class="mt-2 flex items-baseline gap-2 flex-wrap">
						<span class="text-xl sm:text-2xl font-bold font-mono tracking-tight">
							{totalPembelian.toLocaleString('id-ID')}
						</span>
						<span class="text-[10px] sm:text-[11px] text-slate-500 font-light">unit</span>
						<span class="text-[10px] sm:text-[11px] font-mono font-bold {deltaPembelian >= 0 ? 'text-emerald-600' : 'text-rose-600'}">
							{deltaPembelian >= 0 ? '+' : ''}{deltaPembelian}%
						</span>
					</div>
				</div>
				<div class="flex items-end gap-0.5 h-8 sm:h-10 w-16 sm:w-20 flex-shrink-0">
					{#each sparkPembelian as v}
						<div class="flex-1 bg-slate-950 rounded-t-[1px]" style:height="{(v / maxSparkPembelian) * 100}%"></div>
					{/each}
				</div>
			</div>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">vs periode sebelumnya</p>
		</div>

		<!-- 3. PENAMBAHAN BARANG — angka + mini bar sparkline -->
		<div class="lg:col-span-4 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<div class="flex items-start justify-between gap-3">
				<div class="min-w-0">
					<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
						PENAMBAHAN BARANG
					</span>
					<div class="mt-2 flex items-baseline gap-2 flex-wrap">
						<span class="text-xl sm:text-2xl font-bold font-mono tracking-tight">
							{totalBarangBaru.toLocaleString('id-ID')}
						</span>
						<span class="text-[10px] sm:text-[11px] text-slate-500 font-light">barang</span>
						<span class="text-[10px] sm:text-[11px] font-mono font-bold {deltaBarangBaru >= 0 ? 'text-emerald-600' : 'text-rose-600'}">
							{deltaBarangBaru >= 0 ? '+' : ''}{deltaBarangBaru}%
						</span>
					</div>
				</div>
				<div class="flex items-end gap-0.5 h-8 sm:h-10 w-16 sm:w-20 flex-shrink-0">
					{#each sparkBarangBaru as v}
						<div class="flex-1 bg-slate-300 rounded-t-[1px]" style:height="{(v / maxSparkBarangBaru) * 100}%"></div>
					{/each}
				</div>
			</div>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">vs periode sebelumnya</p>
		</div>

		<!-- 4. TREN PENJUALAN — line/area chart besar -->
		<div class="lg:col-span-7 border border-zinc-800/20 rounded-sm p-4 sm:p-5">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				TREN PENJUALAN
			</span>

			<div class="mt-3 w-full">
				<svg viewBox="0 0 {CHART_W} {CHART_H}" class="w-full h-[10rem] sm:h-[12rem]" preserveAspectRatio="none">
					<!-- garis bantu horizontal -->
					{#each [0.25, 0.5, 0.75] as g}
						<line
							x1={PAD_X}
							x2={CHART_W - PAD_X}
							y1={PAD_Y + (CHART_H - PAD_Y * 2) * g}
							y2={PAD_Y + (CHART_H - PAD_Y * 2) * g}
							stroke="currentColor"
							class="text-zinc-800/10"
							stroke-width="1"
						/>
					{/each}

					<path d={areaPath(penjualanPoints, CHART_H, PAD_X)} class="fill-slate-950/5" />
					<path d={linePath(penjualanPoints)} fill="none" class="stroke-slate-950" stroke-width="2" />

					{#each penjualanPoints as p}
						<circle cx={p.x} cy={p.y} r="2.5" class="fill-slate-950" />
					{/each}
				</svg>

				<!-- label sumbu-x, posisinya dihitung persis dari koordinat titik di atas -->
				<div class="relative h-4 mt-1">
					{#each visibleLabelIdx as i}
						<span
							class="absolute -translate-x-1/2 text-[8px] sm:text-[9px] font-mono text-slate-400 uppercase tracking-wider whitespace-nowrap"
							style:left="{(penjualanPoints[i].x / CHART_W) * 100}%"
						>
							{data[i].label}
						</span>
					{/each}
				</div>
			</div>
		</div>

		<!-- 5. PRODUK TERLARIS — horizontal bar list -->
		<div class="lg:col-span-5 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				PRODUK TERLARIS
			</span>

			<div class="mt-4 space-y-3 flex-1 flex flex-col justify-center">
				{#each produkTerlarisMaster as p, i}
					{@const maxUnitProduk = produkTerlarisMaster[0].unitTerjual}
					<div>
						<div class="flex items-center justify-between mb-1">
							<span class="text-[10px] sm:text-[11px] font-medium truncate flex items-center gap-1.5">
								<span class="text-slate-400 font-mono">0{i + 1}</span>
								{p.nama}
							</span>
							<span class="text-[10px] sm:text-[11px] font-bold font-mono flex-shrink-0 ml-2">
								{p.unitTerjual}
							</span>
						</div>
						<div class="w-full h-1 bg-slate-100 rounded-full overflow-hidden">
							<div
								class="h-full bg-slate-950 rounded-full"
								style:width="{(p.unitTerjual / maxUnitProduk) * 100}%"
							></div>
						</div>
					</div>
				{/each}
			</div>
		</div>

		<!-- 6. PERBANDINGAN PEMBELIAN VS PENAMBAHAN BARANG — grouped bar chart -->
		<div class="lg:col-span-12 border border-zinc-800/20 rounded-sm p-4 sm:p-5">
			<div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-2">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					PEMBELIAN VS PENAMBAHAN BARANG
				</span>
				<div class="flex items-center gap-3 text-[9px] sm:text-[10px] font-medium uppercase tracking-wider">
					<span class="flex items-center gap-1.5">
						<span class="w-2 h-2 rounded-full bg-slate-950"></span>
						Pembelian (unit)
					</span>
					<span class="flex items-center gap-1.5">
						<span class="w-2 h-2 rounded-full bg-slate-300"></span>
						Barang Baru
					</span>
				</div>
			</div>

			<div class="mt-4 w-full overflow-x-auto">
				<div class="flex items-end gap-3 sm:gap-4 h-[9rem] sm:h-[10rem] min-w-[480px]">
					{#each data as d}
						<div class="flex-1 flex flex-col items-center justify-end h-full min-w-[1.5rem]">
							<div class="w-full flex items-end justify-center gap-1 h-full">
								<div
									class="w-2 sm:w-3 bg-slate-950 rounded-t-xs"
									style:height="{(d.pembelian / maxPembelian) * 100}%"
								></div>
								<div
									class="w-2 sm:w-3 bg-slate-300 rounded-t-xs"
									style:height="{(d.barangBaru / maxBarangBaru) * 100}%"
								></div>
							</div>
							<span class="mt-2 text-[8px] sm:text-[9px] font-mono text-slate-400 uppercase tracking-wider truncate max-w-full">
								{d.label}
							</span>
						</div>
					{/each}
				</div>
			</div>
		</div>

	</div>
</section>