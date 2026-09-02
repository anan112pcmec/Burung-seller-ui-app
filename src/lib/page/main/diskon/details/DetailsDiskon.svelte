<script lang="ts">
  import CardProduk from "../../general/CardProduk.svelte";

	interface DiskonProduk {
		id_diskon_produk: number;
		nama_diskon_produk: string;
		deskripsi_diskon_produk: string;
		diskon_persen_diskon_produk: number;
		berlaku_mulai_diskon_produk: string;
		berlaku_sampai_diskon_produk: string;
		status_diskon_produk: "Draft" | "Aktif" | "Selesai";
		created_at: string;
	}

	interface BarangDiDiskon {
		id_barang_di_diskon: number;
		id_barang_induk_barang_di_diskon: number;
		nama_barang: string; // asumsi — belum ada di struct BarangInduk yang dikasih
		harga_barang: number; // asumsi
		nama_kategori_barang: string; // asumsi dari relasi KategoriBarang
		status_barang_di_diskon: string; // default "Waiting", nilai lain belum dikonfirmasi
	}

	// ///////////////////////////////////////////////////////////////////////
	// Hardcode dulu — ganti dengan fetch by :id_diskon_produk
	// ///////////////////////////////////////////////////////////////////////
	const diskon: DiskonProduk = {
		id_diskon_produk: 1,
		nama_diskon_produk: "Gajian Sale 20%",
		deskripsi_diskon_produk: "Diskon spesial akhir bulan untuk semua kategori barang.",
		diskon_persen_diskon_produk: 20,
		berlaku_mulai_diskon_produk: "2026-08-20",
		berlaku_sampai_diskon_produk: "2026-08-31",
		status_diskon_produk: "Draft",
		created_at: "2026-08-15T09:00:00Z"
	};

	const barangList: BarangDiDiskon[] = [
		{ id_barang_di_diskon: 1, id_barang_induk_barang_di_diskon: 101, nama_barang: "Kaos Polos Katun Combed", harga_barang: 85000, nama_kategori_barang: "Pakaian Pria", status_barang_di_diskon: "Waiting" },
		{ id_barang_di_diskon: 2, id_barang_induk_barang_di_diskon: 102, nama_barang: "Celana Chino Slimfit", harga_barang: 165000, nama_kategori_barang: "Pakaian Pria", status_barang_di_diskon: "Waiting" },
		{ id_barang_di_diskon: 3, id_barang_induk_barang_di_diskon: 103, nama_barang: "Jaket Hoodie Fleece", harga_barang: 210000, nama_kategori_barang: "Outerwear", status_barang_di_diskon: "Waiting" },
		{ id_barang_di_diskon: 4, id_barang_induk_barang_di_diskon: 104, nama_barang: "Sepatu Sneakers Canvas", harga_barang: 245000, nama_kategori_barang: "Sepatu", status_barang_di_diskon: "Waiting" }
	];

	const bisaDiedit = $derived(diskon.status_diskon_produk === "Draft");

	function formatTanggal(iso: string): string {
		return new Date(iso).toLocaleDateString("id-ID", { day: "2-digit", month: "short", year: "numeric" });
	}

	function formatRupiah(n: number): string {
		return n.toLocaleString("id-ID", { style: "currency", currency: "IDR", minimumFractionDigits: 0 });
	}

	function hargaSetelahDiskon(harga: number): number {
		return Math.round(harga - (harga * diskon.diskon_persen_diskon_produk) / 100);
	}

	function statusDiskonClass(status: string): string {
		if (status === "Aktif") return "bg-teal-50 text-teal-700";
		if (status === "Draft") return "bg-zinc-100 text-zinc-500";
		return "bg-slate-100 text-slate-500"; // Selesai
	}

	function statusBarangClass(status: string): string {
		if (status === "Waiting") return "bg-amber-50 text-amber-700";
		return "bg-zinc-100 text-zinc-500";
	}

	function handleHapusBarang(id: number) {
		// TODO: hubungkan ke DELETE barang_di_diskon/:id — hanya boleh jika bisaDiedit
	}

	function handleTambahBarang() {
		// TODO: buka modal/pilih barang dari master BarangInduk seller
	}
</script>

