<script lang="ts">
	// ///////////////////////////////////////////////////////////////////////
	// Types
	// ///////////////////////////////////////////////////////////////////////

	type RangeMode = 'hari' | 'minggu' | 'bulan' | 'tahun' | 'custom';

	interface BucketData {
		label: string;
		ditambahkan: number; // count AlamatGudang dibuat pada bucket ini (by created_at)
	}

	interface GudangItem {
		id_alamat_gudang: number;
		panggilan_alamat_gudang: string; // label bebas dari seller, mis. "Gudang Pusat Bandung"
		nama_alamat_gudang: string;
		provinsi_alamat_gudang: string;
		kota_alamat_gudang: string;
		created_at: string;
		jumlah_barang: number; // hasil join COUNT(Barang) WHERE id_alamat_gudang = ini
		jumlah_transaksi: number; // hasil join COUNT(Transaksi) yang barangnya berasal dari gudang ini
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
		{ value: 'hari', label: 'Per Hari' },
		{ value: 'minggu', label: 'Per Minggu' },
		{ value: 'bulan', label: 'Per Bulan' },
		{ value: 'tahun', label: 'Per Tahun' },
		{ value: 'custom', label: 'Custom' }
	];

	// ///////////////////////////////////////////////////////////////////////
	// Mock data generator tren — ganti dengan fetch ke gudang_services yang
	// query created_at AlamatGudang
	// ///////////////////////////////////////////////////////////////////////

	function seedNumber(seed: number, min: number, max: number): number {
		const x = Math.sin(seed * 999.77) * 10000;
		const frac = x - Math.floor(x);
		return Math.round(min + frac * (max - min));
	}

	function bucketsForMode(mode: RangeMode, start: string, end: string): BucketData[] {
		if (mode === 'hari') {
			const jam = ['00:00', '03:00', '06:00', '09:00', '12:00', '15:00', '18:00', '21:00'];
			return jam.map((j, i) => ({ label: j, ditambahkan: seedNumber(i + 1, 0, 1) }));
		}

		if (mode === 'minggu') {
			const hari = ['Sen', 'Sel', 'Rab', 'Kam', 'Jum', 'Sab', 'Min'];
			return hari.map((h, i) => ({ label: h, ditambahkan: seedNumber(i + 40, 0, 2) }));
		}

		if (mode === 'bulan') {
			return Array.from({ length: 4 }, (_, i) => ({
				label: `Minggu ${i + 1}`,
				ditambahkan: seedNumber(i + 80, 0, 3)
			}));
		}

		if (mode === 'tahun') {
			const bulan = ['Jan', 'Feb', 'Mar', 'Apr', 'Mei', 'Jun', 'Jul', 'Agu', 'Sep', 'Okt', 'Nov', 'Des'];
			return bulan.map((b, i) => ({ label: b, ditambahkan: seedNumber(i + 120, 0, 4) }));
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
				ditambahkan: seedNumber(i + 160, 0, 2)
			};
		});
	}

	// ///////////////////////////////////////////////////////////////////////
	// Hardcode master gudang — ganti dengan fetch list AlamatGudang milik seller
	// (jumlah_barang & jumlah_transaksi hasil join, lihat catatan di bawah)
	// ///////////////////////////////////////////////////////////////////////

	const gudangList: GudangItem[] = [
		{
			id_alamat_gudang: 1,
			panggilan_alamat_gudang: 'Gudang Pusat',
			nama_alamat_gudang: 'Jl. Industri Raya No. 12',
			provinsi_alamat_gudang: 'Jawa Barat',
			kota_alamat_gudang: 'Bandung',
			created_at: '2025-11-02T08:00:00Z',
			jumlah_barang: 482,
			jumlah_transaksi: 1204
		},
		{
			id_alamat_gudang: 2,
			panggilan_alamat_gudang: 'Gudang Cabang Timur',
			nama_alamat_gudang: 'Jl. Rungkut Industri No. 5',
			provinsi_alamat_gudang: 'Jawa Timur',
			kota_alamat_gudang: 'Surabaya',
			created_at: '2026-02-14T10:00:00Z',
			jumlah_barang: 316,
			jumlah_transaksi: 587
		},
		{
			id_alamat_gudang: 3,
			panggilan_alamat_gudang: 'Gudang Titipan Reseller',
			nama_alamat_gudang: 'Jl. Kaliurang KM 9',
			provinsi_alamat_gudang: 'DI Yogyakarta',
			kota_alamat_gudang: 'Sleman',
			created_at: '2026-05-20T09:30:00Z',
			jumlah_barang: 94,
			jumlah_transaksi: 0
		},
		{
			id_alamat_gudang: 4,
			panggilan_alamat_gudang: 'Gudang Musiman Lebaran',
			nama_alamat_gudang: 'Jl. Soekarno Hatta No. 88',
			provinsi_alamat_gudang: 'Jawa Barat',
			kota_alamat_gudang: 'Bekasi',
			created_at: '2026-08-01T07:00:00Z',
			jumlah_barang: 58,
			jumlah_transaksi: 41
		},
		{
			id_alamat_gudang: 5,
			panggilan_alamat_gudang: 'Gudang Gudang Lama',
			nama_alamat_gudang: 'Jl. Pahlawan No. 3',
			provinsi_alamat_gudang: 'Jawa Barat',
			kota_alamat_gudang: 'Bandung',
			created_at: '2025-06-10T11:00:00Z',
			jumlah_barang: 21,
			jumlah_transaksi: 0
		}
	];

	// ///////////////////////////////////////////////////////////////////////
	// Derived — angka ringkasan
	// ///////////////////////////////////////////////////////////////////////

	let data = $derived(bucketsForMode(rangeMode, customStart, customEnd));
	let totalDitambahkanPeriode = $derived(data.reduce((a, d) => a + d.ditambahkan, 0));
	let deltaDitambahkan = $derived(seedNumber(totalDitambahkanPeriode % 41, -30, 50));

	let totalGudang = $derived(gudangList.length);

	let rataRataBarang = $derived(
		totalGudang > 0
			? Math.round((gudangList.reduce((a, g) => a + g.jumlah_barang, 0) / totalGudang) * 10) / 10
			: 0
	);

	let provinsiTerbanyak = $derived.by(() => {
		const map = new Map<string, number>();
		for (const g of gudangList) map.set(g.provinsi_alamat_gudang, (map.get(g.provinsi_alamat_gudang) ?? 0) + 1);
		const sorted = [...map.entries()].sort((a, b) => b[1] - a[1]);
		return sorted[0] ?? ['-', 0];
	});

	let gudangTanpaTransaksi = $derived(gudangList.filter((g) => g.jumlah_transaksi === 0).length);

	let topGudangTransaksi = $derived([...gudangList].sort((a, b) => b.jumlah_transaksi - a.jumlah_transaksi).slice(0, 5));
	let topGudangBarang = $derived([...gudangList].sort((a, b) => b.jumlah_barang - a.jumlah_barang).slice(0, 5));

	function formatTanggal(iso: string): string {
		return new Date(iso).toLocaleDateString('id-ID', { day: '2-digit', month: 'short', year: 'numeric' });
	}

	// ///////////////////////////////////////////////////////////////////////
	// SVG chart helpers — line tren penambahan gudang
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

	let ditambahkanPoints = $derived(toPoints(data.map((d) => d.ditambahkan), CHART_W, CHART_H, PAD_X));
	let visibleLabelIdx = $derived(
		data.map((_, i) => i).filter((i) => data.length <= 8 || i % Math.ceil(data.length / 8) === 0)
	);
