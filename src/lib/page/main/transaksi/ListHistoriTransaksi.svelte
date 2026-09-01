<script lang="ts">
	interface TransaksiCardData {
		kode_order_sistem: string;
		status: string; // status_transaksi enum
		kuantitas_barang: number;
		total: number; // Total (kotor)
		jenis_pengiriman: string;
		kendaraan_pengiriman: string;
		kode_resi_ekspedisi: string | null;
		provider_pembayaran: string; // dari relasi Pembayaran
		created_at: string;
	}

	// ///////////////////////////////////////////////////////////////////////
	// Hardcode dulu — ganti dengan fetch list transaksi seller (paginated)
	// ///////////////////////////////////////////////////////////////////////
	const transaksiList: TransaksiCardData[] = [
		{
			kode_order_sistem: 'ORD-20260827-0142',
			status: 'Dikirim',
			kuantitas_barang: 3,
			total: 458_000,
			jenis_pengiriman: 'Reguler',
			kendaraan_pengiriman: 'Motor',
			kode_resi_ekspedisi: 'JX0293841123',
			provider_pembayaran: 'QRIS',
			created_at: '2026-08-26T14:32:00Z'
		},
		{
			kode_order_sistem: 'ORD-20260825-0098',
			status: 'Dibatalkan',
			kuantitas_barang: 1,
			total: 129_000,
			jenis_pengiriman: 'Instan',
			kendaraan_pengiriman: 'Motor',
			kode_resi_ekspedisi: null,
			provider_pembayaran: 'Virtual Account',
			created_at: '2026-08-25T09:10:00Z'
		}
	];

	const ulang = transaksiList.length;

	function formatRupiah(n: number): string {
		return `Rp${n.toLocaleString('id-ID')}`;
	}

	function formatTanggal(iso: string): string {
		return new Date(iso).toLocaleDateString('id-ID', { day: '2-digit', month: 'short', year: 'numeric' });
	}

	function statusClass(status: string): string {
		if (status === 'Dibatalkan') return 'bg-rose-50 text-rose-600';
		if (status === 'Selesai') return 'bg-teal-50 text-teal-700';
		return 'bg-zinc-100 text-zinc-600';
	}
</script>