<section id="details-diskon" class="w-full bg-white p-4 sm:p-6 lg:p-8 text-slate-950">

	<!-- BREADCRUMB / BACK -->
	<button class="flex items-center gap-1.5 text-[10px] sm:text-[11px] font-sans uppercase tracking-wider text-slate-500 hover:text-slate-950 mb-5">
		<svg xmlns="http://www.w3.org/2000/svg" class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
			<path d="m15 18-6-6 6-6"/>
		</svg>
		Kembali ke Diskon Toko
	</button>

	{#if !bisaDiedit}
		<div class="flex items-center gap-2 border border-zinc-200 bg-zinc-50 rounded-sm px-3.5 py-2.5 mb-5">
			<span class="w-1.5 h-1.5 rounded-full bg-zinc-400 flex-shrink-0"></span>
			<p class="text-[10px] sm:text-[11px] text-zinc-500 font-light">
				Diskon berstatus <span class="font-medium text-zinc-700">{diskon.status_diskon_produk}</span> — tidak bisa diubah lagi. Hanya diskon berstatus Draft yang bisa diedit.
			</p>
		</div>
	{/if}

	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- RINGKASAN DISKON — 28% / gap / 68% -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="grid grid-cols-1 md:grid-cols-[28%_4%_68%] w-full border border-zinc-800/20 rounded-sm">

		<!-- KIRI — persentase & status -->
		<div class="p-4 sm:p-5 flex flex-col justify-between gap-4">
			<div>
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-sans">
					Persentase Diskon
				</span>
				<div class="mt-2 flex items-end gap-2">
					<span class="text-3xl sm:text-4xl font-bold font-sans tracking-tight text-teal-700">
						{diskon.diskon_persen_diskon_produk}%
					</span>
					<span class="px-1.5 py-0.5 rounded text-[8px] font-medium uppercase tracking-wider mb-1.5 {statusDiskonClass(diskon.status_diskon_produk)}">
						{diskon.status_diskon_produk}
					</span>
				</div>
			</div>

			<button
				disabled={!bisaDiedit}
				class="text-[9px] sm:text-[10px] font-medium uppercase tracking-wider border rounded-xs px-3 py-1.5 self-start transition-colors {bisaDiedit
					? 'border-slate-950 text-slate-950 hover:bg-slate-950 hover:text-white'
					: 'border-zinc-200 text-zinc-300 cursor-not-allowed'}"
			>
				Edit Diskon
			</button>
		</div>

		<!-- GAP — divider vertikal -->
		<div class="hidden md:flex justify-center">
			<div class="w-px h-full bg-zinc-800/10"></div>
		</div>

		<!-- KANAN — info & stat -->
		<div class="p-4 sm:p-5 flex flex-col justify-between gap-4 border-t md:border-t-0 border-zinc-800/10">
			<div>
				<h2 class="text-sm sm:text-base font-semibold text-zinc-800">{diskon.nama_diskon_produk}</h2>
				<p class="text-[10px] sm:text-[11px] text-zinc-400 font-light mt-1 leading-relaxed">
					{diskon.deskripsi_diskon_produk}
				</p>
			</div>

			<div class="grid grid-cols-3 gap-3">
				<div>
					<span class="text-[8px] font-bold tracking-wider text-slate-950/40 uppercase font-sans">Berlaku Mulai</span>
					<p class="text-[11px] font-sans font-medium mt-0.5">{formatTanggal(diskon.berlaku_mulai_diskon_produk)}</p>
				</div>
				<div>
					<span class="text-[8px] font-bold tracking-wider text-slate-950/40 uppercase font-sans">Berlaku Sampai</span>
					<p class="text-[11px] font-sans font-medium mt-0.5">{formatTanggal(diskon.berlaku_sampai_diskon_produk)}</p>
				</div>
				<div>
					<span class="text-[8px] font-bold tracking-wider text-slate-950/40 uppercase font-sans">Total Barang</span>
					<p class="text-[11px] font-sans font-bold mt-0.5 text-teal-700">{barangList.length} item</p>
				</div>
			</div>
		</div>
	</div>

	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- LIST BARANG DALAM DISKON -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="mt-6">
		<div class="flex items-center justify-between mb-3">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-sans">
				Barang Dalam Diskon Ini
			</span>
			<button
				onclick={handleTambahBarang}
				disabled={!bisaDiedit}
				class="text-[9px] sm:text-[10px] font-medium uppercase tracking-wider border rounded-xs px-3 py-1.5 transition-colors {bisaDiedit
					? 'border-slate-950 text-slate-950 hover:bg-slate-950 hover:text-white'
					: 'border-zinc-200 text-zinc-300 cursor-not-allowed'}"
			>
				+ Tambah Barang
			</button>
		</div>

		<div class="flex flex-wrap gap-4">
			{#each barangList as barang (barang.id_barang_di_diskon)}
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

			{#if barangList.length === 0}
				<p class="text-[10px] text-zinc-400 font-sans py-8 text-center">Belum ada barang di diskon ini.</p>
			{/if}
		</div>
	</div>

</section>