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

	// ... (kode script eksisting kamu tetap seperti semula) ...

    // ///////////////////////////////////////////////////////////////////////
    // Mock Data Visualisasi Gudang
    // ///////////////////////////////////////////////////////////////////////

    // 1. Top 10 Kategori Stok (Kategori, bukan barang induk)
    const top10Kategori = [
        { nama: 'Elektronik & Gadget', jumlah: 4200, persen: 85 },
        { nama: 'Pakaian Pria', jumlah: 3100, persen: 62 },
        { nama: 'Aksesori Komputer', jumlah: 2800, persen: 56 },
        { nama: 'Pakaian Wanita', jumlah: 2400, persen: 48 },
        { nama: 'Peralatan Rumah', jumlah: 1900, persen: 38 },
        { nama: 'Sepatu & Sandal', jumlah: 1500, persen: 30 },
        { nama: 'Kecantikan & Perawatan', jumlah: 1200, persen: 24 },
        { nama: 'Tas & Koper', jumlah: 950, persen: 19 },
        { nama: 'Olahraga & Outdoor', jumlah: 700, persen: 14 },
        { nama: 'Otomotif & Aksesori', jumlah: 450, persen: 9 }
    ];

    // 2. Kapasitas & Okupansi
    const okupansi = {
        terpakaiM3: 1420,
        totalM3: 2000,
        persentase: 71
    };

    // 3. Status Inventaris (Health)
    const statusStok = [
        { label: 'Tersedia (Ready)', jumlah: 12450, warna: 'bg-emerald-500' },
        { label: 'Dalam Diproses', jumlah: 1820, warna: 'bg-amber-500' },
        { label: 'Retur / Karantina', jumlah: 340, warna: 'bg-rose-500' },
        { label: 'Deadstock (>90hr)', jumlah: 890, warna: 'bg-slate-400' }
    ];
</script>

