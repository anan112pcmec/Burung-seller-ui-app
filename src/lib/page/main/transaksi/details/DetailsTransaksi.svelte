<script lang="ts">
	import ListTransaksi from '../ListTransaksi.svelte';

	// ///////////////////////////////////////////////////////////////////////
	// Types
	// ///////////////////////////////////////////////////////////////////////

	interface DetailTransaksi {
		// Transaksi
		idTransaksi: number;
		kodeOrderSistem: string;
		status: string;
		dibatalkanOleh: string | null;
		catatan: string;
		kuantitasBarang: number;
		kendaraanPengiriman: string;
		jenisPengiriman: string;
		jarakTempuh: string;
		beratTotalKg: number;
		isEkspedisi: boolean;
		kodeResiEkspedisi: string | null;
		total: number;
		sellerPaid: number;
		kurirPaid: number;
		sistemPaid: number;
		ekspedisiPaid: number;
		reviewed: boolean;
		createdAt: string;

		// BarangInduk (relasi)
		barangInduk: {
			namaBarang: string;
			jenisBarang: string;
			deskripsi: string;
		};

		// KategoriBarang (relasi — varian spesifik yang dibeli)
		kategoriBarang: {
			nama: string;
			warna: string;
			harga: number;
			sku: string;
			beratGram: number;
		};

		// Pembayaran (relasi)
		pembayaran: {
			provider: string;
			paymentType: string;
			kodeTransaksiPg: string;
			paidAt: string;
		};

		// Alamat (ringkas, bukan struct penuh — cukup buat ditampilkan)
		alamatPengguna: string;
		alamatGudang: string;
	}

	// ///////////////////////////////////////////////////////////////////////
	// Mock data — ganti dengan fetch detail transaksi by id_transaksi,
	// termasuk join ke BarangInduk, KategoriBarang, Pembayaran
	// ///////////////////////////////////////////////////////////////////////

	const trx: DetailTransaksi = {
		idTransaksi: 4821,
		kodeOrderSistem: 'ORD-20260827-0142',
		status: 'Dikirim',
		dibatalkanOleh: null,
		catatan: 'Tolong dibungkus rapi, ini mau dikasih ke orang tua.',
		kuantitasBarang: 2,
		kendaraanPengiriman: 'Motor',
		jenisPengiriman: 'Reguler',
		jarakTempuh: '8.4 km',
		beratTotalKg: 1,
		isEkspedisi: true,
		kodeResiEkspedisi: 'JX0293841123',
		total: 458_000,
		sellerPaid: 392_000,
		kurirPaid: 18_000,
		sistemPaid: 38_000,
		ekspedisiPaid: 10_000,
		reviewed: false,
		createdAt: '2026-08-26T14:32:00Z',
		barangInduk: {
			namaBarang: 'Kemeja Flanel Kotak-kotak',
			jenisBarang: 'Fashion Pria',
			deskripsi: 'Bahan flanel tebal, cocok dipakai musim hujan, tersedia beberapa varian warna kotak.'
		},
		kategoriBarang: {
			nama: 'Kemeja Flanel Kotak-kotak - Merah, size L',
			warna: 'Merah',
			harga: 159_000,
			sku: 'KFK-MR-L',
			beratGram: 500
		},
		pembayaran: {
			provider: 'QRIS',
			paymentType: 'e_wallet',
			kodeTransaksiPg: 'PG-QR-88213741',
			paidAt: '2026-08-26T14:35:12Z'
		},
		alamatPengguna: 'Jl. Kenanga No. 12, RT03/RW05, Bekasi Timur, Jawa Barat 17113',
		alamatGudang: 'Gudang Utama - Jl. Industri Raya No. 8, Cikarang, Jawa Barat 17530'
	};

	// ///////////////////////////////////////////////////////////////////////
	// Helpers
	// ///////////////////////////////////////////////////////////////////////

	function formatRupiah(n: number): string {
		return `Rp${n.toLocaleString('id-ID')}`;
	}

	function formatTanggalLengkap(iso: string): string {
		return new Date(iso).toLocaleDateString('id-ID', {
			day: '2-digit',
			month: 'long',
			year: 'numeric',
			hour: '2-digit',
			minute: '2-digit'
		});
	}

	function statusClass(status: string): string {
		if (status === 'Dibatalkan') return 'bg-rose-50 text-rose-600 border-rose-200';
		if (status === 'Selesai') return 'bg-teal-50 text-teal-700 border-teal-200';
		return 'bg-zinc-100 text-zinc-600 border-zinc-200';
	}

	let hargaSatuan = $derived(trx.kategoriBarang.harga);
	let subtotalBarang = $derived(hargaSatuan * trx.kuantitasBarang);
	let biayaLainnya = $derived(trx.total - subtotalBarang);

	// ///////////////////////////////////////////////////////////////////////
	// Aksi — bagikan & unduh sebagai PDF
	// ///////////////////////////////////////////////////////////////////////

	let statusBagikan = $state<'idle' | 'disalin'>('idle');

	async function bagikanTransaksi() {
		const shareUrl = typeof window !== 'undefined' ? window.location.href : '';
		const shareData = {
			title: `Transaksi ${trx.kodeOrderSistem}`,
			text: `Detail transaksi ${trx.kodeOrderSistem} - ${formatRupiah(trx.total)}`,
			url: shareUrl
		};

		// TODO: navigator.share cuma jalan di konteks HTTPS & browser yang
		// mendukung Web Share API (kebanyakan mobile). Fallback-nya copy link.
		if (typeof navigator !== 'undefined' && navigator.share) {
			try {
				await navigator.share(shareData);
				return;
			} catch {
				// pengguna membatalkan share sheet, gak perlu ditangani khusus
			}
		}

		if (typeof navigator !== 'undefined' && navigator.clipboard) {
			await navigator.clipboard.writeText(shareUrl);
			statusBagikan = 'disalin';
			setTimeout(() => (statusBagikan = 'idle'), 2000);
		}
	}

	function unduhSebagaiPdf() {
		// TODO: pendekatan paling sederhana adalah window.print() dengan CSS
		// khusus @media print (sembunyikan navigasi, rapikan layout invoice).
		// Untuk hasil PDF yang lebih presisi/konsisten lintas browser, generate
		// PDF di backend (mis. dari data transaksi yang sama) lalu sediakan
		// link download langsung.
		if (typeof window !== 'undefined') window.print();
	}
