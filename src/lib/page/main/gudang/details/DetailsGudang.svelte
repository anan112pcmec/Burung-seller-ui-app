<script lang="ts">
  import CardProduk from "../../general/CardProduk.svelte";

	const ulang = 10;
	// ///////////////////////////////////////////////////////////////////////
	// Types
	// ///////////////////////////////////////////////////////////////////////

	interface AlamatGudangForm {
		panggilanAlamat: string;
		nomorTelepon: string;
		namaAlamat: string;
		provinsi: string;
		kota: string;
		kodePos: string;
		kodeNegara: string;
		deskripsi: string;
		longitude: number;
		latitude: number;
	}

	// ///////////////////////////////////////////////////////////////////////
	// Mock data — ganti dengan fetch detail AlamatGudang by id
	// TODO: field foto TIDAK ADA di struct AlamatGudang, kemungkinan besar ini
	// nempel di media_services (pola sama seperti dokumen legalitas & banner
	// etalase sebelumnya). fotoGudangUrl di sini murni placeholder.
	// ///////////////////////////////////////////////////////////////////////

	const idAlamatGudang = 8;
	const createdAt = '2026-01-14T08:00:00Z';
	const updatedAt = '2026-08-20T10:32:00Z';
	const fotoGudangUrl = 'https://picsum.photos/seed/gudang-cikarang/800/400';

	let form = $state<AlamatGudangForm>({
		panggilanAlamat: 'Gudang Utama Cikarang',
		nomorTelepon: '021-89012345',
		namaAlamat: 'Jl. Industri Raya No. 8, Kawasan Industri Jababeka',
		provinsi: 'Jawa Barat',
		kota: 'Cikarang',
		kodePos: '17530',
		kodeNegara: 'IDN',
		deskripsi: 'Gudang utama untuk penyimpanan stok fashion & elektronik. Akses truk besar tersedia, ada loading dock di sisi timur bangunan.',
		longitude: 107.154_121,
		latitude: -6.263_442
	});

	// ///////////////////////////////////////////////////////////////////////
	// Helpers
	// ///////////////////////////////////////////////////////////////////////

	function formatTanggal(iso: string): string {
		return new Date(iso).toLocaleDateString('id-ID', { day: '2-digit', month: 'long', year: 'numeric' });
	}

	let tautanGoogleMaps = $derived(`https://www.google.com/maps?q=${form.latitude},${form.longitude}`);

	let statusSimpan = $state<'idle' | 'tersimpan'>('idle');

	function simpanPerubahan() {
		// TODO: panggil EditAlamatGudang dengan seluruh field di atas.
		// Perhatikan: alamat ini dipakai sebagai referensi di Transaksi
		// (id_alamat_gudang), jadi ganti alamat di sini bisa memengaruhi
		// perhitungan ongkir/jarak tempuh transaksi yang sedang berjalan.
		statusSimpan = 'tersimpan';
		setTimeout(() => (statusSimpan = 'idle'), 2000);
	}
</script>

