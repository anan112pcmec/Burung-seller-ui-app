<script lang="ts">
	// ///////////////////////////////////////////////////////////////////////
	// Types
	// ///////////////////////////////////////////////////////////////////////

	type RangeMode = 'hari' | 'minggu' | 'bulan' | 'tahun' | 'custom';

	interface BucketData {
		label: string;
		pendapatan: number; // Rp, gabungan semua rekening
	}

	interface RekeningRingkas {
		idRekeningSeller: number;
		namaBank: string;
		nomorRekening: string;
		isDefault: boolean;
		jumlahBarangTerhubung: number; // count KategoriBarang WHERE id_rekening = ini
		jumlahTransaksi: number; // count Transaksi (join kategori_barang.id_rekening = ini)
		omsetSepanjangWaktu: number; // sum Transaksi.Total (join kategori_barang.id_rekening = ini)
	}

	interface Point {
		x: number;
		y: number;
	}

	// ///////////////////////////////////////////////////////////////////////
	// Mock data master rekening — ganti dengan fetch list RekeningSeller +
	// agregasi omset/transaksi per rekening
	// ///////////////////////////////////////////////////////////////////////

	function maskNorek(nomor: string): string {
		if (nomor.length <= 4) return nomor;
		return `${'•'.repeat(Math.max(nomor.length - 4, 0))}${nomor.slice(-4)}`;
	}

	const rekeningRingkasList: RekeningRingkas[] = [
		{
			idRekeningSeller: 1,
			namaBank: 'BCA',
			nomorRekening: '4520019283',
			isDefault: true,
			jumlahBarangTerhubung: 34,
			jumlahTransaksi: 842,
			omsetSepanjangWaktu: 186_400_000
		},
		{
			idRekeningSeller: 2,
			namaBank: 'Mandiri',
			nomorRekening: '1330008812345',
			isDefault: false,
			jumlahBarangTerhubung: 8,
			jumlahTransaksi: 196,
			omsetSepanjangWaktu: 41_200_000
		},
		{
			idRekeningSeller: 3,
			namaBank: 'Bank Syariah Indonesia (BSI)',
			nomorRekening: '7710092345',
			isDefault: false,
			jumlahBarangTerhubung: 0,
			jumlahTransaksi: 0,
			omsetSepanjangWaktu: 0
		}
	];

	// ///////////////////////////////////////////////////////////////////////
	// State — filter periode
	// ///////////////////////////////////////////////////////////////////////

	let rangeMode = $state<RangeMode>('bulan');
	let customStart = $state('2026-08-01');
	let customEnd = $state('2026-08-27');

	const rangeOptions: { value: RangeMode; label: string }[] = [
		{ value: 'hari', label: 'Per Hari' },
		{ value: 'minggu', label: 'Per Minggu' },
		{ value: 'bulan', label: 'Per Bulan' },
		{ value: 'tahun', label: 'Per Tahun' },
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
			return jam.map((j, i) => ({ label: j, pendapatan: seedNumber(i + 1, 300_000, 2_200_000) }));
		}
		if (mode === 'minggu') {
			const hari = ['Sen', 'Sel', 'Rab', 'Kam', 'Jum', 'Sab', 'Min'];
			return hari.map((h, i) => ({ label: h, pendapatan: seedNumber(i + 40, 2_000_000, 11_000_000) }));
		}
		if (mode === 'bulan') {
			return Array.from({ length: 4 }, (_, i) => ({
				label: `Minggu ${i + 1}`,
				pendapatan: seedNumber(i + 80, 10_000_000, 38_000_000)
			}));
		}
		if (mode === 'tahun') {
			const bulan = ['Jan', 'Feb', 'Mar', 'Apr', 'Mei', 'Jun', 'Jul', 'Agu', 'Sep', 'Okt', 'Nov', 'Des'];
			return bulan.map((b, i) => ({
				label: b,
				pendapatan: seedNumber(i + 120, 35_000_000, 120_000_000)
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
				pendapatan: seedNumber(i + 160, 1_200_000, 8_500_000)
			};
		});
	}

	let data = $derived(bucketsForMode(rangeMode, customStart, customEnd));
	let totalPendapatanPeriode = $derived(data.reduce((a, d) => a + d.pendapatan, 0));
	let deltaPendapatan = $derived(seedNumber(totalPendapatanPeriode % 97, -10, 24));

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
	// Derived — KPI turunan
	// ///////////////////////////////////////////////////////////////////////

	let totalRekening = $derived(rekeningRingkasList.length);
	let totalOmsetSemuaRekening = $derived(rekeningRingkasList.reduce((a, r) => a + r.omsetSepanjangWaktu, 0));
	let rataRataPerRekening = $derived(totalRekening > 0 ? Math.round(totalOmsetSemuaRekening / totalRekening) : 0);
	let rekeningBelumProduktif = $derived(rekeningRingkasList.filter((r) => r.jumlahBarangTerhubung === 0 || r.omsetSepanjangWaktu === 0));

	// ///////////////////////////////////////////////////////////////////////
	// SVG chart helper — tren garis
	// ///////////////////////////////////////////////////////////////////////

	const CHART_W = 640;
	const CHART_H = 200;
	const PAD_X = 8;
	const PAD_Y = 16;

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

	let pendapatanPoints = $derived(toPoints(data.map((d) => d.pendapatan), CHART_W, CHART_H, PAD_X));
	let visibleLabelIdx = $derived(data.map((_, i) => i).filter((i) => data.length <= 8 || i % Math.ceil(data.length / 8) === 0));

	// ///////////////////////////////////////////////////////////////////////
	// Donut — distribusi pendapatan per rekening
	// ///////////////////////////////////////////////////////////////////////

	const paletDonut = ['#0d9488', '#334155', '#0891b2', '#4f46e5', '#71717a'];

	let segmenDonut = $derived(() => {
		const total = totalOmsetSemuaRekening || 1;
		let offsetKumulatif = 0;
		return [...rekeningRingkasList]
			.sort((a, b) => b.omsetSepanjangWaktu - a.omsetSepanjangWaktu)
			.map((r, i) => {
				const persen = (r.omsetSepanjangWaktu / total) * 100;
				const segmen = {
					label: `${r.namaBank} ${maskNorek(r.nomorRekening)}`,
					persen,
					offset: offsetKumulatif,
					warna: paletDonut[i % paletDonut.length]
				};
				offsetKumulatif += persen;
				return segmen;
			});
	});

	// ///////////////////////////////////////////////////////////////////////
	// Rincian rekening produktif — ranking + status
	// ///////////////////////////////////////////////////////////////////////

	let rincianTerurut = $derived([...rekeningRingkasList].sort((a, b) => b.omsetSepanjangWaktu - a.omsetSepanjangWaktu));

	function andilPersen(r: RekeningRingkas): number {
		if (totalOmsetSemuaRekening === 0) return 0;
		return (r.omsetSepanjangWaktu / totalOmsetSemuaRekening) * 100;
	}

	function rataRataPerTransaksi(r: RekeningRingkas): number {
		if (r.jumlahTransaksi === 0) return 0;
		return Math.round(r.omsetSepanjangWaktu / r.jumlahTransaksi);
	}
