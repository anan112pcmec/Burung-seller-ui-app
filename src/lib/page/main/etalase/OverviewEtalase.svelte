<script lang="ts">
	// ///////////////////////////////////////////////////////////////////////
	// Types
	// ///////////////////////////////////////////////////////////////////////

	interface EtalaseItem {
		id: number;
		nama: string;
		jumlahBarang: number; // dari sot_models.Etalase.JumlahBarang
		dilihat: number; // agregat SUM(barang.viewed) utk barang di etalase ini
		disukai: number; // agregat SUM(barang.likes) utk barang di etalase ini
	}

	interface Point {
		x: number;
		y: number;
	}

	// ///////////////////////////////////////////////////////////////////////
	// Mock data — ganti dengan fetch ke etalase_services (list etalase milik
	// seller) yang di-join / diperkaya angka viewed & likes dari barang_services
	// ///////////////////////////////////////////////////////////////////////

	function seedNumber(seed: number, min: number, max: number): number {
		const x = Math.sin(seed * 999.77) * 10000;
		const frac = x - Math.floor(x);
		return Math.round(min + frac * (max - min));
	}

	const etalaseList: EtalaseItem[] = [
		{ id: 1, nama: 'Etalase Utama', jumlahBarang: 42, dilihat: 18_420, disukai: 1_204 },
		{ id: 2, nama: 'Koleksi Baju Pria', jumlahBarang: 27, dilihat: 9_860, disukai: 612 },
		{ id: 3, nama: 'Aksesoris & Tas', jumlahBarang: 15, dilihat: 6_130, disukai: 388 },
		{ id: 4, nama: 'Promo Spesial', jumlahBarang: 8, dilihat: 12_040, disukai: 940 },
		{ id: 5, nama: 'Sepatu & Sandal', jumlahBarang: 19, dilihat: 4_260, disukai: 201 },
		{ id: 6, nama: 'Barang Preloved', jumlahBarang: 0, dilihat: 0, disukai: 0 }
	];

	// total barang milik seller secara keseluruhan (dari barang_services),
	// dipakai buat hitung barang yang belum ditempatkan di etalase manapun
	const totalBarangSeller = 132;

	// ///////////////////////////////////////////////////////////////////////
	// Derived — angka ringkasan
	// ///////////////////////////////////////////////////////////////////////

	let totalEtalase = $derived(etalaseList.length);
	let totalBarangDiEtalase = $derived(etalaseList.reduce((a, e) => a + e.jumlahBarang, 0));
	let etalaseKosong = $derived(etalaseList.filter((e) => e.jumlahBarang === 0).length);
	let barangBelumMasukEtalase = $derived(Math.max(0, totalBarangSeller - totalBarangDiEtalase));

	let totalDilihat = $derived(etalaseList.reduce((a, e) => a + e.dilihat, 0));
	let totalDisukai = $derived(etalaseList.reduce((a, e) => a + e.disukai, 0));

	// delta dummy vs periode sebelumnya, buat kasih konteks naik/turun
	let deltaDilihat = $derived(seedNumber(totalDilihat % 97, -8, 24));
	let deltaDisukai = $derived(seedNumber(totalDisukai % 61, -6, 20));

	let distribusiBarang = $derived([...etalaseList].sort((a, b) => b.jumlahBarang - a.jumlahBarang));
	let topDilihat = $derived([...etalaseList].sort((a, b) => b.dilihat - a.dilihat).slice(0, 5));

	function formatCompact(n: number): string {
		if (n >= 1_000_000) return `${(n / 1_000_000).toFixed(1)}jt`;
		if (n >= 1_000) return `${(n / 1_000).toFixed(1)}rb`;
		return `${n}`;
	}

	// ///////////////////////////////////////////////////////////////////////
	// SVG sparkline helpers
	// ///////////////////////////////////////////////////////////////////////

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

	// sparkline dummy per hari (7 hari terakhir), proporsional ke total saat ini
	let sparkDilihat = $derived(
		toPoints(
			Array.from({ length: 7 }, (_, i) => seedNumber(i + totalDilihat, totalDilihat * 0.08, totalDilihat * 0.2)),
			SPARK_W,
			SPARK_H,
			SPARK_PAD
		)
	);
	let sparkDisukai = $derived(
		toPoints(
			Array.from({ length: 7 }, (_, i) => seedNumber(i + totalDisukai, totalDisukai * 0.08, totalDisukai * 0.2)),
			SPARK_W,
			SPARK_H,
			SPARK_PAD
		)
	);
