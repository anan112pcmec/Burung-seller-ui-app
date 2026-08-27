<script lang="ts">
	// ///////////////////////////////////////////////////////////////////////
	// Types
	// ///////////////////////////////////////////////////////////////////////

	type RangeMode = 'hari' | 'minggu' | 'bulan' | 'tahun' | 'custom';

	interface BucketData {
		label: string;
		totalTransaksi: number; // count Transaksi
		totalNilaiKotor: number; // sum Transaksi.Total (termasuk harga barang)
		pendapatanBersih: number; // sum Transaksi.SellerPaid
		barangTerjual: number; // sum Transaksi.KuantitasBarang
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
	// Mock data generator — ganti dengan fetch ke transaksi_services yang
	// query berdasarkan rangeMode & rentang tanggal terpilih
	// ///////////////////////////////////////////////////////////////////////

	function seedNumber(seed: number, min: number, max: number): number {
		const x = Math.sin(seed * 999.77) * 10000;
		const frac = x - Math.floor(x);
		return Math.round(min + frac * (max - min));
	}

	function bucketsForMode(mode: RangeMode, start: string, end: string): BucketData[] {
		if (mode === 'hari') {
			const jam = ['00:00', '03:00', '06:00', '09:00', '12:00', '15:00', '18:00', '21:00'];
			return jam.map((j, i) => {
				const kotor = seedNumber(i + 1, 600_000, 3_800_000);
				return {
					label: j,
					totalTransaksi: seedNumber(i + 10, 2, 18),
					totalNilaiKotor: kotor,
					pendapatanBersih: Math.round(kotor * 0.78),
					barangTerjual: seedNumber(i + 30, 2, 24)
				};
			});
		}

		if (mode === 'minggu') {
			const hari = ['Sen', 'Sel', 'Rab', 'Kam', 'Jum', 'Sab', 'Min'];
			return hari.map((h, i) => {
				const kotor = seedNumber(i + 40, 4_500_000, 16_000_000);
				return {
					label: h,
					totalTransaksi: seedNumber(i + 60, 12, 60),
					totalNilaiKotor: kotor,
					pendapatanBersih: Math.round(kotor * 0.78),
					barangTerjual: seedNumber(i + 80, 20, 90)
				};
			});
		}

		if (mode === 'bulan') {
			return Array.from({ length: 4 }, (_, i) => {
				const kotor = seedNumber(i + 100, 22_000_000, 68_000_000);
				return {
					label: `Minggu ${i + 1}`,
					totalTransaksi: seedNumber(i + 120, 80, 260),
					totalNilaiKotor: kotor,
					pendapatanBersih: Math.round(kotor * 0.78),
					barangTerjual: seedNumber(i + 140, 120, 420)
				};
			});
		}

		if (mode === 'tahun') {
			const bulan = ['Jan', 'Feb', 'Mar', 'Apr', 'Mei', 'Jun', 'Jul', 'Agu', 'Sep', 'Okt', 'Nov', 'Des'];
			return bulan.map((b, i) => {
				const kotor = seedNumber(i + 160, 90_000_000, 260_000_000);
				return {
					label: b,
					totalTransaksi: seedNumber(i + 180, 320, 980),
					totalNilaiKotor: kotor,
					pendapatanBersih: Math.round(kotor * 0.78),
					barangTerjual: seedNumber(i + 200, 500, 1_800)
				};
			});
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
			const kotor = seedNumber(i + 220, 3_000_000, 14_000_000);
			return {
				label: tgl.toLocaleDateString('id-ID', { day: '2-digit', month: 'short' }),
				totalTransaksi: seedNumber(i + 240, 10, 55),
				totalNilaiKotor: kotor,
				pendapatanBersih: Math.round(kotor * 0.78),
				barangTerjual: seedNumber(i + 260, 15, 80)
			};
		});
	}

	// ///////////////////////////////////////////////////////////////////////
	// Master data breakdown — status transaksi & metode pembayaran
	// (proporsi dummy, dihitung dari total transaksi periode terpilih)
	// ///////////////////////////////////////////////////////////////////////

	const statusMaster: { status: string; proporsi: number }[] = [
		{ status: 'Dibayar', proporsi: 0.28 },
		{ status: 'Diproses', proporsi: 0.18 },
		{ status: 'Dikirim', proporsi: 0.22 },
		{ status: 'Selesai', proporsi: 0.26 },
		{ status: 'Dibatalkan', proporsi: 0.06 }
	];

	const metodePembayaranMaster: { provider: string; proporsi: number }[] = [
		{ provider: 'QRIS', proporsi: 0.38 },
		{ provider: 'Virtual Account', proporsi: 0.29 },
		{ provider: 'E-Wallet', proporsi: 0.21 },
		{ provider: 'Kartu Kredit', proporsi: 0.12 }
	];

	// ///////////////////////////////////////////////////////////////////////
	// Derived — angka ringkasan
	// ///////////////////////////////////////////////////////////////////////

	let data = $derived(bucketsForMode(rangeMode, customStart, customEnd));

	let totalTransaksi = $derived(data.reduce((a, d) => a + d.totalTransaksi, 0));
	let totalNilaiKotor = $derived(data.reduce((a, d) => a + d.totalNilaiKotor, 0));
	let pendapatanBersih = $derived(data.reduce((a, d) => a + d.pendapatanBersih, 0));
	let barangTerjual = $derived(data.reduce((a, d) => a + d.barangTerjual, 0));

	let rataRataNilaiTransaksi = $derived(totalTransaksi > 0 ? Math.round(totalNilaiKotor / totalTransaksi) : 0);

	let statusBreakdown = $derived(
		statusMaster.map((s) => ({ ...s, jumlah: Math.round(totalTransaksi * s.proporsi) }))
	);
	let transaksiDibatalkan = $derived(statusBreakdown.find((s) => s.status === 'Dibatalkan')?.jumlah ?? 0);

	let paymentBreakdown = $derived(
		[...metodePembayaranMaster]
			.map((p) => ({ ...p, jumlah: Math.round(totalTransaksi * p.proporsi) }))
			.sort((a, b) => b.jumlah - a.jumlah)
	);

	// delta dummy vs periode sebelumnya
	let deltaTransaksi = $derived(seedNumber(totalTransaksi % 97, -10, 22));
	let deltaKotor = $derived(seedNumber(totalNilaiKotor % 89, -8, 26));
	let deltaBersih = $derived(seedNumber(pendapatanBersih % 73, -8, 24));
	let deltaBarangTerjual = $derived(seedNumber(barangTerjual % 61, -12, 20));

	function formatRupiah(n: number): string {
		if (n >= 1_000_000_000) return `Rp${(n / 1_000_000_000).toFixed(1)}M`;
		if (n >= 1_000_000) return `Rp${(n / 1_000_000).toFixed(1)}jt`;
		if (n >= 1_000) return `Rp${(n / 1_000).toFixed(0)}rb`;
		return `Rp${n}`;
	}

	// ///////////////////////////////////////////////////////////////////////
	// SVG chart helpers — dual-line kotor vs bersih
	// ///////////////////////////////////////////////////////////////////////

	const CHART_W = 640;
	const CHART_H = 200;
	const PAD_X = 8;
	const PAD_Y = 16;

	function toPoints(values: number[], w: number, h: number, pad: number, sharedMax?: number): Point[] {
		const max = sharedMax ?? Math.max(...values, 1);
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

	// dua garis berbagi skala yang sama biar proporsional kelihatan (bersih < kotor)
	let sharedMax = $derived(Math.max(...data.map((d) => d.totalNilaiKotor), 1));
	let kotorPoints = $derived(toPoints(data.map((d) => d.totalNilaiKotor), CHART_W, CHART_H, PAD_X, sharedMax));
	let bersihPoints = $derived(toPoints(data.map((d) => d.pendapatanBersih), CHART_W, CHART_H, PAD_X, sharedMax));

	let visibleLabelIdx = $derived(
		data.map((_, i) => i).filter((i) => data.length <= 8 || i % Math.ceil(data.length / 8) === 0)
	);
</script>

<section id="overview-transaksi" class="w-full bg-white p-4 sm:p-6 lg:p-8 text-slate-950">
	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- HEADER + FILTER PERIODE -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="flex flex-col lg:flex-row lg:items-end lg:justify-between gap-4 pb-6 mb-6 border-b border-zinc-800/10">
		<div>
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
				ANALITIK TRANSAKSI
			</span>
			<h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
				Ringkasan Transaksi
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

		<!-- 1. TOTAL TRANSAKSI -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				TOTAL TRANSAKSI
			</span>
			<div class="mt-2 flex items-baseline gap-2 flex-wrap">
				<span class="text-xl sm:text-2xl font-bold font-mono tracking-tight">
					{totalTransaksi.toLocaleString('id-ID')}
				</span>
				<span class="text-[10px] sm:text-[11px] font-mono font-bold {deltaTransaksi >= 0 ? 'text-teal-600' : 'text-rose-600'}">
					{deltaTransaksi >= 0 ? '+' : ''}{deltaTransaksi}%
				</span>
			</div>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">vs periode sebelumnya</p>
		</div>

		<!-- 2. TOTAL NILAI TRANSAKSI (kotor, termasuk harga barang) -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				NILAI TRANSAKSI (KOTOR)
			</span>
			<div class="mt-2 flex items-baseline gap-2 flex-wrap">
				<span class="text-xl sm:text-2xl font-bold font-mono tracking-tight">
					{formatRupiah(totalNilaiKotor)}
				</span>
				<span class="text-[10px] sm:text-[11px] font-mono font-bold {deltaKotor >= 0 ? 'text-teal-600' : 'text-rose-600'}">
					{deltaKotor >= 0 ? '+' : ''}{deltaKotor}%
				</span>
			</div>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">termasuk harga barang & ongkir</p>
		</div>

		<!-- 3. PENDAPATAN BERSIH (SellerPaid) -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				PENDAPATAN BERSIH
			</span>
			<div class="mt-2 flex items-baseline gap-2 flex-wrap">
				<span class="text-xl sm:text-2xl font-bold font-mono tracking-tight text-teal-700">
					{formatRupiah(pendapatanBersih)}
				</span>
				<span class="text-[10px] sm:text-[11px] font-mono font-bold {deltaBersih >= 0 ? 'text-teal-600' : 'text-rose-600'}">
					{deltaBersih >= 0 ? '+' : ''}{deltaBersih}%
				</span>
			</div>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">setelah dipotong kurir/sistem/ekspedisi</p>
		</div>

		<!-- 4. BARANG TERJUAL -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				BARANG TERJUAL
			</span>
			<div class="mt-2 flex items-baseline gap-2 flex-wrap">
				<span class="text-xl sm:text-2xl font-bold font-mono tracking-tight">
					{barangTerjual.toLocaleString('id-ID')}
				</span>
				<span class="text-[10px] sm:text-[11px] text-slate-500 font-light">unit</span>
				<span class="text-[10px] sm:text-[11px] font-mono font-bold {deltaBarangTerjual >= 0 ? 'text-teal-600' : 'text-rose-600'}">
					{deltaBarangTerjual >= 0 ? '+' : ''}{deltaBarangTerjual}%
				</span>
			</div>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">total kuantitas dari semua transaksi</p>
		</div>

		<!-- ///////////////////////////////////////////////////////////// -->
		<!-- ROW B — chart tren kotor vs bersih + AOV & dibatalkan -->
		<!-- ///////////////////////////////////////////////////////////// -->

		<!-- TREN NILAI TRANSAKSI: KOTOR VS BERSIH -->
		<div class="lg:col-span-7 border border-zinc-800/20 rounded-sm p-4 sm:p-5">
			<div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-2">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					TREN NILAI TRANSAKSI
				</span>
				<div class="flex items-center gap-3 text-[9px] sm:text-[10px] font-medium uppercase tracking-wider">
					<span class="flex items-center gap-1.5">
						<span class="w-2 h-2 rounded-full bg-slate-950"></span>
						Kotor
					</span>
					<span class="flex items-center gap-1.5">
						<span class="w-2 h-2 rounded-full bg-teal-600"></span>
						Bersih
					</span>
				</div>
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

					<path d={areaPath(bersihPoints, CHART_H, PAD_X)} class="fill-teal-600/10" />
					<path d={linePath(kotorPoints)} fill="none" class="stroke-slate-950" stroke-width="2" />
					<path d={linePath(bersihPoints)} fill="none" class="stroke-teal-600" stroke-width="2" />
				</svg>

				<div class="relative h-4 mt-1">
					{#each visibleLabelIdx as i}
						<span
							class="absolute -translate-x-1/2 text-[8px] sm:text-[9px] font-mono text-slate-400 uppercase tracking-wider whitespace-nowrap"
							style:left="{(kotorPoints[i].x / CHART_W) * 100}%"
						>
							{data[i].label}
						</span>
					{/each}
				</div>
			</div>
		</div>

		<!-- AOV & DIBATALKAN — stacked compact cards -->
		<div class="lg:col-span-5 flex flex-col gap-4">
			<!-- 5. RATA-RATA NILAI PER TRANSAKSI (AOV) -->
			<div class="flex-1 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					RATA-RATA NILAI / TRANSAKSI
				</span>
				<span class="mt-2 text-xl sm:text-2xl font-bold font-mono tracking-tight">
					{formatRupiah(rataRataNilaiTransaksi)}
				</span>
				<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-2">nilai kotor ÷ jumlah transaksi</p>
			</div>

			<!-- 6. TRANSAKSI DIBATALKAN — actionable -->
			<div class="flex-1 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
				<div class="flex items-center justify-between gap-2">
					<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
						TRANSAKSI DIBATALKAN
					</span>
					{#if transaksiDibatalkan > 0}
						<span class="w-1.5 h-1.5 rounded-full bg-teal-600 flex-shrink-0"></span>
					{/if}
				</div>
				<span class="mt-2 text-xl sm:text-2xl font-bold font-mono tracking-tight">
					{transaksiDibatalkan.toLocaleString('id-ID')}
				</span>
				<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-2">
					{totalTransaksi > 0 ? Math.round((transaksiDibatalkan / totalTransaksi) * 100) : 0}% dari total transaksi periode ini
				</p>
			</div>
		</div>

		<!-- ///////////////////////////////////////////////////////////// -->
		<!-- ROW C — distribusi status & metode pembayaran -->
		<!-- ///////////////////////////////////////////////////////////// -->

		<!-- 7. DISTRIBUSI STATUS TRANSAKSI -->
		<div class="lg:col-span-6 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				DISTRIBUSI STATUS TRANSAKSI
			</span>

			<div class="mt-4 space-y-3 flex-1 flex flex-col justify-center">
				{#each statusBreakdown as s}
					{@const maxJumlah = Math.max(...statusBreakdown.map((x) => x.jumlah), 1)}
					<div>
						<div class="flex items-center justify-between mb-1">
							<span class="text-[10px] sm:text-[11px] font-medium truncate">
								{s.status}
							</span>
							<span class="text-[10px] sm:text-[11px] font-bold font-mono flex-shrink-0 ml-2">
								{s.jumlah.toLocaleString('id-ID')}
							</span>
						</div>
						<div class="w-full h-1 bg-slate-100 rounded-full overflow-hidden">
							<div
								class="h-full rounded-full {s.status === 'Dibatalkan' ? 'bg-slate-300' : 'bg-slate-950'}"
								style:width="{(s.jumlah / maxJumlah) * 100}%"
							></div>
						</div>
					</div>
				{/each}
			</div>
		</div>

		<!-- 8. METODE PEMBAYARAN TERPOPULER -->
		<div class="lg:col-span-6 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				METODE PEMBAYARAN TERPOPULER
			</span>

			<div class="mt-4 space-y-3 flex-1 flex flex-col justify-center">
				{#each paymentBreakdown as p, i}
					{@const maxJumlah = Math.max(paymentBreakdown[0].jumlah, 1)}
					<div>
						<div class="flex items-center justify-between mb-1">
							<span class="text-[10px] sm:text-[11px] font-medium truncate flex items-center gap-1.5">
								<span class="text-teal-700 font-mono">0{i + 1}</span>
								{p.provider}
							</span>
							<span class="text-[10px] sm:text-[11px] font-bold font-mono flex-shrink-0 ml-2">
								{p.jumlah.toLocaleString('id-ID')}
							</span>
						</div>
						<div class="w-full h-1 bg-slate-100 rounded-full overflow-hidden">
							<div class="h-full bg-teal-600 rounded-full" style:width="{(p.jumlah / maxJumlah) * 100}%"></div>
						</div>
					</div>
				{/each}
			</div>
		</div>

	</div>
</section>