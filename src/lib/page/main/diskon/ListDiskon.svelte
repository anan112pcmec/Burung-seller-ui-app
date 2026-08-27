<script lang="ts">
	interface DiskonItem {
		id_diskon_produk: number;
		nama_diskon_produk: string;
		deskripsi_diskon_produk: string;
		diskon_persen_diskon_produk: number;
		berlaku_mulai_diskon_produk: string;
		berlaku_sampai_diskon_produk: string;
		status_diskon_produk: string; // Draft | Aktif | Nonaktif | Berakhir
		created_at: string;
	}

	let searchQuery = $state("");
	let selectedStatus = $state("all");
	let sortBy = $state("newest");

	// ///////////////////////////////////////////////////////////////////////
	// Hardcode dulu — ganti dengan fetch list diskon milik seller
	// ///////////////////////////////////////////////////////////////////////
	const diskonList: DiskonItem[] = [
		{
			id_diskon_produk: 1,
			nama_diskon_produk: "Gajian Sale 20%",
			deskripsi_diskon_produk: "Diskon spesial akhir bulan untuk semua kategori barang.",
			diskon_persen_diskon_produk: 20,
			berlaku_mulai_diskon_produk: "2026-08-20",
			berlaku_sampai_diskon_produk: "2026-08-31",
			status_diskon_produk: "Aktif",
			created_at: "2026-08-15T09:00:00Z"
		},
		{
			id_diskon_produk: 2,
			nama_diskon_produk: "Flash Sale Akhir Pekan",
			deskripsi_diskon_produk: "Potongan besar khusus barang kategori elektronik, kuota terbatas.",
			diskon_persen_diskon_produk: 35,
			berlaku_mulai_diskon_produk: "2026-08-22",
			berlaku_sampai_diskon_produk: "2026-08-24",
			status_diskon_produk: "Berakhir",
			created_at: "2026-08-18T14:00:00Z"
		},
		{
			id_diskon_produk: 3,
			nama_diskon_produk: "Diskon Member Baru",
			deskripsi_diskon_produk: "Diberikan untuk pembeli yang baru pertama kali checkout di toko.",
			diskon_persen_diskon_produk: 10,
			berlaku_mulai_diskon_produk: "2026-07-01",
			berlaku_sampai_diskon_produk: "2026-12-31",
			status_diskon_produk: "Aktif",
			created_at: "2026-06-25T08:30:00Z"
		},
		{
			id_diskon_produk: 4,
			nama_diskon_produk: "17-an Merdeka Sale",
			deskripsi_diskon_produk: "Perayaan kemerdekaan, diskon merata untuk semua barang.",
			diskon_persen_diskon_produk: 17,
			berlaku_mulai_diskon_produk: "2026-08-17",
			berlaku_sampai_diskon_produk: "2026-08-19",
			status_diskon_produk: "Berakhir",
			created_at: "2026-08-10T10:00:00Z"
		},
		{
			id_diskon_produk: 5,
			nama_diskon_produk: "Bundling September",
			deskripsi_diskon_produk: "Promo bundling untuk barang yang dibeli minimal 2 pcs.",
			diskon_persen_diskon_produk: 15,
			berlaku_mulai_diskon_produk: "2026-09-01",
			berlaku_sampai_diskon_produk: "2026-09-15",
			status_diskon_produk: "Draft",
			created_at: "2026-08-26T11:20:00Z"
		},
		{
			id_diskon_produk: 6,
			nama_diskon_produk: "Clearance Preloved",
			deskripsi_diskon_produk: "Diskon besar untuk menghabiskan stok barang preloved lama.",
			diskon_persen_diskon_produk: 25,
			berlaku_mulai_diskon_produk: "2026-08-10",
			berlaku_sampai_diskon_produk: "2026-08-31",
			status_diskon_produk: "Nonaktif",
			created_at: "2026-08-05T07:45:00Z"
		}
	];

	const HARI_MS = 86_400_000;
	const now = new Date();

	function formatTanggal(iso: string): string {
		return new Date(iso).toLocaleDateString("id-ID", { day: "2-digit", month: "short", year: "numeric" });
	}

	function statusClass(status: string): string {
		if (status === "Aktif") return "bg-teal-50 text-teal-700";
		if (status === "Draft") return "bg-zinc-100 text-zinc-500";
		if (status === "Berakhir") return "bg-rose-50 text-rose-600";
		return "bg-zinc-100 text-zinc-400"; // Nonaktif
	}

	function persenWaktuBerjalan(mulai: string, sampai: string): number {
		const start = new Date(mulai).getTime();
		const end = new Date(sampai).getTime();
		const total = Math.max(end - start, HARI_MS);
		const berjalan = now.getTime() - start;
		return Math.min(100, Math.max(0, Math.round((berjalan / total) * 100)));
	}

	function sisaHari(sampai: string): number {
		return Math.ceil((new Date(sampai).getTime() - now.getTime()) / HARI_MS);
	}