</script>

<section id="overview-etalase" class="w-full bg-white p-4 sm:p-6 lg:p-8 text-slate-950">
	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- HEADER -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="pb-6 mb-6 border-b border-zinc-800/10">
		<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
			ANALITIK ETALASE
		</span>
		<h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
			Ringkasan Etalase Kamu
		</h1>
	</div>

	<div class="grid grid-cols-1 lg:grid-cols-12 gap-4">

		<!-- ///////////////////////////////////////////////////////////// -->
		<!-- ROW A — 4 KPI angka murni -->
		<!-- ///////////////////////////////////////////////////////////// -->

		<!-- 1. TOTAL ETALASE -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				TOTAL ETALASE
			</span>
			<span class="mt-2 text-2xl sm:text-3xl font-bold font-mono tracking-tight">
				{totalEtalase}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">etalase aktif kamu kelola</p>
		</div>

		<!-- 2. TOTAL BARANG DI ETALASE -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				BARANG DI ETALASE
			</span>
			<span class="mt-2 text-2xl sm:text-3xl font-bold font-mono tracking-tight">
				{totalBarangDiEtalase.toLocaleString('id-ID')}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">tersebar di {totalEtalase} etalase</p>
		</div>

		<!-- 3. ETALASE KOSONG — actionable -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<div class="flex items-center justify-between gap-2">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					ETALASE KOSONG
				</span>
				{#if etalaseKosong > 0}
					<span class="w-1.5 h-1.5 rounded-full bg-teal-600 flex-shrink-0"></span>
				{/if}
			</div>
			<span class="mt-2 text-2xl sm:text-3xl font-bold font-mono tracking-tight">
				{etalaseKosong}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">
				{etalaseKosong > 0 ? 'belum diisi barang sama sekali' : 'semua etalase sudah terisi'}
			</p>
		</div>

		<!-- 4. BARANG BELUM MASUK ETALASE — actionable -->
		<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<div class="flex items-center justify-between gap-2">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					BELUM DIKELOMPOKKAN
				</span>
				{#if barangBelumMasukEtalase > 0}
					<span class="w-1.5 h-1.5 rounded-full bg-teal-600 flex-shrink-0"></span>
				{/if}
			</div>
			<span class="mt-2 text-2xl sm:text-3xl font-bold font-mono tracking-tight">
				{barangBelumMasukEtalase.toLocaleString('id-ID')}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">barang belum masuk etalase manapun</p>
		</div>

		<!-- ///////////////////////////////////////////////////////////// -->
		<!-- ROW B — 2 KPI besar dengan sparkline (agregat views & likes) -->
		<!-- ///////////////////////////////////////////////////////////// -->

		<!-- 5. TOTAL DILIHAT -->
		<div class="lg:col-span-6 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<div class="flex items-start justify-between gap-3">
				<div class="min-w-0">
					<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
						TOTAL DILIHAT
					</span>
					<div class="mt-2 flex items-baseline gap-2 flex-wrap">
						<span class="text-xl sm:text-2xl font-bold font-mono tracking-tight text-teal-700">
							{formatCompact(totalDilihat)}
						</span>
						<span class="text-[10px] sm:text-[11px] font-mono font-bold {deltaDilihat >= 0 ? 'text-teal-600' : 'text-rose-600'}">
							{deltaDilihat >= 0 ? '+' : ''}{deltaDilihat}%
						</span>
					</div>
				</div>
				<svg viewBox="0 0 {SPARK_W} {SPARK_H}" class="w-16 sm:w-20 h-8 sm:h-10 flex-shrink-0" preserveAspectRatio="none">
					<path d={areaPath(sparkDilihat, SPARK_H, SPARK_PAD)} class="fill-teal-600/10" />
					<path d={linePath(sparkDilihat)} fill="none" class="stroke-teal-600" stroke-width="1.5" />
				</svg>
			</div>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">gabungan dilihat semua barang di etalase, vs minggu lalu</p>
		</div>

		<!-- 6. TOTAL DISUKAI -->
		<div class="lg:col-span-6 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<div class="flex items-start justify-between gap-3">
				<div class="min-w-0">
					<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
						TOTAL DISUKAI
					</span>
					<div class="mt-2 flex items-baseline gap-2 flex-wrap">
						<span class="text-xl sm:text-2xl font-bold font-mono tracking-tight">
							{formatCompact(totalDisukai)}
						</span>
						<span class="text-[10px] sm:text-[11px] font-mono font-bold {deltaDisukai >= 0 ? 'text-teal-600' : 'text-rose-600'}">
							{deltaDisukai >= 0 ? '+' : ''}{deltaDisukai}%
						</span>
					</div>
				</div>
				<svg viewBox="0 0 {SPARK_W} {SPARK_H}" class="w-16 sm:w-20 h-8 sm:h-10 flex-shrink-0" preserveAspectRatio="none">
					<path d={areaPath(sparkDisukai, SPARK_H, SPARK_PAD)} class="fill-slate-950/5" />
					<path d={linePath(sparkDisukai)} fill="none" class="stroke-slate-950" stroke-width="1.5" />
				</svg>
			</div>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">gabungan disukai semua barang di etalase, vs minggu lalu</p>
		</div>

		<!-- ///////////////////////////////////////////////////////////// -->
		<!-- ROW C — distribusi barang per etalase + ranking paling dilihat -->
		<!-- ///////////////////////////////////////////////////////////// -->

		<!-- 7. DISTRIBUSI BARANG PER ETALASE -->
		<div class="lg:col-span-7 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				DISTRIBUSI BARANG PER ETALASE
			</span>

			<div class="mt-4 space-y-3 flex-1 flex flex-col justify-center">
				{#each distribusiBarang as e, i}
					{@const maxBarang = Math.max(distribusiBarang[0].jumlahBarang, 1)}
					<div>
						<div class="flex items-center justify-between mb-1">
							<span class="text-[10px] sm:text-[11px] font-medium truncate flex items-center gap-1.5">
								<span class="text-slate-400 font-mono">0{i + 1}</span>
								{e.nama}
							</span>
							<span class="text-[10px] sm:text-[11px] font-bold font-mono flex-shrink-0 ml-2">
								{e.jumlahBarang}
							</span>
						</div>
						<div class="w-full h-1 bg-slate-100 rounded-full overflow-hidden">
							<div
								class="h-full rounded-full {e.jumlahBarang === 0 ? 'bg-slate-200' : 'bg-slate-950'}"
								style:width="{(e.jumlahBarang / maxBarang) * 100}%"
							></div>
						</div>
					</div>
				{/each}
			</div>
		</div>

		<!-- 8. ETALASE PALING BANYAK DILIHAT -->
		<div class="lg:col-span-5 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				PALING BANYAK DILIHAT
			</span>

			<div class="mt-4 space-y-3 flex-1 flex flex-col justify-center">
				{#each topDilihat as e, i}
					{@const maxDilihat = Math.max(topDilihat[0].dilihat, 1)}
					<div>
						<div class="flex items-center justify-between mb-1">
							<span class="text-[10px] sm:text-[11px] font-medium truncate flex items-center gap-1.5">
								<span class="text-teal-700 font-mono">0{i + 1}</span>
								{e.nama}
							</span>
							<span class="text-[10px] sm:text-[11px] font-bold font-mono flex-shrink-0 ml-2">
								{formatCompact(e.dilihat)}
							</span>
						</div>
						<div class="w-full h-1 bg-slate-100 rounded-full overflow-hidden">
							<div class="h-full bg-teal-600 rounded-full" style:width="{(e.dilihat / maxDilihat) * 100}%"></div>
						</div>
					</div>
				{/each}
			</div>
		</div>

	</div>
</section>