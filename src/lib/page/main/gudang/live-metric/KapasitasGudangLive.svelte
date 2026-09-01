<script lang="ts">
	// ///////////////////////////////////////////////////////////////////////
	// Types
	// ///////////////////////////////////////////////////////////////////////

	interface VarianDiGudang {
		idKategoriBarang: number;
		namaBarangInduk: string; // BarangInduk.NamaBarang (join)
		namaVarian: string; // KategoriBarang.Nama
		stok: number; // KategoriBarang.Stok
		beratGram: number; // KategoriBarang.BeratGram
	}

	// ///////////////////////////////////////////////////////////////////////
	// Mock data — ganti dengan fetch KategoriBarang WHERE id_alamat_gudang = ini,
	// di-join ke BarangInduk buat dapat nama barang induknya
	// ///////////////////////////////////////////////////////////////////////

	const namaGudang = 'Gudang Utama Cikarang';
	const ambangStokKritis = 10;

	const variasiDiGudang: VarianDiGudang[] = [
		{ idKategoriBarang: 1, namaBarangInduk: 'Kemeja Flanel Kotak-kotak', namaVarian: 'Merah, size L', stok: 42, beratGram: 500 },
		{ idKategoriBarang: 2, namaBarangInduk: 'Kemeja Flanel Kotak-kotak', namaVarian: 'Biru, size M', stok: 3, beratGram: 480 },
		{ idKategoriBarang: 3, namaBarangInduk: 'Kaos Polos Combed 30s', namaVarian: 'Hitam, size M', stok: 128, beratGram: 180 },
		{ idKategoriBarang: 4, namaBarangInduk: 'Kaos Polos Combed 30s', namaVarian: 'Putih, size L', stok: 94, beratGram: 190 },
		{ idKategoriBarang: 5, namaBarangInduk: 'Celana Chino Slimfit', namaVarian: 'Khaki, size 32', stok: 7, beratGram: 420 },
		{ idKategoriBarang: 6, namaBarangInduk: 'Celana Chino Slimfit', namaVarian: 'Navy, size 30', stok: 61, beratGram: 410 },
		{ idKategoriBarang: 7, namaBarangInduk: 'Jaket Bomber Varsity', namaVarian: 'Navy, size L', stok: 5, beratGram: 900 },
		{ idKategoriBarang: 8, namaBarangInduk: 'Tumbler Stainless 500ml', namaVarian: 'Sage Green', stok: 210, beratGram: 320 },
		{ idKategoriBarang: 9, namaBarangInduk: 'Tumbler Stainless 500ml', namaVarian: 'Hitam Doff', stok: 8, beratGram: 320 },
		{ idKategoriBarang: 10, namaBarangInduk: 'Matras Yoga Anti Slip', namaVarian: '6mm, Ungu', stok: 36, beratGram: 1_100 }
	];

	// ///////////////////////////////////////////////////////////////////////
	// Derived — 5 metrik utama
	// ///////////////////////////////////////////////////////////////////////

	let totalVarian = $derived(variasiDiGudang.length);
	let totalStok = $derived(variasiDiGudang.reduce((a, v) => a + v.stok, 0));
	let totalBeratGram = $derived(variasiDiGudang.reduce((a, v) => a + v.stok * v.beratGram, 0));
	let totalBeratKg = $derived(totalBeratGram / 1000);

	let distribusiPerBarangInduk = $derived(() => {
		const map = new Map<string, number>();
		for (const v of variasiDiGudang) {
			map.set(v.namaBarangInduk, (map.get(v.namaBarangInduk) ?? 0) + v.stok);
		}
		return Array.from(map.entries())
			.map(([nama, stok]) => ({ nama, stok }))
			.sort((a, b) => b.stok - a.stok);
	});

	let varianStokKritis = $derived(
		[...variasiDiGudang].filter((v) => v.stok < ambangStokKritis).sort((a, b) => a.stok - b.stok)
	);

	function formatBerat(kg: number): string {
		if (kg >= 1000) return `${(kg / 1000).toFixed(1)} ton`;
		return `${kg.toFixed(1)} kg`;
	}

	// ///////////////////////////////////////////////////////////////////////
	// Indikator "live" — refresh manual, mock waktu update terakhir
	// ///////////////////////////////////////////////////////////////////////

	let waktuDiperbarui = $state(new Date());
	let sedangRefresh = $state(false);

	function refreshData() {
		// TODO: ganti dengan re-fetch data KategoriBarang untuk gudang ini.
		// Untuk beneran "live", ini idealnya websocket/polling interval, bukan
		// cuma tombol manual seperti sekarang.
		sedangRefresh = true;
		setTimeout(() => {
			waktuDiperbarui = new Date();
			sedangRefresh = false;
		}, 600);
	}

	function formatJam(d: Date): string {
		return d.toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit', second: '2-digit' });
	}
</script>