</script>

<section id="overview-gudang" class="w-full bg-white p-4 sm:p-6 lg:p-8 text-slate-950">
	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- HEADER + FILTER PERIODE -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="flex flex-col lg:flex-row lg:items-end lg:justify-between gap-4 pb-6 mb-6 border-b border-zinc-800/10">
		<div>
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
				ANALITIK GUDANG
			</span>
			<h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
				Ringkasan Alamat Gudang
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

		<!-- 1. TOTAL ALAMAT GUDANG -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				TOTAL ALAMAT GUDANG
			</span>
			<span class="mt-2 text-2xl sm:text-3xl font-bold font-mono tracking-tight">
				{totalGudang}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">lokasi gudang aktif kamu kelola</p>
		</div>

		<!-- 2. RATA-RATA BARANG PER GUDANG -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				RATA-RATA BARANG / GUDANG
			</span>
			<span class="mt-2 text-2xl sm:text-3xl font-bold font-mono tracking-tight text-teal-700">
				{rataRataBarang}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">sebaran stok antar gudang</p>
		</div>

		<!-- 3. PROVINSI TERBANYAK -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				PROVINSI TERBANYAK
			</span>
			<span class="mt-2 text-lg sm:text-xl font-bold tracking-tight truncate">
				{provinsiTerbanyak[0]}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">{provinsiTerbanyak[1]} gudang di provinsi ini</p>
		</div>

		<!-- 4. GUDANG TANPA TRANSAKSI — actionable -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<div class="flex items-center justify-between gap-2">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					GUDANG TANPA TRANSAKSI
				</span>
				{#if gudangTanpaTransaksi > 0}
					<span class="w-1.5 h-1.5 rounded-full bg-teal-600 flex-shrink-0"></span>
				{/if}
			</div>
			<span class="mt-2 text-2xl sm:text-3xl font-bold font-mono tracking-tight">
				{gudangTanpaTransaksi}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">belum pernah jadi asal pengiriman</p>
		</div>

		<!-- ///////////////////////////////////////////////////////////// -->
		<!-- ROW B — chart tren penambahan gudang + gudang baru periode ini -->
		<!-- ///////////////////////////////////////////////////////////// -->

		<!-- 5. TREN PENAMBAHAN GUDANG -->
		<div class="lg:col-span-7 border border-zinc-800/20 rounded-sm p-4 sm:p-5">
			<div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-2">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					TREN PENAMBAHAN GUDANG
				</span>
				<span class="text-[10px] sm:text-[11px] font-mono font-bold {deltaDitambahkan >= 0 ? 'text-teal-600' : 'text-rose-600'}">
					{deltaDitambahkan >= 0 ? '+' : ''}{deltaDitambahkan}% vs periode lalu
				</span>
			</div>

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

					<path d={areaPath(ditambahkanPoints, CHART_H, PAD_X)} class="fill-teal-600/10" />
					<path d={linePath(ditambahkanPoints)} fill="none" class="stroke-teal-600" stroke-width="2" />

					{#each ditambahkanPoints as p}
						<circle cx={p.x} cy={p.y} r="2.5" class="fill-teal-600" />
					{/each}
				</svg>

				<div class="relative h-4 mt-1">
					{#each visibleLabelIdx as i}
						<span
							class="absolute -translate-x-1/2 text-[8px] sm:text-[9px] font-mono text-slate-400 uppercase tracking-wider whitespace-nowrap"
							style:left="{(ditambahkanPoints[i].x / CHART_W) * 100}%"
						>
							{data[i].label}
						</span>
					{/each}
				</div>
			</div>
		</div>

		<!-- 6. GUDANG BARU DITAMBAHKAN (PERIODE INI) -->
		<div class="lg:col-span-5 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				GUDANG BARU (PERIODE INI)
			</span>
			<span class="mt-2 text-2xl sm:text-3xl font-bold font-mono tracking-tight text-teal-700">
				{totalDitambahkanPeriode}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">
				sesuai filter periode di atas — bandingkan dengan tren untuk lihat lonjakan ekspansi gudang
			</p>
		</div>

		<!-- ///////////////////////////////////////////////////////////// -->
		<!-- ROW C — ranking gudang transaksi & gudang barang terbanyak -->
		<!-- ///////////////////////////////////////////////////////////// -->

		<!-- 7. TOP GUDANG PALING SERING JADI WADAH TRANSAKSI -->
		<div class="lg:col-span-6 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				TOP GUDANG PALING SERING TRANSAKSI
			</span>

			<div class="mt-4 space-y-3 flex-1 flex flex-col justify-center">
				{#each topGudangTransaksi as g, i}
					{@const maxTransaksi = Math.max(topGudangTransaksi[0].jumlah_transaksi, 1)}
					<div>
						<div class="flex items-center justify-between mb-1">
							<span class="text-[10px] sm:text-[11px] font-medium truncate flex items-center gap-1.5">
								<span class="text-teal-700 font-mono">0{i + 1}</span>
								{g.panggilan_alamat_gudang}
								<span class="text-slate-400 font-mono text-[9px]">— {g.kota_alamat_gudang}</span>
							</span>
							<span class="text-[10px] sm:text-[11px] font-bold font-mono flex-shrink-0 ml-2">
								{g.jumlah_transaksi}x
							</span>
						</div>
						<div class="w-full h-1 bg-slate-100 rounded-full overflow-hidden">
							<div class="h-full bg-teal-600 rounded-full" style:width="{(g.jumlah_transaksi / maxTransaksi) * 100}%"></div>
						</div>
					</div>
				{/each}
			</div>
		</div>

		<!-- 8. GUDANG DENGAN BARANG TERBANYAK -->
		<div class="lg:col-span-6 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				GUDANG DENGAN BARANG TERBANYAK
			</span>

			<div class="mt-4 space-y-3 flex-1 flex flex-col justify-center">
				{#each topGudangBarang as g, i}
					{@const maxBarang = Math.max(topGudangBarang[0].jumlah_barang, 1)}
					<div>
						<div class="flex items-center justify-between mb-1">
							<span class="text-[10px] sm:text-[11px] font-medium truncate flex items-center gap-1.5">
								<span class="text-teal-700 font-mono">0{i + 1}</span>
								{g.panggilan_alamat_gudang}
								<span class="text-slate-400 font-mono text-[9px]">— {g.kota_alamat_gudang}</span>
							</span>
							<span class="text-[10px] sm:text-[11px] font-bold font-mono flex-shrink-0 ml-2">
								{g.jumlah_barang}
							</span>
						</div>
						<div class="w-full h-1 bg-slate-100 rounded-full overflow-hidden">
							<div class="h-full bg-teal-600 rounded-full" style:width="{(g.jumlah_barang / maxBarang) * 100}%"></div>
						</div>
					</div>
				{/each}
			</div>
		</div>

	</div>
</section>