</script>

{#snippet CardDiskon(diskon: DiskonItem, index: number)}
	{@const persenWaktu = persenWaktuBerjalan(diskon.berlaku_mulai_diskon_produk, diskon.berlaku_sampai_diskon_produk)}
	{@const sisa = sisaHari(diskon.berlaku_sampai_diskon_produk)}
	{@const aktif = diskon.status_diskon_produk === "Aktif"}
	<div class="relative w-[23rem] flex-shrink-0 border border-zinc-200 hover:border-zinc-400 rounded-lg bg-white shadow-sm overflow-hidden flex transition-colors duration-150">

		<!-- SISI KIRI — stub kupon, persentase besar -->
		<div class="w-24 flex-shrink-0 flex flex-col items-center justify-center gap-1 border-r border-dashed border-zinc-200 bg-zinc-50/60 py-4">
			<span class="text-[9px] font-mono text-zinc-400 uppercase tracking-wider">Diskon</span>
			<span class="text-2xl font-bold font-mono tracking-tight {aktif ? 'text-teal-700' : 'text-zinc-800'}">
				{diskon.diskon_persen_diskon_produk}%
			</span>
			<span class="px-1.5 py-0.5 rounded text-[8px] font-medium uppercase tracking-wider {statusClass(diskon.status_diskon_produk)}">
				{diskon.status_diskon_produk}
			</span>
		</div>

		<!-- notch efek tiket -->
		<div class="absolute left-[5.5rem] -top-1.5 w-3 h-3 rounded-full bg-zinc-50 border border-zinc-200"></div>
		<div class="absolute left-[5.5rem] -bottom-1.5 w-3 h-3 rounded-full bg-zinc-50 border border-zinc-200"></div>

		<!-- SISI KANAN — detail -->
		<div class="flex-1 p-3 flex flex-col justify-between min-w-0">
			<div>
				<div class="flex items-center justify-between gap-1 mb-1">
					<span class="text-[9px] text-zinc-400 font-mono uppercase tracking-wider">#{String(index + 1).padStart(2, '0')}</span>
					<span class="text-[9px] text-zinc-400 font-mono">Dibuat {formatTanggal(diskon.created_at)}</span>
				</div>
				<h3 class="text-xs font-semibold text-zinc-800 leading-tight truncate">
					{diskon.nama_diskon_produk}
				</h3>
				<p class="text-[10px] text-zinc-400 line-clamp-2 mt-0.5 leading-tight">
					{diskon.deskripsi_diskon_produk}
				</p>
			</div>

			<div class="mt-2">
				<div class="flex items-center justify-between text-[9px] font-mono text-zinc-500 mb-1">
					<span>{formatTanggal(diskon.berlaku_mulai_diskon_produk)}</span>
					<span>{formatTanggal(diskon.berlaku_sampai_diskon_produk)}</span>
				</div>
				<div class="w-full h-1 bg-slate-100 rounded-full overflow-hidden">
					<div
						class="h-full rounded-full {diskon.status_diskon_produk === 'Berakhir' ? 'bg-slate-300' : 'bg-teal-600'}"
						style:width="{persenWaktu}%"
					></div>
				</div>
				<p class="text-[9px] text-zinc-400 font-light mt-1">
					{#if diskon.status_diskon_produk === "Berakhir"}
						sudah berakhir {Math.abs(sisa)} hari lalu
					{:else if diskon.status_diskon_produk === "Draft"}
						belum dipublish
					{:else if sisa <= 0}
						berakhir hari ini
					{:else}
						{sisa} hari lagi berakhir
					{/if}
				</p>
			</div>
		</div>
	</div>
{/snippet}

<section id="list-diskon" class="p-6 w-full grid grid-rows-[auto_1fr] gap-6 bg-white text-zinc-800 font-sans">
	<div>
		<h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
			Diskon Toko Kamu
		</h1>
	</div>

	<!-- Header: Search, Filter, & Sorting -->
	<div class="flex flex-col sm:flex-row items-stretch sm:items-center justify-between gap-3 w-full border-b border-zinc-200 pb-4">

		<!-- Searchbar -->
		<div class="relative flex-1 max-w-xs">
			<svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4 absolute left-3 top-1/2 -translate-y-1/2 text-zinc-400" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.75">
				<circle cx="11" cy="11" r="8"/>
				<path d="m21 21-4.3-4.3"/>
			</svg>
			<input
				type="text"
				bind:value={searchQuery}
				placeholder="Cari nama diskon..."
				class="w-full bg-white border border-zinc-300 rounded-md pl-9 pr-3 py-1.5 text-xs text-zinc-800 placeholder-zinc-400 focus:outline-none focus:border-zinc-800 transition-colors shadow-2xs"
			/>
		</div>

		<!-- Controls Group -->
		<div class="flex items-center gap-2">
			<!-- Filter Status -->
			<div class="relative">
				<select
					bind:value={selectedStatus}
					class="appearance-none bg-white border border-zinc-300 rounded-md pl-3 pr-8 py-1.5 text-xs text-zinc-700 focus:outline-none focus:border-zinc-800 cursor-pointer transition-colors shadow-2xs"
				>
					<option value="all">Semua Status</option>
					<option value="Aktif">Aktif</option>
					<option value="Draft">Draft</option>
					<option value="Nonaktif">Nonaktif</option>
					<option value="Berakhir">Berakhir</option>
				</select>
				<svg xmlns="http://www.w3.org/2000/svg" class="w-3.5 h-3.5 absolute right-2.5 top-1/2 -translate-y-1/2 text-zinc-400 pointer-events-none" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.75">
					<path d="m6 9 6 6 6-6"/>
				</svg>
			</div>

			<!-- Sorting -->
			<div class="relative">
				<select
					bind:value={sortBy}
					class="appearance-none bg-white border border-zinc-300 rounded-md pl-3 pr-8 py-1.5 text-xs text-zinc-700 focus:outline-none focus:border-zinc-800 cursor-pointer transition-colors shadow-2xs"
				>
					<option value="newest">Urutkan: Terbaru</option>
					<option value="persen-high">Persentase: Terbesar</option>
					<option value="persen-low">Persentase: Terkecil</option>
					<option value="ending-soon">Segera Berakhir</option>
				</select>
				<svg xmlns="http://www.w3.org/2000/svg" class="w-3.5 h-3.5 absolute right-2.5 top-1/2 -translate-y-1/2 text-zinc-400 pointer-events-none" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.75">
					<path d="m6 9 6 6 6-6"/>
				</svg>
			</div>
		</div>

	</div>

	<!-- Diskon Grid / Container -->
	<div class="flex flex-wrap gap-4 items-start w-full overflow-y-auto scrollbar-none pr-1">
		{#each diskonList as diskon, i (diskon.id_diskon_produk)}
			{@render CardDiskon(diskon, i)}
		{/each}
	</div>

</section>