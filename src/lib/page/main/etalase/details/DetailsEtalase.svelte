<script lang="ts">
	// ///////////////////////////////////////////////////////////////////////
	// Types
	// ///////////////////////////////////////////////////////////////////////

	type RangeMode = 'hari' | 'minggu' | 'bulan' | 'tahun' | 'custom';

	interface BucketData {
		label: string;
		dilihat: number;
		disukai: number;
	}

	interface BarangDalamEtalase {
		idBarangKeEtalase: number; // BarangKeEtalase.ID — dipakai buat "keluarkan dari etalase"
		idBarangInduk: number;
		nama: string;
		kategori: string;
		harga: number;
		deskripsi: string;
		viewed: number;
		likes: number;
		totalKomentar: number;
	}

	interface Point {
		x: number;
		y: number;
	}

	// ///////////////////////////////////////////////////////////////////////
	// Mock data — Etalase yang sedang dibuka
	// ganti dengan hasil fetch detail etalase by id_etalase
	// TODO: field foto/banner TIDAK ADA di struct Etalase manapun yang tersedia,
	// kemungkinan besar ini nempel di media_services (pola serupa dokumen
	// legalitas seller). fotoBannerUrl di sini murni placeholder.
	// ///////////////////////////////////////////////////////////////////////

	const etalase = {
		id_etalase: 1,
		nama_etalase: 'Koleksi Baju Pria',
		deskripsi_etalase:
			'Kumpulan kemeja, kaos, dan celana khusus kategori pria dewasa. Etalase ini fokus ke gaya kasual harian dengan bahan yang nyaman dipakai sehari-hari.',
		jumlah_barang: 27,
		created_at: '2026-03-02T09:15:00Z',
		updated_at: '2026-08-18T14:02:00Z',
		fotoBannerUrl: 'https://picsum.photos/seed/etalase-baju-pria/1200/400'
	};

	// ///////////////////////////////////////////////////////////////////////
	// State — filter periode analitik
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
			return jam.map((j, i) => ({ label: j, dilihat: seedNumber(i + 1, 20, 140), disukai: seedNumber(i + 20, 1, 12) }));
		}
		if (mode === 'minggu') {
			const hari = ['Sen', 'Sel', 'Rab', 'Kam', 'Jum', 'Sab', 'Min'];
			return hari.map((h, i) => ({ label: h, dilihat: seedNumber(i + 40, 150, 620), disukai: seedNumber(i + 60, 8, 46) }));
		}
		if (mode === 'bulan') {
			return Array.from({ length: 4 }, (_, i) => ({
				label: `Minggu ${i + 1}`,
				dilihat: seedNumber(i + 80, 900, 3200),
				disukai: seedNumber(i + 100, 40, 180)
			}));
		}
		if (mode === 'tahun') {
			const bulan = ['Jan', 'Feb', 'Mar', 'Apr', 'Mei', 'Jun', 'Jul', 'Agu', 'Sep', 'Okt', 'Nov', 'Des'];
			return bulan.map((b, i) => ({
				label: b,
				dilihat: seedNumber(i + 120, 3500, 14000),
				disukai: seedNumber(i + 140, 180, 720)
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
				dilihat: seedNumber(i + 160, 100, 700),
				disukai: seedNumber(i + 180, 5, 34)
			};
		});
	}

	let dataTren = $derived(bucketsForMode(rangeMode, customStart, customEnd));
	let totalDilihatPeriode = $derived(dataTren.reduce((a, d) => a + d.dilihat, 0));
	let totalDisukaiPeriode = $derived(dataTren.reduce((a, d) => a + d.disukai, 0));

	let deltaDilihat = $derived(seedNumber(totalDilihatPeriode % 97, -10, 24));
	let deltaDisukai = $derived(seedNumber(totalDisukaiPeriode % 61, -8, 20));

	function formatCompact(n: number): string {
		if (n >= 1_000_000) return `${(n / 1_000_000).toFixed(1)}jt`;
		if (n >= 1_000) return `${(n / 1_000).toFixed(1)}rb`;
		return `${n}`;
	}

	function formatTanggal(iso: string): string {
		return new Date(iso).toLocaleDateString('id-ID', { day: '2-digit', month: 'short', year: 'numeric' });
	}

	// ///////////////////////////////////////////////////////////////////////
	// SVG chart helper
	// ///////////////////////////////////////////////////////////////////////

	const CHART_W = 640;
	const CHART_H = 180;
	const PAD_X = 8;
	const PAD_Y = 14;

	function toPoints(values: number[], w: number, h: number, pad: number, sharedMax?: number): Point[] {
		const max = sharedMax ?? Math.max(...values, 1);
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

	let sharedMax = $derived(Math.max(...dataTren.map((d) => d.dilihat), 1));
	let dilihatPoints = $derived(toPoints(dataTren.map((d) => d.dilihat), CHART_W, CHART_H, PAD_X, sharedMax));
	let visibleLabelIdx = $derived(dataTren.map((_, i) => i).filter((i) => dataTren.length <= 8 || i % Math.ceil(dataTren.length / 8) === 0));

	// ///////////////////////////////////////////////////////////////////////
	// Mock data — barang terlaris di etalase ini (agregat dari transaksi)
	// ///////////////////////////////////////////////////////////////////////

	const barangTerlaris = [
		{ nama: 'Kemeja Flanel Kotak-kotak', terjual: 84 },
		{ nama: 'Kaos Polos Combed 30s', terjual: 61 },
		{ nama: 'Celana Chino Slimfit', terjual: 47 },
		{ nama: 'Jaket Bomber Varsity', terjual: 32 }
	];

	// ///////////////////////////////////////////////////////////////////////
	// State — edit info etalase (inline)
	// ///////////////////////////////////////////////////////////////////////

	let modeEdit = $state(false);
	let formEdit = $state({ nama: etalase.nama_etalase, deskripsi: etalase.deskripsi_etalase });

	function bukaEdit() {
		formEdit = { nama: etalase.nama_etalase, deskripsi: etalase.deskripsi_etalase };
		modeEdit = true;
	}

	function simpanEdit() {
		// TODO: panggil EditEtalase — hanya nama & deskripsi yang bisa diubah
		// dari struct ini; foto banner (kalau memang ada) lewat endpoint media
		// terpisah.
		etalase.nama_etalase = formEdit.nama;
		etalase.deskripsi_etalase = formEdit.deskripsi;
		modeEdit = false;
	}

	// ///////////////////////////////////////////////////////////////////////
	// Mock data — barang di dalam etalase ini
	// ///////////////////////////////////////////////////////////////////////

	let barangList = $state<BarangDalamEtalase[]>([
		{
			idBarangKeEtalase: 101,
			idBarangInduk: 1,
			nama: 'Kemeja Flanel Kotak-kotak',
			kategori: 'Atasan Pria',
			harga: 159_000,
			deskripsi: 'Bahan flanel tebal, cocok dipakai musim hujan, tersedia beberapa varian warna kotak.',
			viewed: 3240,
			likes: 210,
			totalKomentar: 42
		},
		{
			idBarangKeEtalase: 102,
			idBarangInduk: 2,
			nama: 'Kaos Polos Combed 30s',
			kategori: 'Atasan Pria',
			harga: 65_000,
			deskripsi: 'Bahan combed 30s adem, nyaman dipakai harian, tersedia banyak pilihan warna.',
			viewed: 5120,
			likes: 388,
			totalKomentar: 76
		},
		{
			idBarangKeEtalase: 103,
			idBarangInduk: 3,
			nama: 'Celana Chino Slimfit',
			kategori: 'Bawahan Pria',
			harga: 189_000,
			deskripsi: 'Potongan slimfit modern, bahan stretch nyaman gerak, cocok formal maupun kasual.',
			viewed: 2870,
			likes: 174,
			totalKomentar: 29
		},
		{
			idBarangKeEtalase: 104,
			idBarangInduk: 4,
			nama: 'Jaket Bomber Varsity',
			kategori: 'Outerwear Pria',
			harga: 245_000,
			deskripsi: 'Jaket bomber gaya varsity, bahan tebal hangat, cocok buat cuaca dingin atau AC kantor.',
			viewed: 1980,
			likes: 156,
			totalKomentar: 18
		}
	]);

	function keluarkanDariEtalase(id: number) {
		// TODO: panggil HapusBarangKeEtalase — ini hapus RELASI, bukan hapus
		// barangnya. BarangKeEtalase hard delete, jadi begitu dihapus langsung
		// hilang tanpa bisa dipulihkan lewat soft-delete.
		barangList = barangList.filter((b) => b.idBarangKeEtalase !== id);
	}

	function formatRupiah(n: number): string {
		return `Rp${n.toLocaleString('id-ID')}`;
	}
</script>

<section id="details-etalase" class="w-full bg-white text-slate-950">
	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- BREADCRUMB RINGKAS -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="px-4 sm:px-6 lg:px-8 pt-4 sm:pt-6">
		<span class="text-[9px] sm:text-[10px] font-mono text-slate-400 uppercase tracking-wider">
			Etalase Saya / {etalase.nama_etalase}
		</span>
	</div>

	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- SECTION 1 — DATA VISUAL PELAPORAN ETALASE INI -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="px-4 sm:px-6 lg:px-8 pt-3 pb-6 mb-6 border-b border-zinc-800/10">
		<div class="flex flex-col lg:flex-row lg:items-end lg:justify-between gap-4 mb-4">
			<div>
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
					ANALITIK ETALASE INI
				</span>
				<h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
					Performa {etalase.nama_etalase}
				</h1>
			</div>

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
		</div>

		{#if rangeMode === 'custom'}
			<div class="flex items-center gap-1.5 text-[10px] sm:text-[11px] font-mono mb-4">
				<input type="date" bind:value={customStart} class="border border-zinc-800/20 rounded-xs px-2 py-1.5 text-slate-950 focus:outline-none focus:border-slate-950" />
				<span class="text-slate-400">—</span>
				<input type="date" bind:value={customEnd} class="border border-zinc-800/20 rounded-xs px-2 py-1.5 text-slate-950 focus:outline-none focus:border-slate-950" />
			</div>
		{/if}

		<div class="grid grid-cols-1 lg:grid-cols-12 gap-4">
			<!-- KPI Total Barang -->
			<div class="lg:col-span-3 border border-zinc-800/20 rounded-sm p-4 flex flex-col justify-between">
				<span class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">Total Barang</span>
				<span class="mt-2 text-2xl font-bold font-mono tracking-tight">{etalase.jumlah_barang}</span>
				<p class="text-[9px] text-slate-400 font-light mt-2">tersimpan di etalase ini</p>
			</div>

			<!-- KPI Total Dilihat -->
			<div class="lg:col-span-4 border border-zinc-800/20 rounded-sm p-4 flex flex-col justify-between">
				<span class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">Total Dilihat</span>
				<div class="mt-2 flex items-baseline gap-2 flex-wrap">
					<span class="text-2xl font-bold font-mono tracking-tight text-teal-700">{formatCompact(totalDilihatPeriode)}</span>
					<span class="text-[10px] font-mono font-bold {deltaDilihat >= 0 ? 'text-teal-600' : 'text-rose-600'}">
						{deltaDilihat >= 0 ? '+' : ''}{deltaDilihat}%
					</span>
				</div>
				<p class="text-[9px] text-slate-400 font-light mt-2">gabungan semua barang di etalase ini</p>
			</div>

			<!-- KPI Total Disukai -->
			<div class="lg:col-span-5 border border-zinc-800/20 rounded-sm p-4 flex flex-col justify-between">
				<span class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">Total Disukai</span>
				<div class="mt-2 flex items-baseline gap-2 flex-wrap">
					<span class="text-2xl font-bold font-mono tracking-tight">{formatCompact(totalDisukaiPeriode)}</span>
					<span class="text-[10px] font-mono font-bold {deltaDisukai >= 0 ? 'text-teal-600' : 'text-rose-600'}">
						{deltaDisukai >= 0 ? '+' : ''}{deltaDisukai}%
					</span>
				</div>
				<p class="text-[9px] text-slate-400 font-light mt-2">gabungan semua barang di etalase ini</p>
			</div>

			<!-- Chart tren dilihat -->
			<div class="lg:col-span-7 border border-zinc-800/20 rounded-sm p-4">
				<span class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">Tren Dilihat</span>
				<div class="mt-2 w-full">
					<svg viewBox="0 0 {CHART_W} {CHART_H}" class="w-full h-[9rem]" preserveAspectRatio="none">
						{#each [0.25, 0.5, 0.75] as g}
							<line x1={PAD_X} x2={CHART_W - PAD_X} y1={PAD_Y + (CHART_H - PAD_Y * 2) * g} y2={PAD_Y + (CHART_H - PAD_Y * 2) * g} stroke="currentColor" class="text-zinc-800/10" stroke-width="1" />
						{/each}
						<path d={areaPath(dilihatPoints, CHART_H, PAD_X)} class="fill-teal-600/10" />
						<path d={linePath(dilihatPoints)} fill="none" class="stroke-teal-600" stroke-width="2" />
					</svg>
					<div class="relative h-4 mt-1">
						{#each visibleLabelIdx as i}
							<span class="absolute -translate-x-1/2 text-[8px] font-mono text-slate-400 uppercase tracking-wider whitespace-nowrap" style:left="{(dilihatPoints[i].x / CHART_W) * 100}%">
								{dataTren[i].label}
							</span>
						{/each}
					</div>
				</div>
			</div>

			<!-- Barang terlaris di etalase ini -->
			<div class="lg:col-span-5 border border-zinc-800/20 rounded-sm p-4 flex flex-col">
				<span class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">Barang Terlaris di Etalase Ini</span>
				<div class="mt-3 space-y-2.5 flex-1 flex flex-col justify-center">
					{#each barangTerlaris as b, i}
						{@const maxTerjual = Math.max(barangTerlaris[0].terjual, 1)}
						<div>
							<div class="flex items-center justify-between mb-1">
								<span class="text-[10px] font-medium truncate flex items-center gap-1.5">
									<span class="text-teal-700 font-mono">0{i + 1}</span>
									{b.nama}
								</span>
								<span class="text-[10px] font-bold font-mono flex-shrink-0 ml-2">{b.terjual}</span>
							</div>
							<div class="w-full h-1 bg-slate-100 rounded-full overflow-hidden">
								<div class="h-full bg-teal-600 rounded-full" style:width="{(b.terjual / maxTerjual) * 100}%"></div>
							</div>
						</div>
					{/each}
				</div>
			</div>
		</div>
	</div>

	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- SECTION 2 — BANNER, NAMA, DESKRIPSI ETALASE -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="mb-8">
		<!-- Banner -->
		<div class="relative w-full h-40 sm:h-56 lg:h-64 bg-zinc-100 overflow-hidden">
			<img src={etalase.fotoBannerUrl} alt="Banner {etalase.nama_etalase}" class="w-full h-full object-cover" />
			<div class="absolute inset-0 bg-gradient-to-t from-black/60 via-black/10 to-transparent"></div>
			<div class="absolute bottom-3 left-4 sm:left-6 lg:left-8 right-4">
				<span class="px-2 py-0.5 rounded text-[9px] font-medium uppercase tracking-wider bg-white/90 text-slate-800">
					{etalase.jumlah_barang} Barang
				</span>
				<h2 class="mt-1.5 text-lg sm:text-2xl font-bold text-white tracking-tight leading-none drop-shadow">
					{etalase.nama_etalase}
				</h2>
			</div>
			<button
				type="button"
				class="absolute top-3 right-3 sm:right-4 lg:right-6 px-3 py-1.5 bg-white/90 hover:bg-white text-[10px] font-bold uppercase tracking-wider text-slate-800 rounded-sm transition-colors"
			>
				Ganti Banner
			</button>
		</div>

		<!-- Info & deskripsi -->
		<div class="px-4 sm:px-6 lg:px-8 pt-4">
			{#if !modeEdit}
				<div class="flex items-start justify-between gap-3 flex-wrap">
					<div class="max-w-2xl">
						<p class="text-xs sm:text-sm text-slate-700 leading-relaxed">{etalase.deskripsi_etalase}</p>
						<div class="flex items-center gap-3 mt-3 text-[10px] font-mono text-slate-400">
							<span>Dibuat {formatTanggal(etalase.created_at)}</span>
							<span>·</span>
							<span>Diperbarui {formatTanggal(etalase.updated_at)}</span>
						</div>
					</div>
					<button
						type="button"
						onclick={bukaEdit}
						class="text-[10px] font-bold uppercase tracking-wider text-slate-950 border border-zinc-300 rounded px-3 py-1.5 hover:bg-zinc-50 transition-colors flex-shrink-0"
					>
						Edit Etalase
					</button>
				</div>
			{:else}
				<div class="border border-zinc-800/20 rounded-sm p-4 max-w-2xl">
					<div class="flex flex-col gap-3">
						<label class="flex flex-col gap-1.5">
							<span class="text-[10px] font-medium text-slate-600">Nama Etalase</span>
							<input
								type="text"
								bind:value={formEdit.nama}
								class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 focus:outline-none focus:border-slate-950 transition-colors"
							/>
						</label>
						<label class="flex flex-col gap-1.5">
							<span class="text-[10px] font-medium text-slate-600">Deskripsi</span>
							<textarea
								bind:value={formEdit.deskripsi}
								rows={3}
								class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 focus:outline-none focus:border-slate-950 transition-colors resize-none"
							></textarea>
						</label>
						<div class="flex items-center gap-3">
							<button type="button" onclick={simpanEdit} class="px-4 py-2 bg-slate-950 text-white text-[10px] font-bold uppercase tracking-wider rounded-sm hover:bg-slate-800 transition-colors">
								Simpan Perubahan
							</button>
							<button type="button" onclick={() => (modeEdit = false)} class="text-[10px] font-medium text-slate-500 underline">
								Batal
							</button>
						</div>
					</div>
				</div>
			{/if}
		</div>
	</div>

	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- SECTION 3 — LIST BARANG DALAM ETALASE -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="px-4 sm:px-6 lg:px-8 pb-8">
		<div class="flex items-center justify-between flex-wrap gap-2 pb-3 mb-4 border-b border-zinc-800/10">
			<div>
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
					ISI ETALASE
				</span>
				<h2 class="mt-1 text-base sm:text-lg font-bold uppercase tracking-tight leading-none">
					{barangList.length} Barang Terdaftar
				</h2>
			</div>
			<button
				type="button"
				class="text-[10px] font-bold uppercase tracking-wider text-slate-950 border border-zinc-300 rounded px-3 py-1.5 hover:bg-zinc-50 transition-colors"
			>
				+ Tambah Barang ke Etalase
			</button>
		</div>

		<div class="flex flex-wrap gap-4">
			{#each barangList as barang (barang.idBarangKeEtalase)}
				<div class="w-full sm:w-[23rem] border border-zinc-200 hover:border-zinc-400 rounded-lg bg-white shadow-sm overflow-hidden flex flex-col transition-colors duration-150">
					<div class="p-3 flex flex-col gap-2">
						<div class="flex items-start justify-between gap-2">
							<div class="min-w-0">
								<span class="px-1.5 py-0.5 bg-zinc-100 text-zinc-600 text-[9px] font-medium rounded uppercase tracking-wider">
									{barang.kategori}
								</span>
								<h3 class="text-xs font-semibold text-zinc-800 leading-tight truncate mt-1">{barang.nama}</h3>
							</div>
							<span class="text-[11px] font-bold text-zinc-900 font-mono flex-shrink-0">{formatRupiah(barang.harga)}</span>
						</div>
						<p class="text-[10px] text-zinc-400 line-clamp-2 leading-tight">{barang.deskripsi}</p>

						<div class="grid grid-cols-3 gap-2 pt-2 mt-1 border-t border-zinc-100 text-[10px] text-zinc-600 font-mono">
							<div class="flex items-center gap-1">
								<svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-zinc-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
									<path d="M2.062 12.348a1 1 0 0 1 0-.696 10.75 10.75 0 0 1 19.876 0 1 1 0 0 1 0 .696 10.75 10.75 0 0 1-19.876 0"/><circle cx="12" cy="12" r="3"/>
								</svg>
								<span>{formatCompact(barang.viewed)}</span>
							</div>
							<div class="flex items-center gap-1">
								<svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-zinc-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
									<path d="M19 14c1.49-1.46 3-3.21 3-5.5A5.5 5.5 0 0 0 16.5 3c-1.76 0-3 .5-4.5 2-1.5-1.5-2.74-2-4.5-2A5.5 5.5 0 0 0 2 8.5c0 2.3 1.5 4.05 3 5.5l7 7Z"/>
								</svg>
								<span>{barang.likes}</span>
							</div>
							<div class="flex items-center gap-1">
								<svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-zinc-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
									<path d="M7.9 20A9 9 0 1 0 4 16.1L2 22Z"/>
								</svg>
								<span>{barang.totalKomentar}</span>
							</div>
						</div>

						<button
							type="button"
							onclick={() => keluarkanDariEtalase(barang.idBarangKeEtalase)}
							class="mt-1 text-[9px] font-medium text-rose-500 underline self-start"
						>
							Keluarkan dari Etalase
						</button>
					</div>
				</div>
			{/each}

			{#if barangList.length === 0}
				<p class="text-[11px] text-zinc-400 py-8 w-full text-center">Etalase ini masih kosong. Tambahkan barang supaya bisa dilihat pembeli.</p>
			{/if}
		</div>
	</div>
</section>