<section id="details-gudang" class="w-full bg-white text-slate-950">
	<div class="col-span-1 lg:col-span-12 border border-zinc-800/20 rounded-sm p-4 sm:p-5 mt-2">
    <div class="flex items-center justify-between pb-3 mb-4 border-b border-zinc-800/10">
        <div>
            <span class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
                Analitik & Metrik Ops
            </span>
            <h3 class="text-xs sm:text-sm font-bold uppercase tracking-tight"> Visualisasi Performa Gudang</h3>
        </div>
        <span class="text-[10px] font-mono text-slate-500">Update Realtime</span>
    </div>

    <!-- Ringkasan KPI Cards (Visual 2 & 5) -->
    <div class="grid grid-cols-1 sm:grid-cols-3 gap-3 mb-5">
        <!-- Card 1: Okupansi Kapasitas -->
        <div class="p-3 bg-zinc-50 border border-zinc-200 rounded-xs">
            <span class="text-[9px] font-bold uppercase text-slate-500 tracking-wider">Okupansi Kapasitas</span>
            <div class="flex items-baseline justify-between mt-1">
                <span class="text-base font-bold font-mono text-slate-900">{okupansi.persentase}%</span>
                <span class="text-[10px] text-zinc-500">{okupansi.terpakaiM3} / {okupansi.totalM3} m³</span>
            </div>
            <div class="w-full bg-zinc-200 h-1.5 rounded-full mt-2 overflow-hidden">
                <div class="bg-slate-900 h-full" style="width: {okupansi.persentase}%"></div>
            </div>
        </div>

        <!-- Card 2: Estimasi Total Nilai Stok -->
        <div class="p-3 bg-zinc-50 border border-zinc-200 rounded-xs">
            <span class="text-[9px] font-bold uppercase text-slate-500 tracking-wider">Total Valuation Stok</span>
            <div class="mt-1">
                <span class="text-base font-bold font-mono text-slate-900">Rp 1.482.500.000</span>
            </div>
            <span class="text-[9px] text-teal-700 font-medium">+4.2% dari bulan lalu</span>
        </div>

        <!-- Card 3: Arus Outbound 30 Hari -->
        <div class="p-3 bg-zinc-50 border border-zinc-200 rounded-xs">
            <span class="text-[9px] font-bold uppercase text-slate-500 tracking-wider">Throughput Outbound</span>
            <div class="mt-1">
                <span class="text-base font-bold font-mono text-slate-900">3.420 Item</span>
            </div>
            <span class="text-[9px] text-slate-500">Fast-Moving Ratio: <strong class="text-slate-800">68%</strong></span>
        </div>
    </div>

    <div class="grid grid-cols-1 lg:grid-cols-12 gap-5">
        <!-- VISUAL 1: Top 10 Kategori Barang (Stok Masuk & Idle) -->
        <div class="lg:col-span-7 flex flex-col gap-2">
            <span class="text-[10px] font-bold text-slate-700 uppercase tracking-wider">
                Top 10 Kategori Barang (Volume Stok Masuk & Idle)
            </span>
            <div class="flex flex-col gap-2 mt-1 bg-white p-3 border border-zinc-200 rounded-xs">
                {#each top10Kategori as item}
                    <div class="flex flex-col gap-0.5">
                        <div class="flex justify-between text-[10px] font-sans">
                            <span class="text-slate-700 truncate max-w-[200px] sm:max-w-[280px]">{item.nama}</span>
                            <span class="font-mono text-slate-900 font-medium">{item.jumlah.toLocaleString('id-ID')} unit</span>
                        </div>
                        <div class="w-full bg-zinc-100 h-2 rounded-2xs overflow-hidden">
                            <div class="bg-slate-900 h-full rounded-2xs transition-all duration-300" style="width: {item.persen}%"></div>
                        </div>
                    </div>
                {/each}
            </div>
        </div>

        <!-- VISUAL 3 & 4: Status Health Stok & Arus Masuk/Keluar -->
        <div class="lg:col-span-5 flex flex-col gap-4">
            <!-- VISUAL 4: Kesehatan Stok -->
            <div class="flex flex-col gap-2">
                <span class="text-[10px] font-bold text-slate-700 uppercase tracking-wider">
                    Distribusi Status Stok
                </span>
                <div class="bg-white p-3 border border-zinc-200 rounded-xs flex flex-col gap-3">
                    <div class="flex h-3 w-full rounded-2xs overflow-hidden">
                        <div class="bg-emerald-500 h-full" style="width: 80%" title="Tersedia"></div>
                        <div class="bg-amber-500 h-full" style="width: 11%" title="Diproses"></div>
                        <div class="bg-slate-400 h-full" style="width: 6%" title="Deadstock"></div>
                        <div class="bg-rose-500 h-full" style="width: 3%" title="Retur"></div>
                    </div>
                    <div class="grid grid-cols-2 gap-2">
                        {#each statusStok as st}
                            <div class="flex items-center gap-1.5 text-[10px]">
                                <span class="w-2 h-2 rounded-full {st.warna} shrink-0"></span>
                                <span class="text-slate-600 truncate">{st.label}:</span>
                                <span class="font-mono font-bold text-slate-900">{st.jumlah.toLocaleString('id-ID')}</span>
                            </div>
                        {/each}
                    </div>
                </div>
            </div>

            <!-- VISUAL 3: Arus Inbound vs Outbound Ringkas -->
            <div class="flex flex-col gap-2">
                <span class="text-[10px] font-bold text-slate-700 uppercase tracking-wider">
                    Rasio Barang Masuk vs Barang Keluar (Bulan Ini)
                </span>
                <div class="bg-white p-3 border border-zinc-200 rounded-xs flex flex-col gap-2">
                    <div class="flex justify-between items-center text-[10px]">
                        <span class="text-slate-600">Barang Masuk (Inbound)</span>
                        <span class="font-mono font-bold text-teal-700">+4.120 unit</span>
                    </div>
                    <div class="w-full bg-zinc-100 h-2 rounded-2xs overflow-hidden">
                        <div class="bg-teal-600 h-full" style="width: 55%"></div>
                    </div>

                    <div class="flex justify-between items-center text-[10px] mt-1">
                        <span class="text-slate-600">Barang Keluar (Outbound)</span>
                        <span class="font-mono font-bold text-slate-900">-3.420 unit</span>
                    </div>
                    <div class="w-full bg-zinc-100 h-2 rounded-2xs overflow-hidden">
                        <div class="bg-slate-800 h-full" style="width: 45%"></div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
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