<section id="details-gudang" class="w-full bg-white text-slate-950">
	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- BREADCRUMB -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="px-4 sm:px-6 lg:px-8 pt-4 sm:pt-6">
		<span class="text-[9px] sm:text-[10px] font-mono text-slate-400 uppercase tracking-wider">
			Alamat Gudang / {form.panggilanAlamat}
		</span>
	</div>

	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- FOTO GUDANG -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="relative w-full h-40 sm:h-56 lg:h-64 bg-zinc-100 overflow-hidden mt-3">
		<img src={fotoGudangUrl} alt="Foto {form.panggilanAlamat}" class="w-full h-full object-cover" />
		<div class="absolute inset-0 bg-gradient-to-t from-black/60 via-black/10 to-transparent"></div>
		<div class="absolute bottom-3 left-4 sm:left-6 lg:left-8 right-4">
			<h1 class="text-lg sm:text-2xl font-bold text-white tracking-tight leading-none drop-shadow">
				{form.panggilanAlamat}
			</h1>
			<p class="text-[11px] text-white/80 mt-1">{form.kota}, {form.provinsi}</p>
		</div>
		<button
			type="button"
			class="absolute top-3 right-3 sm:right-4 lg:right-6 px-3 py-1.5 bg-white/90 hover:bg-white text-[10px] font-bold uppercase tracking-wider text-slate-800 rounded-sm transition-colors"
		>
			Ganti Foto
		</button>
	</div>

	<div class="px-4 sm:px-6 lg:px-8 py-6">
		<div class="flex items-center justify-between flex-wrap gap-3 pb-4 mb-5 border-b border-zinc-800/10">
			<div>
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
					DETAIL ALAMAT GUDANG
				</span>
				<h2 class="mt-1 text-lg sm:text-xl font-bold uppercase tracking-tight leading-none">
					Informasi &amp; Lokasi
				</h2>
			</div>
			<div class="flex items-center gap-3">
				{#if statusSimpan === 'tersimpan'}
					<span class="text-[10px] font-medium text-teal-700">Tersimpan ✓</span>
				{/if}
				<button
					type="button"
					onclick={simpanPerubahan}
					class="px-4 py-2 bg-slate-950 text-white text-[10px] font-bold uppercase tracking-wider rounded-sm hover:bg-slate-800 transition-colors"
				>
					Simpan Perubahan
				</button>
			</div>
		</div>

		<div class="grid grid-cols-1 lg:grid-cols-12 gap-5">
			<!-- ///////////////////////////////////////////////////////// -->
			<!-- KOLOM KIRI — form data alamat -->
			<!-- ///////////////////////////////////////////////////////// -->
			<div class="lg:col-span-8 flex flex-col gap-4">
				<div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5">
					<span class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
						Identitas Gudang
					</span>

					<div class="mt-3 grid grid-cols-1 sm:grid-cols-2 gap-4">
						<label class="flex flex-col gap-1.5 sm:col-span-2">
							<span class="text-[10px] font-medium text-slate-600">Nama Panggilan Alamat</span>
							<input
								type="text"
								bind:value={form.panggilanAlamat}
								class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 focus:outline-none focus:border-slate-950 transition-colors"
							/>
						</label>

						<label class="flex flex-col gap-1.5">
							<span class="text-[10px] font-medium text-slate-600">Nomor Telepon</span>
							<input
								type="tel"
								bind:value={form.nomorTelepon}
								class="border border-zinc-300 rounded-md px-3 py-2 text-xs font-mono text-slate-900 focus:outline-none focus:border-slate-950 transition-colors"
							/>
						</label>

						<label class="flex flex-col gap-1.5">
							<span class="text-[10px] font-medium text-slate-600">Kode Negara</span>
							<input
								type="text"
								bind:value={form.kodeNegara}
								maxlength="3"
								class="border border-zinc-300 rounded-md px-3 py-2 text-xs font-mono uppercase text-slate-900 focus:outline-none focus:border-slate-950 transition-colors"
							/>
						</label>
					</div>
				</div>

				<div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5">
					<span class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
						Alamat Lengkap
					</span>

					<div class="mt-3 flex flex-col gap-4">
						<label class="flex flex-col gap-1.5">
							<span class="text-[10px] font-medium text-slate-600">Nama Alamat / Jalan</span>
							<textarea
								bind:value={form.namaAlamat}
								rows={2}
								class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 focus:outline-none focus:border-slate-950 transition-colors resize-none"
							></textarea>
						</label>

						<div class="grid grid-cols-1 sm:grid-cols-3 gap-4">
							<label class="flex flex-col gap-1.5">
								<span class="text-[10px] font-medium text-slate-600">Provinsi</span>
								<input
									type="text"
									bind:value={form.provinsi}
									class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 focus:outline-none focus:border-slate-950 transition-colors"
								/>
							</label>
							<label class="flex flex-col gap-1.5">
								<span class="text-[10px] font-medium text-slate-600">Kota</span>
								<input
									type="text"
									bind:value={form.kota}
									class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 focus:outline-none focus:border-slate-950 transition-colors"
								/>
							</label>
							<label class="flex flex-col gap-1.5">
								<span class="text-[10px] font-medium text-slate-600">Kode Pos</span>
								<input
									type="text"
									bind:value={form.kodePos}
									class="border border-zinc-300 rounded-md px-3 py-2 text-xs font-mono text-slate-900 focus:outline-none focus:border-slate-950 transition-colors"
								/>
							</label>
						</div>

						<label class="flex flex-col gap-1.5">
							<span class="text-[10px] font-medium text-slate-600">Deskripsi / Catatan Akses</span>
							<textarea
								bind:value={form.deskripsi}
								rows={3}
								placeholder="Info tambahan: patokan lokasi, akses kendaraan, jam operasional, dll."
								class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 placeholder-zinc-400 focus:outline-none focus:border-slate-950 transition-colors resize-none"
							></textarea>
						</label>
					</div>
				</div>
			</div>

			<!-- ///////////////////////////////////////////////////////// -->
			<!-- KOLOM KANAN — koordinat & metadata -->
			<!-- ///////////////////////////////////////////////////////// -->
			<div class="lg:col-span-4 flex flex-col gap-4">
				<div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5">
					<span class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
						Titik Koordinat
					</span>

					<div class="mt-3 grid grid-cols-2 gap-3">
						<label class="flex flex-col gap-1.5">
							<span class="text-[10px] font-medium text-slate-600">Latitude</span>
							<input
								type="number"
								step="0.00000001"
								bind:value={form.latitude}
								class="border border-zinc-300 rounded-md px-3 py-2 text-xs font-mono text-slate-900 focus:outline-none focus:border-slate-950 transition-colors"
							/>
						</label>
						<label class="flex flex-col gap-1.5">
							<span class="text-[10px] font-medium text-slate-600">Longitude</span>
							<input
								type="number"
								step="0.00000001"
								bind:value={form.longitude}
								class="border border-zinc-300 rounded-md px-3 py-2 text-xs font-mono text-slate-900 focus:outline-none focus:border-slate-950 transition-colors"
							/>
						</label>
					</div>

					<a
						href={tautanGoogleMaps}
						target="_blank"
						rel="noopener noreferrer"
						class="mt-3 flex items-center justify-center gap-1.5 w-full border border-zinc-300 rounded-md px-3 py-2 text-[10px] font-bold uppercase tracking-wider text-slate-700 hover:bg-zinc-50 transition-colors">
					
						<svg xmlns="http://www.w3.org/2000/svg" class="w-3.5 h-3.5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
							<path d="M20 10c0 6-8 12-8 12s-8-6-8-12a8 8 0 0 1 16 0Z"/><circle cx="12" cy="10" r="3"/>
						</svg>
						Buka di Google Maps
					</a>
					<p class="text-[9px] text-zinc-400 mt-2 leading-relaxed">
						Koordinat ini dipakai sistem untuk menghitung jarak tempuh & estimasi ongkir pada transaksi yang
						dikirim dari gudang ini. Pastikan titiknya akurat.
					</p>
				</div>

				<div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5">
					<span class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
						Metadata
					</span>
					<div class="mt-3 flex flex-col gap-2 text-[11px]">
						<div class="flex items-center justify-between">
							<span class="text-zinc-500">ID Alamat Gudang</span>
							<span class="font-mono text-zinc-700">{idAlamatGudang}</span>
						</div>
						<div class="flex items-center justify-between">
							<span class="text-zinc-500">Dibuat</span>
							<span class="font-mono text-zinc-700">{formatTanggal(createdAt)}</span>
						</div>
						<div class="flex items-center justify-between">
							<span class="text-zinc-500">Diperbarui</span>
							<span class="font-mono text-zinc-700">{formatTanggal(updatedAt)}</span>
						</div>
					</div>
				</div>
			</div>
		</div>
		<div class="mt-10">
    <div class="flex items-center justify-between mb-3">
   <div>
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
					DETAIL ALAMAT GUDANG
				</span>
				<h2 class="mt-1 text-lg sm:text-xl font-bold uppercase tracking-tight leading-none">
					Barang Dalam Gudang
				</h2>
			</div>
    
    <div class="flex items-center gap-2">
        <!-- Search Input -->
        <input
            type="text"
            placeholder="Cari nama barang..."
            class="px-2.5 py-1 text-[10px] font-sans bg-white border border-slate-300 rounded-xs text-slate-950 placeholder:text-slate-400 focus:outline-none focus:border-slate-950 transition-colors"
        />

        <!-- Sort Dropdown -->
        <select
            class="px-2.5 py-1 text-[10px] font-sans bg-white border border-slate-300 rounded-xs text-slate-950 focus:outline-none focus:border-slate-950 transition-colors cursor-pointer"
        >
            <option value="terbaru">Urutkan: Terbaru</option>
            <option value="harga_asc">Harga: Rendah ke Tinggi</option>
            <option value="harga_desc">Harga: Tinggi ke Rendah</option>
            <option value="populer">Paling Populer</option>
        </select>

        <button
            class="text-[9px] sm:text-[10px] font-medium uppercase tracking-wider border rounded-xs px-3 py-1.5 transition-colors border-slate-950 text-slate-950 hover:bg-slate-950 hover:text-white shrink-0"
        >
            + Tambah Barang
        </button>
    </div>
</div>

    <div class="flex flex-wrap gap-4">
        {#each Array(ulang) as _}
            <CardProduk
                kategori="Elektronik" 
                harga={850000} 
                nama="Wireless Mechanical Keyboard" 
                deskripsi="Keyboard mekanik 65% dengan konektivitas Bluetooth 5.1 dan Gateron switches." 
                viewed={1420} 
                likes={310} 
                total_komen={45} 
                action={() => {}} 
                action_el={null} 
                photo="https://via.placeholder.com/150" 
            />
        {/each}
    </div>
</div>
	</div>
	
</section>