{#snippet TransaksiCard(i: number)}
	{@const trx = transaksiList[i]}
	<div class="relative w-72 flex-shrink-0 border border-zinc-200 hover:border-zinc-400 rounded-lg bg-white shadow-sm overflow-hidden flex flex-col transition-colors duration-150">

		<!-- HEADER: kode order + status -->
		<div class="p-3 border-b border-dashed border-zinc-200">
            <div class="grid grid-cols-[10%_2%_56%_2%_30%]">
                <div class="flex items-center justify-center"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="30" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-receipt-text-icon lucide-receipt-text"><path d="M13 16H8"/><path d="M14 8H8"/><path d="M16 12H8"/><path d="M4 3a1 1 0 0 1 1-1 1.3 1.3 0 0 1 .7.2l.933.6a1.3 1.3 0 0 0 1.4 0l.934-.6a1.3 1.3 0 0 1 1.4 0l.933.6a1.3 1.3 0 0 0 1.4 0l.933-.6a1.3 1.3 0 0 1 1.4 0l.934.6a1.3 1.3 0 0 0 1.4 0l.933-.6A1.3 1.3 0 0 1 19 2a1 1 0 0 1 1 1v18a1 1 0 0 1-1 1 1.3 1.3 0 0 1-.7-.2l-.933-.6a1.3 1.3 0 0 0-1.4 0l-.934.6a1.3 1.3 0 0 1-1.4 0l-.933-.6a1.3 1.3 0 0 0-1.4 0l-.933.6a1.3 1.3 0 0 1-1.4 0l-.934-.6a1.3 1.3 0 0 0-1.4 0l-.933.6a1.3 1.3 0 0 1-.7.2 1 1 0 0 1-1-1z"/></svg></div>
                <div></div>
                <div class="p-3 rounded-lg">
                    <div class="flex flex-col items-start">
                        <span class="text-[9px] text-zinc-400 font-mono uppercase tracking-wider">Kode Order</span>
                        <p class="text-xs font-bold font-mono text-zinc-800 truncate">{trx.kode_order_sistem}</p>
                    </div>
                </div>
                <div></div>
                <div class="flex items-center"><span class=" px-2 py-0.5 rounded text-[9px] font-medium uppercase tracking-wider flex-shrink-0 ml-2 {statusClass(trx.status)}">
				{trx.status}
			</span></div>
            </div>
		</div>

		<!-- notch efek tiket -->
		<div class="absolute -left-1.5 top-[3.1rem] w-3 h-3 rounded-full bg-zinc-50 border border-zinc-200"></div>
		<div class="absolute -right-1.5 top-[3.1rem] w-3 h-3 rounded-full bg-zinc-50 border border-zinc-200"></div>

		<!-- RINGKASAN -->
		<div class="p-3 flex-1 flex flex-col justify-center gap-2">
			<div class="flex items-center justify-between text-[11px]">
				<span class="text-zinc-500">Jumlah Barang</span>
				<span class="font-mono font-semibold text-zinc-800">{trx.kuantitas_barang} unit</span>
			</div>
			<div class="flex items-center justify-between text-[11px]">
				<span class="text-zinc-500">Pengiriman</span>
				<span class="font-mono font-semibold text-zinc-800">{trx.jenis_pengiriman} · {trx.kendaraan_pengiriman}</span>
			</div>
			<div class="flex items-center justify-between text-[11px]">
				<span class="text-zinc-500">Resi</span>
				<span class="font-mono font-semibold text-zinc-800">
					{trx.kode_resi_ekspedisi ?? 'Belum tersedia'}
				</span>
			</div>
			<div class="flex items-center justify-between pt-1 mt-1 border-t border-zinc-100">
				<span class="text-[11px] text-zinc-500">Total Tagihan</span>
				<span class="font-mono font-bold text-zinc-900 text-sm">{formatRupiah(trx.total)}</span>
			</div>
		</div>

		<!-- FOOTER: metode bayar + tanggal -->
		<div class="p-3 border-t border-dashed border-zinc-200 flex items-center justify-between bg-zinc-50/60">
			<div class="flex items-center gap-1 text-[10px] text-zinc-500">
				<svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-zinc-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
					<rect width="20" height="14" x="2" y="5" rx="2"/>
					<path d="M2 10h20"/>
				</svg>
				<span>{trx.provider_pembayaran}</span>
			</div>
			<span class="text-[9px] font-mono text-zinc-400">{formatTanggal(trx.created_at)}</span>
		</div>
	</div>
{/snippet}

<section id="list-transaksi" class="mb-10 mx-8 space-y-4 font-sans text-zinc-800">
  
  <!-- Header: Title & Metrik Transaksi -->
  <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-3 border-b border-zinc-200/80 pb-3">
    
    <!-- Left Side: Icon & Title -->
    <div class="flex items-center gap-2.5">
      <div class="p-1.5 bg-zinc-100 rounded-md text-zinc-700">
        <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4 stroke-[1.75]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round">
          <path d="M12 3a9.75 9.75 0 0 1 6.74 2.74"/>
          <path d="M18.74 5.74 21 8"/>
          <path d="M21 8V3"/>
          <path d="M7.5 19.794c-6-3.464-6-12.124 0-15.588"/>
          <path d="M7.5 4.206A9 9 0 0 1 12 3"/>
          <path d="M12 7v5l4 2"/>
          <path d="M14 20.775A9 9 0 0 1 12 21"/>
          <path d="M19 17.656a9 9 0 0 1-1.5 1.456"/>
          <path d="M21 12a9 9 0 0 1-.228 2"/>
          <path d="M21 8h-5"/>
        </svg>
      </div>
      <div>
        <h2 class="text-xs font-bold text-zinc-900 tracking-tight uppercase">Histori Transaksi</h2>
        <p class="text-[10px] text-zinc-400">Ringkasan transaksi dan log aktivitas penjualan</p>
      </div>
    </div>

    <!-- Right Side: Total Stats -->
    <div class="flex items-center gap-4 text-xs font-mono">
      <div class="flex flex-col sm:items-end">
        <span class="text-[9px] text-zinc-400 font-sans font-medium uppercase tracking-wider">Total Pesanan</span>
        <span class="font-bold text-zinc-900">1,248 <span class="text-[10px] text-zinc-400 font-normal">trx</span></span>
      </div>
      <div class="h-6 w-px bg-zinc-200"></div>
      <div class="flex flex-col sm:items-end">
        <span class="text-[9px] text-zinc-400 font-sans font-medium uppercase tracking-wider">Total Volume</span>
        <span class="font-bold text-zinc-900">Rp 48.5M</span>
      </div>
      <div class="h-6 w-px bg-zinc-200"></div>
      <div class="flex flex-col sm:items-end">
        <span class="text-[9px] text-zinc-400 font-sans font-medium uppercase tracking-wider">Tingkat Sukses</span>
        <span class="font-bold text-zinc-900">98.4%</span>
      </div>
    </div>

  </div>

  <!-- Horizontal Scrollable Cards -->
  <div class="flex items-start gap-4 overflow-x-auto scrollbar-none pb-2">
    {#each Array(ulang) as _, i (i)}
      {@render TransaksiCard(i)}
    {/each}
  </div>

</section>