<section id="kapasitas-gudang-live" class="w-full bg-white p-4 sm:p-6 lg:p-8 text-slate-950">
	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- HEADER -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="flex flex-col sm:flex-row sm:items-end sm:justify-between gap-3 pb-6 mb-6 border-b border-zinc-800/10">
		<div>
			<div class="flex items-center gap-2">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
					KAPASITAS GUDANG
				</span>
				<span class="inline-flex items-center gap-1 px-1.5 py-0.5 rounded bg-rose-50 text-rose-600 text-[8px] font-bold uppercase tracking-wider">
					<span class="w-1.5 h-1.5 rounded-full bg-rose-500 animate-pulse"></span>
					Live
				</span>
			</div>
			<h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
				{namaGudang}
			</h1>
		</div>

		<div class="flex items-center gap-2">
			<span class="text-[9px] font-mono text-slate-400">Diperbarui {formatJam(waktuDiperbarui)}</span>
			<button
				type="button"
				onclick={refreshData}
				class="px-3 py-1.5 border border-zinc-300 rounded-sm text-[9px] font-bold uppercase tracking-wider text-slate-700 hover:bg-zinc-50 transition-colors"
			>
				{sedangRefresh ? 'Memuat...' : 'Refresh'}
			</button>
		</div>
	</div>

	<div class="grid grid-cols-1 lg:grid-cols-12 gap-4">
		<!-- 1. TOTAL VARIAN DI GUDANG INI -->
		<div class="lg:col-span-4 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				TOTAL VARIAN BARANG
			</span>
			<span class="mt-2 text-2xl sm:text-3xl font-bold font-mono tracking-tight">
				{totalVarian}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">
				kategori/varian tersimpan di gudang ini
			</p>
		</div>

		<!-- 2. TOTAL STOK KESELURUHAN -->
		<div class="lg:col-span-4 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				TOTAL STOK KESELURUHAN
			</span>
			<span class="mt-2 text-2xl sm:text-3xl font-bold font-mono tracking-tight text-teal-700">
				{totalStok.toLocaleString('id-ID')}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">
				unit, gabungan semua varian di gudang ini
			</p>
		</div>

		<!-- 3. KAPASITAS BERDASARKAN BERAT -->
		<div class="lg:col-span-4 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col justify-between">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				KAPASITAS BERDASARKAN BERAT
			</span>
			<span class="mt-2 text-2xl sm:text-3xl font-bold font-mono tracking-tight">
				{formatBerat(totalBeratKg)}
			</span>
			<p class="text-[9px] sm:text-[10px] text-slate-400 font-light mt-3">
				total berat seluruh stok (stok × berat per unit)
			</p>
		</div>

		<!-- 4. DISTRIBUSI STOK PER BARANG INDUK -->
		<div class="lg:col-span-7 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				DISTRIBUSI STOK PER BARANG INDUK
			</span>

			<div class="mt-4 space-y-3 flex-1 flex flex-col justify-center">
				{#each distribusiPerBarangInduk() as item, i}
					{@const maxStok = Math.max(distribusiPerBarangInduk()[0].stok, 1)}
					<div>
						<div class="flex items-center justify-between mb-1">
							<span class="text-[10px] sm:text-[11px] font-medium truncate flex items-center gap-1.5">
								<span class="text-teal-700 font-mono">0{i + 1}</span>
								{item.nama}
							</span>
							<span class="text-[10px] sm:text-[11px] font-bold font-mono flex-shrink-0 ml-2">
								{item.stok} unit
							</span>
						</div>
						<div class="w-full h-1 bg-slate-100 rounded-full overflow-hidden">
							<div class="h-full bg-teal-600 rounded-full" style:width="{(item.stok / maxStok) * 100}%"></div>
						</div>
					</div>
				{/each}
			</div>
		</div>

		<!-- 5. VARIAN STOK KRITIS DI GUDANG INI — actionable -->
		<div class="lg:col-span-5 border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col">
			<div class="flex items-center justify-between gap-2">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					STOK KRITIS DI GUDANG INI
				</span>
				{#if varianStokKritis.length > 0}
					<span class="w-1.5 h-1.5 rounded-full bg-rose-500 flex-shrink-0"></span>
				{/if}
			</div>
			<span class="text-[9px] text-slate-400 font-light mt-0.5">di bawah {ambangStokKritis} unit</span>

			<div class="mt-3 flex-1 flex flex-col justify-center gap-2">
				{#each varianStokKritis as v (v.idKategoriBarang)}
					<div class="flex items-center justify-between gap-2 border border-rose-100 bg-rose-50/50 rounded-md px-3 py-2">
						<div class="min-w-0">
							<p class="text-[10px] font-semibold text-zinc-800 truncate">{v.namaBarangInduk}</p>
							<p class="text-[9px] text-zinc-500 truncate">{v.namaVarian}</p>
						</div>
						<span class="text-[10px] font-bold font-mono text-rose-600 flex-shrink-0">{v.stok} tersisa</span>
					</div>
				{/each}

				{#if varianStokKritis.length === 0}
					<p class="text-[11px] text-zinc-400 text-center py-4">Semua varian di gudang ini stoknya aman.</p>
				{/if}
			</div>
		</div>
	</div>
</section>