</script>

<section id="overview-rekening" class="w-full bg-white text-slate-950">
	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- HEADER + FILTER PERIODE -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="flex flex-col lg:flex-row lg:items-end lg:justify-between gap-4 pb-6 mb-6 border-b border-zinc-800/10">
		<div>
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
				ANALITIK REKENING
			</span>
			<h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
				Ringkasan Pencairan Dana
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

	<div class="grid grid-cols-1 lg:grid-cols-12 gap-4">

		<!-- ///////////////////////////////////////////////////////////// -->
		<!-- ROW A — 4 KPI utama -->
		<!-- ///////////////////////////////////////////////////////////// -->

		<!-- 1. TOTAL PENDAPATAN TOKO (PERIODE) -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				TOTAL PENDAPATAN
			</span>
			<div class="mt-2 flex items-baseline gap-2 flex-wrap">
				<span class="text-xl sm:text-2xl font-bold font-mono tracking-tight text-teal-700">
					{formatRupiah(totalPendapatanPeriode)}
				</span>
				<span class="text-[10px] font-mono font-bold {deltaPendapatan >= 0 ? 'text-teal-600' : 'text-rose-600'}">
					{deltaPendapatan >= 0 ? '+' : ''}{deltaPendapatan}%
				</span>
			</div>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">gabungan semua rekening, sesuai periode</p>
		</div>

		<!-- 2. TOTAL REKENING TERDAFTAR -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				TOTAL REKENING
			</span>
			<span class="mt-2 text-2xl sm:text-3xl font-bold font-mono tracking-tight">
				{totalRekening}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">rekening pencairan dana terdaftar</p>
		</div>

		<!-- 3. RATA-RATA PENDAPATAN PER REKENING -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				RATA-RATA / REKENING
			</span>
			<span class="mt-2 text-xl sm:text-2xl font-bold font-mono tracking-tight">
				{formatRupiah(rataRataPerRekening)}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">sepanjang waktu, bukan per periode</p>
		</div>

		<!-- 4. REKENING BELUM PRODUKTIF — actionable -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<div class="flex items-center justify-between gap-2">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					BELUM PRODUKTIF
				</span>
				{#if rekeningBelumProduktif.length > 0}
					<span class="w-1.5 h-1.5 rounded-full bg-teal-600 flex-shrink-0"></span>
				{/if}
			</div>
			<span class="mt-2 text-2xl sm:text-3xl font-bold font-mono tracking-tight">
				{rekeningBelumProduktif.length}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">rekening belum ada barang terhubung</p>
		</div>

		<!-- ///////////////////////////////////////////////////////////// -->
		<!-- ROW B — pie distribusi + tren pendapatan -->
		<!-- ///////////////////////////////////////////////////////////// -->

		<!-- 5. DISTRIBUSI PENDAPATAN PER REKENING (PIE) -->
		<div class="lg:col-span-5 border border-zinc-800/20 rounded-sm p-4 sm:p-5">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				DISTRIBUSI PENDAPATAN PER REKENING
			</span>

			<div class="mt-4 flex flex-col items-center gap-4">
				<div class="relative w-36 h-36 sm:w-40 sm:h-40 flex-shrink-0">
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
						<span class="text-[8px] text-zinc-400 uppercase tracking-wider">Total</span>
						<span class="text-xs font-bold font-mono text-zinc-800">{formatRupiah(totalOmsetSemuaRekening)}</span>
					</div>
				</div>

				<div class="w-full flex flex-col gap-1.5">
					{#each segmenDonut() as seg}
						<div class="flex items-center justify-between gap-2 text-[10px]">
							<span class="flex items-center gap-1.5 min-w-0">
								<span class="w-2 h-2 rounded-full flex-shrink-0" style:background-color={seg.warna}></span>
								<span class="truncate text-zinc-600">{seg.label}</span>
							</span>
							<span class="font-mono font-semibold text-zinc-800 flex-shrink-0">{seg.persen.toFixed(1)}%</span>
						</div>
					{/each}
				</div>
			</div>
		</div>

		<!-- 6. TREN PENDAPATAN -->
		<div class="lg:col-span-7 border border-zinc-800/20 rounded-sm p-4 sm:p-5">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				TREN PENDAPATAN
			</span>

			<div class="mt-3 w-full">
				<svg viewBox="0 0 {CHART_W} {CHART_H}" class="w-full h-[10rem] sm:h-[12rem]" preserveAspectRatio="none">
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

					<path d={areaPath(pendapatanPoints, CHART_H, PAD_X)} class="fill-teal-600/10" />
					<path d={linePath(pendapatanPoints)} fill="none" class="stroke-teal-600" stroke-width="2" />

					{#each pendapatanPoints as p}
						<circle cx={p.x} cy={p.y} r="2.5" class="fill-teal-600" />
					{/each}
				</svg>

				<div class="relative h-4 mt-1">
					{#each visibleLabelIdx as i}
						<span
							class="absolute -translate-x-1/2 text-[8px] sm:text-[9px] font-mono text-slate-400 uppercase tracking-wider whitespace-nowrap"
							style:left="{(pendapatanPoints[i].x / CHART_W) * 100}%"
						>
							{data[i].label}
						</span>
					{/each}
				</div>
			</div>
		</div>

		<!-- ///////////////////////////////////////////////////////////// -->
		<!-- ROW C — 7. RINCIAN REKENING PRODUKTIF -->
		<!-- ///////////////////////////////////////////////////////////// -->
		<div class="lg:col-span-12 border border-zinc-800/20 rounded-sm p-4 sm:p-5">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				RINCIAN REKENING PRODUKTIF
			</span>
			<p class="text-[10px] text-zinc-400 mt-1">
				Rekening ditandai "Produktif" kalau sudah ada barang terhubung dan pernah menerima transaksi.
			</p>

			<div class="mt-4 flex flex-col gap-3">
				{#each rincianTerurut as r, i (r.idRekeningSeller)}
					{@const produktif = r.jumlahBarangTerhubung > 0 && r.omsetSepanjangWaktu > 0}
					<div class="border border-zinc-200 rounded-md p-3 flex flex-col sm:flex-row sm:items-center gap-3">
						<div class="flex items-center gap-2 sm:w-64 flex-shrink-0">
							<span class="text-[10px] font-mono text-zinc-400">0{i + 1}</span>
							<div class="min-w-0">
								<p class="text-[11px] font-semibold text-zinc-800 truncate flex items-center gap-1.5">
									{r.namaBank}
									{#if r.isDefault}
										<span class="px-1 py-0.5 rounded text-[8px] font-medium uppercase tracking-wider bg-teal-50 text-teal-700">Default</span>
									{/if}
								</p>
								<p class="text-[9px] font-mono text-zinc-400">{maskNorek(r.nomorRekening)}</p>
							</div>
						</div>

						<div class="flex-1 grid grid-cols-2 sm:grid-cols-4 gap-3 text-[10px]">
							<div>
								<p class="text-zinc-400 uppercase tracking-wider text-[9px]">Omset</p>
								<p class="font-mono font-bold text-zinc-800 mt-0.5">{formatRupiahPenuh(r.omsetSepanjangWaktu)}</p>
							</div>
							<div>
								<p class="text-zinc-400 uppercase tracking-wider text-[9px]">Andil</p>
								<p class="font-mono font-bold text-teal-700 mt-0.5">{andilPersen(r).toFixed(1)}%</p>
							</div>
							<div>
								<p class="text-zinc-400 uppercase tracking-wider text-[9px]">Barang Terhubung</p>
								<p class="font-mono font-bold text-zinc-800 mt-0.5">{r.jumlahBarangTerhubung} varian</p>
							</div>
							<div>
								<p class="text-zinc-400 uppercase tracking-wider text-[9px]">Rata-rata / Transaksi</p>
								<p class="font-mono font-bold text-zinc-800 mt-0.5">{formatRupiah(rataRataPerTransaksi(r))}</p>
							</div>
						</div>

						<span
							class="px-2 py-1 rounded text-[9px] font-medium uppercase tracking-wider flex-shrink-0 self-start sm:self-center {produktif
								? 'bg-teal-50 text-teal-700'
								: 'bg-zinc-100 text-zinc-500'}"
						>
							{produktif ? 'Produktif' : 'Belum Produktif'}
						</span>
					</div>
				{/each}
			</div>
		</div>

	</div>
</section>