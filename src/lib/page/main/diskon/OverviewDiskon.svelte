<script lang="ts">
	// ///////////////////////////////////////////////////////////////////////
	// Types
	// ///////////////////////////////////////////////////////////////////////

	type RangeMode = 'hari' | 'minggu' | 'bulan' | 'tahun' | 'custom';

	interface BucketData {
		label: string;
		diterbitkan: number; // count DiskonProduk dibuat pada bucket ini (by created_at)
		dipakai: number; // count Transaksi dengan id_diskon terisi pada bucket ini
	}

	interface DiskonItem {
		id_diskon_produk: number;
		nama_diskon_produk: string;
		diskon_persen_diskon_produk: number;
		berlaku_mulai_diskon_produk: string;
		berlaku_sampai_diskon_produk: string;
		status_diskon_produk: string; // Draft | Aktif | Nonaktif | Berakhir
		dipakai: number; // count Transaksi.id_diskon = ini (hasil join)
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
	// Mock data generator tren — ganti dengan fetch ke diskon_services yang
	// query created_at (diterbitkan) & join transaksi.id_diskon (dipakai)
	// ///////////////////////////////////////////////////////////////////////

	function seedNumber(seed: number, min: number, max: number): number {
		const x = Math.sin(seed * 999.77) * 10000;
		const frac = x - Math.floor(x);
		return Math.round(min + frac * (max - min));
	}

	function bucketsForMode(mode: RangeMode, start: string, end: string): BucketData[] {
		if (mode === 'hari') {
			const jam = ['00:00', '03:00', '06:00', '09:00', '12:00', '15:00', '18:00', '21:00'];
			return jam.map((j, i) => ({
				label: j,
				diterbitkan: seedNumber(i + 1, 0, 2),
				dipakai: seedNumber(i + 20, 1, 12)
			}));
		}

		if (mode === 'minggu') {
			const hari = ['Sen', 'Sel', 'Rab', 'Kam', 'Jum', 'Sab', 'Min'];
			return hari.map((h, i) => ({
				label: h,
				diterbitkan: seedNumber(i + 40, 0, 3),
				dipakai: seedNumber(i + 60, 8, 46)
			}));
		}

		if (mode === 'bulan') {
			return Array.from({ length: 4 }, (_, i) => ({
				label: `Minggu ${i + 1}`,
				diterbitkan: seedNumber(i + 80, 1, 6),
				dipakai: seedNumber(i + 100, 40, 180)
			}));
		}

		if (mode === 'tahun') {
			const bulan = ['Jan', 'Feb', 'Mar', 'Apr', 'Mei', 'Jun', 'Jul', 'Agu', 'Sep', 'Okt', 'Nov', 'Des'];
			return bulan.map((b, i) => ({
				label: b,
				diterbitkan: seedNumber(i + 120, 2, 14),
				dipakai: seedNumber(i + 140, 180, 720)
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
				diterbitkan: seedNumber(i + 160, 0, 3),
				dipakai: seedNumber(i + 180, 5, 34)
			};
		});
	}

	// ///////////////////////////////////////////////////////////////////////
	// Hardcode master diskon — ganti dengan fetch list diskon milik seller
	// (dipakai dihitung dari COUNT(Transaksi) WHERE id_diskon = id ini)
	// ///////////////////////////////////////////////////////////////////////

	const diskonList: DiskonItem[] = [
		{
			id_diskon_produk: 1,
			nama_diskon_produk: 'Gajian Sale 20%',
			diskon_persen_diskon_produk: 20,
			berlaku_mulai_diskon_produk: '2026-08-20',
			berlaku_sampai_diskon_produk: '2026-08-31',
			status_diskon_produk: 'Aktif',
			dipakai: 214
		},
		{
			id_diskon_produk: 2,
			nama_diskon_produk: 'Flash Sale Akhir Pekan',
			diskon_persen_diskon_produk: 35,
			berlaku_mulai_diskon_produk: '2026-08-22',
			berlaku_sampai_diskon_produk: '2026-08-24',
			status_diskon_produk: 'Berakhir',
			dipakai: 156
		},
		{
			id_diskon_produk: 3,
			nama_diskon_produk: 'Diskon Member Baru',
			diskon_persen_diskon_produk: 10,
			berlaku_mulai_diskon_produk: '2026-07-01',
			berlaku_sampai_diskon_produk: '2026-12-31',
			status_diskon_produk: 'Aktif',
			dipakai: 98
		},
		{
			id_diskon_produk: 4,
			nama_diskon_produk: '17-an Merdeka Sale',
			diskon_persen_diskon_produk: 17,
			berlaku_mulai_diskon_produk: '2026-08-17',
			berlaku_sampai_diskon_produk: '2026-08-19',
			status_diskon_produk: 'Berakhir',
			dipakai: 87
		},
		{
			id_diskon_produk: 5,
			nama_diskon_produk: 'Bundling September',
			diskon_persen_diskon_produk: 15,
			berlaku_mulai_diskon_produk: '2026-09-01',
			berlaku_sampai_diskon_produk: '2026-09-15',
			status_diskon_produk: 'Draft',
			dipakai: 0
		},
		{
			id_diskon_produk: 6,
			nama_diskon_produk: 'Clearance Preloved',
			diskon_persen_diskon_produk: 25,
			berlaku_mulai_diskon_produk: '2026-08-10',
			berlaku_sampai_diskon_produk: '2026-08-31',
			status_diskon_produk: 'Nonaktif',
			dipakai: 12
		}
	];

	// ///////////////////////////////////////////////////////////////////////
	// Derived — angka ringkasan
	// ///////////////////////////////////////////////////////////////////////

	let data = $derived(bucketsForMode(rangeMode, customStart, customEnd));

	let totalDiterbitkanPeriode = $derived(data.reduce((a, d) => a + d.diterbitkan, 0));
	let totalDipakaiPeriode = $derived(data.reduce((a, d) => a + d.dipakai, 0));

	let totalDiskon = $derived(diskonList.length);
	let diskonAktif = $derived(diskonList.filter((d) => d.status_diskon_produk === 'Aktif').length);
	let diskonDraft = $derived(diskonList.filter((d) => d.status_diskon_produk === 'Draft').length);
	let diskonBerakhir = $derived(diskonList.filter((d) => d.status_diskon_produk === 'Berakhir').length);

	let rataRataPersen = $derived(
		totalDiskon > 0
			? Math.round((diskonList.reduce((a, d) => a + d.diskon_persen_diskon_produk, 0) / totalDiskon) * 10) / 10
			: 0
	);

	let totalDipakaiKeseluruhan = $derived(diskonList.reduce((a, d) => a + d.dipakai, 0));

	const statusUrutan = ['Draft', 'Aktif', 'Nonaktif', 'Berakhir'];
	let statusBreakdown = $derived(
		statusUrutan.map((status) => ({
			status,
			jumlah: diskonList.filter((d) => d.status_diskon_produk === status).length
		}))
	);

	let diskonTerpakai = $derived([...diskonList].sort((a, b) => b.dipakai - a.dipakai).slice(0, 5));

	// delta dummy vs periode sebelumnya
	let deltaDiterbitkan = $derived(seedNumber(totalDiterbitkanPeriode % 41, -20, 40));
	let deltaDipakai = $derived(seedNumber(totalDipakaiPeriode % 89, -10, 26));

	function formatTanggal(iso: string): string {
		return new Date(iso).toLocaleDateString('id-ID', { day: '2-digit', month: 'short', year: 'numeric' });
	}

	function statusClass(status: string): string {
		if (status === 'Aktif') return 'bg-teal-50 text-teal-700';
		if (status === 'Draft') return 'bg-zinc-100 text-zinc-500';
		if (status === 'Berakhir') return 'bg-rose-50 text-rose-600';
		return 'bg-zinc-100 text-zinc-400';
	}

	// ///////////////////////////////////////////////////////////////////////
	// SVG chart helpers — dual-line diterbitkan vs dipakai
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

	let dipakaiPoints = $derived(toPoints(data.map((d) => d.dipakai), CHART_W, CHART_H, PAD_X));
	let visibleLabelIdx = $derived(
		data.map((_, i) => i).filter((i) => data.length <= 8 || i % Math.ceil(data.length / 8) === 0)
	);

	let maxDiterbitkan = $derived(Math.max(...data.map((d) => d.diterbitkan), 1));
</script>

<section id="overview-diskon" class="w-full bg-white p-4 sm:p-6 lg:p-8 text-slate-950">
	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- HEADER + FILTER PERIODE -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="flex flex-col lg:flex-row lg:items-end lg:justify-between gap-4 pb-6 mb-6 border-b border-zinc-800/10">
		<div>
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
				ANALITIK DISKON
			</span>
			<h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
				Ringkasan Diskon Toko
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
		<!-- ROW A — 4 KPI status diskon -->
		<!-- ///////////////////////////////////////////////////////////// -->

		<!-- 1. TOTAL DISKON DITERBITKAN -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				TOTAL DISKON DITERBITKAN
			</span>
			<span class="mt-2 text-2xl sm:text-3xl font-bold font-mono tracking-tight">
				{totalDiskon}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">sepanjang waktu, semua status</p>
		</div>

		<!-- 2. DISKON AKTIF -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				DISKON AKTIF
			</span>
			<span class="mt-2 text-2xl sm:text-3xl font-bold font-mono tracking-tight text-teal-700">
				{diskonAktif}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">sedang berjalan saat ini</p>
		</div>

		<!-- 3. DISKON DRAFT — actionable -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<div class="flex items-center justify-between gap-2">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					DISKON DRAFT
				</span>
				{#if diskonDraft > 0}
					<span class="w-1.5 h-1.5 rounded-full bg-teal-600 flex-shrink-0"></span>
				{/if}
			</div>
			<span class="mt-2 text-2xl sm:text-3xl font-bold font-mono tracking-tight">
				{diskonDraft}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">belum dipublish ke pembeli</p>
		</div>

		<!-- 4. DISKON BERAKHIR — actionable -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<div class="flex items-center justify-between gap-2">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					DISKON BERAKHIR
				</span>
				{#if diskonBerakhir > 0}
					<span class="w-1.5 h-1.5 rounded-full bg-teal-600 flex-shrink-0"></span>
				{/if}
			</div>
			<span class="mt-2 text-2xl sm:text-3xl font-bold font-mono tracking-tight">
				{diskonBerakhir}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">masa berlaku sudah lewat</p>
		</div>

		<!-- ///////////////////////////////////////////////////////////// -->
		<!-- ROW B — chart tren pemakaian + rata-rata persen & total dipakai -->
		<!-- ///////////////////////////////////////////////////////////// -->

		<!-- TREN PEMAKAIAN DISKON -->
		<div class="lg:col-span-7 border border-zinc-800/20 rounded-sm p-4 sm:p-5">
			<div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-2">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					TREN PEMAKAIAN DISKON
				</span>
				<span class="text-[10px] sm:text-[11px] font-mono font-bold {deltaDipakai >= 0 ? 'text-teal-600' : 'text-rose-600'}">
					{deltaDipakai >= 0 ? '+' : ''}{deltaDipakai}% vs periode lalu
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

					<path d={areaPath(dipakaiPoints, CHART_H, PAD_X)} class="fill-teal-600/10" />
					<path d={linePath(dipakaiPoints)} fill="none" class="stroke-teal-600" stroke-width="2" />

					{#each dipakaiPoints as p}
						<circle cx={p.x} cy={p.y} r="2.5" class="fill-teal-600" />
					{/each}
				</svg>

				<div class="relative h-4 mt-1">
					{#each visibleLabelIdx as i}
						<span
							class="absolute -translate-x-1/2 text-[8px] sm:text-[9px] font-mono text-slate-400 uppercase tracking-wider whitespace-nowrap"
							style:left="{(dipakaiPoints[i].x / CHART_W) * 100}%"
						>
							{data[i].label}
						</span>
					{/each}
				</div>
			</div>
		</div>

		<!-- RATA-RATA PERSEN & TOTAL DIPAKAI — stacked compact cards -->
		<div class="lg:col-span-5 flex flex-col gap-4">
			<!-- 5. RATA-RATA PERSENTASE DISKON -->
			<div class="flex-1 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					RATA-RATA PERSENTASE DISKON
				</span>
				<span class="mt-2 text-xl sm:text-2xl font-bold font-mono tracking-tight">
					{rataRataPersen}%
				</span>
				<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-2">dari semua diskon yang diterbitkan</p>
			</div>

			<!-- 6. TOTAL DIPAKAI DALAM TRANSAKSI -->
			<div class="flex-1 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					TOTAL DIPAKAI DALAM TRANSAKSI
				</span>
				<span class="mt-2 text-xl sm:text-2xl font-bold font-mono tracking-tight text-teal-700">
					{totalDipakaiKeseluruhan.toLocaleString('id-ID')}
				</span>
				<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-2">gabungan semua diskon, sepanjang waktu</p>
			</div>
		</div>

		<!-- ///////////////////////////////////////////////////////////// -->
		<!-- ROW C — distribusi status & diskon paling sering dipakai -->
		<!-- ///////////////////////////////////////////////////////////// -->

		<!-- 7. DISTRIBUSI STATUS DISKON -->
		<div class="lg:col-span-6 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				DISTRIBUSI STATUS DISKON
			</span>

			<div class="mt-4 space-y-3 flex-1 flex flex-col justify-center">
				{#each statusBreakdown as s}
					{@const maxJumlah = Math.max(...statusBreakdown.map((x) => x.jumlah), 1)}
					<div>
						<div class="flex items-center justify-between mb-1">
							<span class="text-[10px] sm:text-[11px] font-medium truncate flex items-center gap-1.5">
								<span class="px-1 py-0.5 rounded text-[8px] uppercase tracking-wider {statusClass(s.status)}">
									{s.status}
								</span>
							</span>
							<span class="text-[10px] sm:text-[11px] font-bold font-mono flex-shrink-0 ml-2">
								{s.jumlah}
							</span>
						</div>
						<div class="w-full h-1 bg-slate-100 rounded-full overflow-hidden">
							<div
								class="h-full rounded-full {s.status === 'Aktif' ? 'bg-teal-600' : 'bg-slate-950'}"
								style:width="{(s.jumlah / maxJumlah) * 100}%"
							></div>
						</div>
					</div>
				{/each}
			</div>
		</div>

		<!-- 8. DISKON PALING SERING DIPAKAI -->
		<div class="lg:col-span-6 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				DISKON PALING SERING DIPAKAI
			</span>

			<div class="mt-4 space-y-3 flex-1 flex flex-col justify-center">
				{#each diskonTerpakai as d, i}
					{@const maxDipakai = Math.max(diskonTerpakai[0].dipakai, 1)}
					<div>
						<div class="flex items-center justify-between mb-1">
							<span class="text-[10px] sm:text-[11px] font-medium truncate flex items-center gap-1.5">
								<span class="text-teal-700 font-mono">0{i + 1}</span>
								{d.nama_diskon_produk}
								<span class="text-slate-400 font-mono text-[9px]">-{d.diskon_persen_diskon_produk}%</span>
							</span>
							<span class="text-[10px] sm:text-[11px] font-bold font-mono flex-shrink-0 ml-2">
								{d.dipakai}x
							</span>
						</div>
						<div class="w-full h-1 bg-slate-100 rounded-full overflow-hidden">
							<div class="h-full bg-teal-600 rounded-full" style:width="{(d.dipakai / maxDipakai) * 100}%"></div>
						</div>
					</div>
				{/each}
			</div>
		</div>

	</div>
</section>