</script>

<section id="details-transaksi" class="w-full bg-white p-4 sm:p-6 lg:p-8 text-slate-950">
	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- HEADER -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="flex flex-col sm:flex-row sm:items-start sm:justify-between gap-4 pb-6 mb-6 border-b border-zinc-800/10">
		<div>
			<span class="text-[9px] sm:text-[10px] font-mono text-slate-400 uppercase tracking-wider">
				Transaksi / {trx.kodeOrderSistem}
			</span>
			<h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
				Detail Transaksi
			</h1>
			<div class="flex items-center gap-2 mt-2">
				<span class="px-2 py-0.5 rounded text-[10px] font-medium uppercase tracking-wider border {statusClass(trx.status)}">
					{trx.status}
				</span>
				<span class="text-[10px] font-mono text-slate-400">{formatTanggalLengkap(trx.createdAt)}</span>
			</div>
		</div>

		<div class="flex items-center gap-2 flex-shrink-0">
			<button
				type="button"
				onclick={bagikanTransaksi}
				class="px-3 py-2 border border-zinc-300 rounded-sm text-[10px] font-bold uppercase tracking-wider text-slate-700 hover:bg-zinc-50 transition-colors"
			>
				{statusBagikan === 'disalin' ? 'Link Disalin' : 'Bagikan'}
			</button>
			<button
				type="button"
				onclick={unduhSebagaiPdf}
				class="px-3 py-2 bg-slate-950 text-white rounded-sm text-[10px] font-bold uppercase tracking-wider hover:bg-slate-800 transition-colors"
			>
				Unduh sebagai PDF
			</button>
		</div>
	</div>

	<div class="grid grid-cols-1 lg:grid-cols-12 gap-5">
		<!-- ///////////////////////////////////////////////////////////// -->
		<!-- KOLOM UTAMA -->
		<!-- ///////////////////////////////////////////////////////////// -->
		<div class="lg:col-span-8 flex flex-col gap-4">

			<!-- BARANG YANG DIBELI -->
			<div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					Barang Dibeli
				</span>

				<div class="mt-3 flex items-start justify-between gap-3">
					<div class="min-w-0">
						<span class="text-[9px] text-zinc-400 font-mono uppercase tracking-wider">{trx.barangInduk.jenisBarang}</span>
						<h3 class="text-sm font-semibold text-zinc-800 leading-tight mt-0.5">{trx.barangInduk.namaBarang}</h3>
						<p class="text-[11px] text-zinc-500 mt-1">
							Varian: <span class="font-medium text-zinc-700">{trx.kategoriBarang.nama}</span>
						</p>
						<p class="text-[10px] text-zinc-400 mt-2 leading-relaxed max-w-md">{trx.barangInduk.deskripsi}</p>
					</div>
				</div>

				<div class="mt-4 pt-4 border-t border-dashed border-zinc-200 grid grid-cols-2 sm:grid-cols-4 gap-3 text-[10px]">
					<div>
						<p class="text-zinc-400 uppercase tracking-wider">SKU</p>
						<p class="font-mono font-semibold text-zinc-700 mt-0.5">{trx.kategoriBarang.sku}</p>
					</div>
					<div>
						<p class="text-zinc-400 uppercase tracking-wider">Warna</p>
						<p class="font-mono font-semibold text-zinc-700 mt-0.5">{trx.kategoriBarang.warna}</p>
					</div>
					<div>
						<p class="text-zinc-400 uppercase tracking-wider">Berat / Unit</p>
						<p class="font-mono font-semibold text-zinc-700 mt-0.5">{trx.kategoriBarang.beratGram}g</p>
					</div>
					<div>
						<p class="text-zinc-400 uppercase tracking-wider">Kuantitas</p>
						<p class="font-mono font-semibold text-zinc-700 mt-0.5">{trx.kuantitasBarang} unit</p>
					</div>
				</div>

				<div class="mt-4 pt-4 border-t border-zinc-200 flex items-center justify-between">
					<span class="text-[11px] text-zinc-500">{formatRupiah(hargaSatuan)} × {trx.kuantitasBarang}</span>
					<span class="text-sm font-bold font-mono text-zinc-900">{formatRupiah(subtotalBarang)}</span>
				</div>
			</div>

			<!-- INFO PENGIRIMAN -->
			<div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					Pengiriman
				</span>

				<div class="mt-3 grid grid-cols-2 sm:grid-cols-4 gap-3 text-[10px]">
					<div>
						<p class="text-zinc-400 uppercase tracking-wider">Jenis</p>
						<p class="font-mono font-semibold text-zinc-700 mt-0.5">{trx.jenisPengiriman}</p>
					</div>
					<div>
						<p class="text-zinc-400 uppercase tracking-wider">Kendaraan</p>
						<p class="font-mono font-semibold text-zinc-700 mt-0.5">{trx.kendaraanPengiriman}</p>
					</div>
					<div>
						<p class="text-zinc-400 uppercase tracking-wider">Jarak Tempuh</p>
						<p class="font-mono font-semibold text-zinc-700 mt-0.5">{trx.jarakTempuh}</p>
					</div>
					<div>
						<p class="text-zinc-400 uppercase tracking-wider">Berat Total</p>
						<p class="font-mono font-semibold text-zinc-700 mt-0.5">{trx.beratTotalKg} kg</p>
					</div>
				</div>

				<div class="mt-4 pt-4 border-t border-dashed border-zinc-200 grid grid-cols-1 sm:grid-cols-2 gap-4">
					<div>
						<p class="text-[10px] text-zinc-400 uppercase tracking-wider">Alamat Tujuan</p>
						<p class="text-[11px] text-zinc-700 mt-1 leading-relaxed">{trx.alamatPengguna}</p>
					</div>
					<div>
						<p class="text-[10px] text-zinc-400 uppercase tracking-wider">Alamat Asal (Gudang)</p>
						<p class="text-[11px] text-zinc-700 mt-1 leading-relaxed">{trx.alamatGudang}</p>
					</div>
				</div>

				{#if trx.kodeResiEkspedisi}
					<div class="mt-4 pt-4 border-t border-zinc-200 flex items-center justify-between">
						<span class="text-[11px] text-zinc-500">Nomor Resi</span>
						<span class="text-xs font-bold font-mono text-zinc-900">{trx.kodeResiEkspedisi}</span>
					</div>
				{/if}
			</div>

			{#if trx.catatan}
				<div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5">
					<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
						Catatan Pembeli
					</span>
					<p class="text-[11px] text-zinc-600 mt-2 leading-relaxed italic">"{trx.catatan}"</p>
				</div>
			{/if}
		</div>

		<!-- ///////////////////////////////////////////////////////////// -->
		<!-- SIDEBAR — pembayaran & rincian biaya -->
		<!-- ///////////////////////////////////////////////////////////// -->
		<div class="lg:col-span-4 flex flex-col gap-4">
			<div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					Pembayaran
				</span>
				<div class="mt-3 flex flex-col gap-2 text-[11px]">
					<div class="flex items-center justify-between">
						<span class="text-zinc-500">Provider</span>
						<span class="font-medium text-zinc-800">{trx.pembayaran.provider}</span>
					</div>
					<div class="flex items-center justify-between">
						<span class="text-zinc-500">Metode</span>
						<span class="font-medium text-zinc-800">{trx.pembayaran.paymentType}</span>
					</div>
					<div class="flex items-center justify-between">
						<span class="text-zinc-500">Kode Transaksi PG</span>
						<span class="font-mono text-zinc-800 text-[10px]">{trx.pembayaran.kodeTransaksiPg}</span>
					</div>
					<div class="flex items-center justify-between">
						<span class="text-zinc-500">Dibayar Pada</span>
						<span class="font-medium text-zinc-800 text-[10px]">{formatTanggalLengkap(trx.pembayaran.paidAt)}</span>
					</div>
				</div>
			</div>

			<div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					Rincian Tagihan
				</span>
				<div class="mt-3 flex flex-col gap-2 text-[11px]">
					<div class="flex items-center justify-between">
						<span class="text-zinc-500">Subtotal Barang</span>
						<span class="font-mono text-zinc-700">{formatRupiah(subtotalBarang)}</span>
					</div>
					<div class="flex items-center justify-between">
						<span class="text-zinc-500">Ongkir &amp; Biaya Lain</span>
						<span class="font-mono text-zinc-700">{formatRupiah(biayaLainnya)}</span>
					</div>
					<div class="flex items-center justify-between pt-2 mt-1 border-t border-zinc-200">
						<span class="font-bold text-zinc-800">Total Dibayar Pembeli</span>
						<span class="font-mono font-bold text-zinc-900">{formatRupiah(trx.total)}</span>
					</div>
				</div>

				<div class="mt-4 pt-4 border-t border-dashed border-zinc-200">
					<p class="text-[9px] text-zinc-400 uppercase tracking-wider mb-2">Distribusi Dana</p>
					<div class="flex flex-col gap-1.5 text-[10px]">
						<div class="flex items-center justify-between">
							<span class="text-zinc-500">Diterima Seller</span>
							<span class="font-mono font-semibold text-teal-700">{formatRupiah(trx.sellerPaid)}</span>
						</div>
						<div class="flex items-center justify-between">
							<span class="text-zinc-500">Kurir</span>
							<span class="font-mono text-zinc-600">{formatRupiah(trx.kurirPaid)}</span>
						</div>
						<div class="flex items-center justify-between">
							<span class="text-zinc-500">Sistem</span>
							<span class="font-mono text-zinc-600">{formatRupiah(trx.sistemPaid)}</span>
						</div>
						{#if trx.isEkspedisi}
							<div class="flex items-center justify-between">
								<span class="text-zinc-500">Ekspedisi</span>
								<span class="font-mono text-zinc-600">{formatRupiah(trx.ekspedisiPaid)}</span>
							</div>
						{/if}
					</div>
				</div>
			</div>

			{#if trx.status === 'Dibatalkan' && trx.dibatalkanOleh}
				<div class="border border-rose-200 bg-rose-50 rounded-sm p-4">
					<p class="text-[10px] font-bold text-rose-700 uppercase tracking-wider">Transaksi Dibatalkan</p>
					<p class="text-[11px] text-rose-600 mt-1">Dibatalkan oleh: {trx.dibatalkanOleh}</p>
				</div>
			{/if}
		</div>
	</div>

	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- TRANSAKSI TERKAIT -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="mt-8 pt-6 border-t border-zinc-800/10">
		<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
			TRANSAKSI LAIN DARI PEMBELI INI
		</span>
		<div class="mt-3">
			<ListTransaksi />
		</div>
	</div>
</section>