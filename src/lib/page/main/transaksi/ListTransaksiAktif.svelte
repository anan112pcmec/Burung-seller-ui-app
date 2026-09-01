<script lang="ts">
	// ///////////////////////////////////////////////////////////////////////
	// Types
	// ///////////////////////////////////////////////////////////////////////

	type TabAktif = 'dipesan' | 'diproses';
	type StatusFotoPacking = 'kosong' | 'mengunggah' | 'terunggah';

	interface TransaksiAktif {
		idTransaksi: number;
		kodeOrderSistem: string;
		namaBarang: string;
		namaVarian: string;
		kuantitas: number;
		total: number;
		waktuPesan: string;
		catatanPembeli: string;
	}

	interface FotoPacking {
		id: number;
		namaFile: string;
		status: StatusFotoPacking;
	}

	// ///////////////////////////////////////////////////////////////////////
	// State — tab & seleksi
	// ///////////////////////////////////////////////////////////////////////

	let tabAktif = $state<TabAktif>('dipesan');
	let idTerpilih = $state<number | null>(null);

	// ///////////////////////////////////////////////////////////////////////
	// Mock data — ganti dengan fetch list transaksi by status & id_seller
	// ///////////////////////////////////////////////////////////////////////

	let transaksiDipesan = $state<TransaksiAktif[]>([
		{
			idTransaksi: 5001,
			kodeOrderSistem: 'ORD-20260828-0011',
			namaBarang: 'Kemeja Flanel Kotak-kotak',
			namaVarian: 'Merah, size L',
			kuantitas: 1,
			total: 159_000,
			waktuPesan: '2026-08-28T08:12:00Z',
			catatanPembeli: 'Tolong dibungkus rapi ya kak.'
		},
		{
			idTransaksi: 5002,
			kodeOrderSistem: 'ORD-20260828-0014',
			namaBarang: 'Kaos Polos Combed 30s',
			namaVarian: 'Hitam, size M',
			kuantitas: 3,
			total: 195_000,
			waktuPesan: '2026-08-28T09:40:00Z',
			catatanPembeli: ''
		},
		{
			idTransaksi: 5003,
			kodeOrderSistem: 'ORD-20260828-0022',
			namaBarang: 'Celana Chino Slimfit',
			namaVarian: 'Khaki, size 32',
			kuantitas: 1,
			total: 189_000,
			waktuPesan: '2026-08-28T10:05:00Z',
			catatanPembeli: 'Kalau bisa dikirim hari ini kak, buat acara besok.'
		}
	]);

	let transaksiDiproses = $state<TransaksiAktif[]>([
		{
			idTransaksi: 4990,
			kodeOrderSistem: 'ORD-20260827-0142',
			namaBarang: 'Jaket Bomber Varsity',
			namaVarian: 'Navy, size L',
			kuantitas: 1,
			total: 245_000,
			waktuPesan: '2026-08-27T14:32:00Z',
			catatanPembeli: ''
		},
		{
			idTransaksi: 4988,
			kodeOrderSistem: 'ORD-20260827-0139',
			namaBarang: 'Tumbler Stainless 500ml',
			namaVarian: 'Sage Green',
			kuantitas: 2,
			total: 130_000,
			waktuPesan: '2026-08-27T11:20:00Z',
			catatanPembeli: ''
		}
	]);

	let daftarAktif = $derived(tabAktif === 'dipesan' ? transaksiDipesan : transaksiDiproses);
	let transaksiTerpilih = $derived(daftarAktif.find((t) => t.idTransaksi === idTerpilih) ?? null);

	function pilihTab(tab: TabAktif) {
		tabAktif = tab;
		idTerpilih = null;
		catatanTolak = '';
		jadwalkanOtomatis = false;
		waktuAutoKirim = '';
	}

	function formatRupiah(n: number): string {
		return `Rp${n.toLocaleString('id-ID')}`;
	}

	function formatWaktu(iso: string): string {
		return new Date(iso).toLocaleString('id-ID', { day: '2-digit', month: 'short', hour: '2-digit', minute: '2-digit' });
	}

	// ///////////////////////////////////////////////////////////////////////
	// Aksi — tab "Dipesan": Terima / Tolak
	// ///////////////////////////////////////////////////////////////////////

	let catatanTolak = $state('');
	let modeTolak = $state(false);
	let jadwalkanOtomatis = $state(false);
	let waktuAutoKirim = $state('');

	function terimaPesanan() {
		if (!transaksiTerpilih) return;
		// TODO: panggil ApproveOrderTransaksi. Kalau jadwalkanOtomatis dicentang,
		// kirim IsAuto=true & AutoPengiriman=waktuAutoKirim — backend akan
		// menaruh jadwal ini di Redis (TTL) buat auto-trigger pengiriman nanti.
		transaksiDipesan = transaksiDipesan.filter((t) => t.idTransaksi !== transaksiTerpilih!.idTransaksi);
		idTerpilih = null;
		jadwalkanOtomatis = false;
		waktuAutoKirim = '';
	}

	function bukaTolak() {
		catatanTolak = '';
		modeTolak = true;
	}

	function konfirmasiTolak() {
		if (!transaksiTerpilih || catatanTolak.trim().length === 0) return;
		// TODO: panggil UnApproveOrderTransaksi dengan Catatan = catatanTolak.
		// Backend WAJIB alasan ini, gak bisa kosong.
		transaksiDipesan = transaksiDipesan.filter((t) => t.idTransaksi !== transaksiTerpilih!.idTransaksi);
		idTerpilih = null;
		modeTolak = false;
		catatanTolak = '';
	}

	// ///////////////////////////////////////////////////////////////////////
	// Aksi — tab "Diproses": upload foto packing lalu Kirim Barang
	// ///////////////////////////////////////////////////////////////////////

	let fotoPackingPerTransaksi = $state<Record<number, FotoPacking[]>>({});

	function fotoUntuk(id: number): FotoPacking[] {
		return fotoPackingPerTransaksi[id] ?? [];
	}

	function tambahFotoPacking(e: Event, idTransaksi: number) {
		const target = e.target as HTMLInputElement;
		const file = target.files?.[0];
		if (!file) return;

		if (!fotoPackingPerTransaksi[idTransaksi]) fotoPackingPerTransaksi[idTransaksi] = [];

		const baru: FotoPacking = { id: Date.now(), namaFile: file.name, status: 'mengunggah' };
		fotoPackingPerTransaksi[idTransaksi].push(baru);

		// TODO: ganti dengan alur upload nyata ke media_services (dua tahap:
		// minta slot/ekstensi, lalu PUT file), baru set status 'terunggah'.
		setTimeout(() => {
			baru.status = 'terunggah';
		}, 800);
	}

	function hapusFotoPacking(idTransaksi: number, idFoto: number) {
		if (!fotoPackingPerTransaksi[idTransaksi]) return;
		fotoPackingPerTransaksi[idTransaksi] = fotoPackingPerTransaksi[idTransaksi].filter((f) => f.id !== idFoto);
	}

	let fotoSiapUntukKirim = $derived(
		transaksiTerpilih ? fotoUntuk(transaksiTerpilih.idTransaksi).some((f) => f.status === 'terunggah') : false
	);

	function kirimBarang() {
		if (!transaksiTerpilih || !fotoSiapUntukKirim) return;
		// TODO: panggil KirimOrderTransaksi. Backend akan bikin record
		// Pengiriman/PengirimanEkspedisi tergantung IsEkspedisi transaksi ini.
		transaksiDiproses = transaksiDiproses.filter((t) => t.idTransaksi !== transaksiTerpilih!.idTransaksi);
		idTerpilih = null;
	}
</script>

<section id="list-transaksi-aktif" class="w-full h-screen bg-white p-4 sm:px-6 lg:p-8 text-slate-950">
	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- HEADER + TAB -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="pb-5 mb-5 border-b border-zinc-800/10">
		<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
			ORDER MASUK
		</span>
		<h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
			Pesanan Masuk
		</h1>

		<div class="inline-flex border border-zinc-800/20 rounded-xs overflow-hidden mt-4">
			<button
				type="button"
				onclick={() => pilihTab('dipesan')}
				class="px-4 py-2 text-[10px] sm:text-[11px] font-medium uppercase tracking-wider transition duration-200 {tabAktif === 'dipesan'
					? 'bg-slate-950 text-white'
					: 'text-slate-600 hover:bg-slate-50'}"
			>
				Pesanan Masuk ({transaksiDipesan.length})
			</button>
			<button
				type="button"
				onclick={() => pilihTab('diproses')}
				class="px-4 py-2 text-[10px] sm:text-[11px] font-medium uppercase tracking-wider transition duration-200 {tabAktif === 'diproses'
					? 'bg-slate-950 text-white'
					: 'text-slate-600 hover:bg-slate-50'}"
			>
				Pesanan Telah Diproses ({transaksiDiproses.length})
			</button>
		</div>
	</div>

	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- LIST + PANEL AKSI -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="grid grid-cols-1 lg:grid-cols-[68%_32%] gap-4">
		<!-- LIST TRANSAKSI AKTIF -->
		<div class="flex flex-col gap-2">
			{#each daftarAktif as trx (trx.idTransaksi)}
				<button
					type="button"
					onclick={() => {
						idTerpilih = trx.idTransaksi;
						modeTolak = false;
					}}
					class="text-left border rounded-sm p-3 transition-colors {idTerpilih === trx.idTransaksi
						? 'border-slate-950 bg-zinc-50'
						: 'border-zinc-200 hover:border-zinc-400 bg-white'}"
				>
					<div class="flex items-start justify-between gap-3 flex-wrap">
						<div class="min-w-0">
							<span class="text-[9px] text-zinc-400 font-mono uppercase tracking-wider">{trx.kodeOrderSistem}</span>
							<p class="text-xs font-semibold text-zinc-800 truncate mt-0.5">{trx.namaBarang}</p>
							<p class="text-[10px] text-zinc-500 mt-0.5">{trx.namaVarian} · {trx.kuantitas} unit</p>
							{#if trx.catatanPembeli}
								<p class="text-[10px] text-zinc-400 italic mt-1 truncate">"{trx.catatanPembeli}"</p>
							{/if}
						</div>
						<div class="text-right flex-shrink-0">
							<p class="text-xs font-bold font-mono text-zinc-900">{formatRupiah(trx.total)}</p>
							<p class="text-[9px] text-zinc-400 font-mono mt-0.5">{formatWaktu(trx.waktuPesan)}</p>
						</div>
					</div>
				</button>
			{/each}

			{#if daftarAktif.length === 0}
				<p class="text-[11px] text-zinc-400 text-center py-10">
					{tabAktif === 'dipesan' ? 'Belum ada pesanan baru masuk.' : 'Tidak ada pesanan yang sedang diproses.'}
				</p>
			{/if}
		</div>

		<!-- PANEL AKSI -->
		<div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5 h-fit lg:sticky lg:top-4">
			{#if !transaksiTerpilih}
				<p class="text-[11px] text-zinc-400 text-center py-8">
					Pilih pesanan dari daftar di sebelah kiri untuk melihat aksi yang tersedia.
				</p>
			{:else if tabAktif === 'dipesan'}
				<!-- AKSI: TERIMA / TOLAK -->
				<span class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					{transaksiTerpilih.kodeOrderSistem}
				</span>
				<p class="text-xs font-semibold text-zinc-800 mt-1">{transaksiTerpilih.namaBarang}</p>
				<p class="text-[10px] text-zinc-500 mt-0.5">{transaksiTerpilih.namaVarian} · {transaksiTerpilih.kuantitas} unit</p>
				<p class="text-sm font-bold font-mono text-zinc-900 mt-2">{formatRupiah(transaksiTerpilih.total)}</p>

				{#if !modeTolak}
					<div class="mt-4 border-t border-dashed border-zinc-200 pt-4">
						<label class="flex items-center gap-2 text-[10px] text-zinc-600 mb-2">
							<input type="checkbox" bind:checked={jadwalkanOtomatis} class="accent-slate-950" />
							Jadwalkan kirim otomatis
						</label>
						{#if jadwalkanOtomatis}
							<input
								type="datetime-local"
								bind:value={waktuAutoKirim}
								class="w-full border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 focus:outline-none focus:border-slate-950 transition-colors mb-3"
							/>
							<p class="text-[9px] text-zinc-400 mb-3 leading-relaxed">
								Sistem akan otomatis memproses pengiriman pada waktu yang dipilih tanpa perlu kamu buka aplikasi lagi.
							</p>
						{/if}

						<button
							type="button"
							onclick={terimaPesanan}
							class="w-full px-4 py-2.5 bg-slate-950 text-white text-[10px] font-bold uppercase tracking-wider rounded-sm hover:bg-slate-800 transition-colors"
						>
							Terima Pesanan
						</button>
						<button
							type="button"
							onclick={bukaTolak}
							class="w-full mt-2 px-4 py-2.5 border border-rose-300 text-rose-600 text-[10px] font-bold uppercase tracking-wider rounded-sm hover:bg-rose-50 transition-colors"
						>
							Tolak Pesanan
						</button>
					</div>
				{:else}
					<div class="mt-4 border-t border-dashed border-zinc-200 pt-4">
						<label class="flex flex-col gap-1.5">
							<span class="text-[10px] font-medium text-slate-600">Alasan Penolakan (wajib diisi)</span>
							<textarea
								bind:value={catatanTolak}
								rows={3}
								placeholder="Jelaskan alasan menolak pesanan ini ke pembeli..."
								class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 placeholder-zinc-400 focus:outline-none focus:border-rose-500 transition-colors resize-none"
							></textarea>
						</label>
						<div class="flex items-center gap-3 mt-3">
							<button
								type="button"
								disabled={catatanTolak.trim().length === 0}
								onclick={konfirmasiTolak}
								class="px-4 py-2 bg-rose-600 text-white text-[10px] font-bold uppercase tracking-wider rounded-sm hover:bg-rose-700 transition-colors disabled:opacity-30 disabled:cursor-not-allowed"
							>
								Konfirmasi Tolak
							</button>
							<button type="button" onclick={() => (modeTolak = false)} class="text-[10px] font-medium text-slate-500 underline">
								Batal
							</button>
						</div>
					</div>
				{/if}
			{:else}
				<!-- AKSI: UPLOAD FOTO PACKING LALU KIRIM -->
				<span class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					{transaksiTerpilih.kodeOrderSistem}
				</span>
				<p class="text-xs font-semibold text-zinc-800 mt-1">{transaksiTerpilih.namaBarang}</p>
				<p class="text-[10px] text-zinc-500 mt-0.5">{transaksiTerpilih.namaVarian} · {transaksiTerpilih.kuantitas} unit</p>

				<div class="mt-4 border-t border-dashed border-zinc-200 pt-4">
					<div class="bg-zinc-50 border border-zinc-200 rounded-md p-3 text-[10px] text-zinc-500 leading-relaxed mb-3">
						Unggah minimal satu foto bukti barang sudah dikemas rapi sebelum bisa mengirim pesanan ini. Foto ini
						berguna sebagai bukti kondisi barang saat berangkat, terutama kalau nanti ada komplain dari pembeli.
					</div>

					<div class="flex flex-col gap-2 mb-3">
						{#each fotoUntuk(transaksiTerpilih.idTransaksi) as foto (foto.id)}
							<div class="flex items-center gap-2 border border-zinc-200 rounded-md p-2">
								<div class="w-7 h-7 rounded flex items-center justify-center flex-shrink-0 {foto.status === 'terunggah' ? 'bg-teal-50' : 'bg-zinc-100'}">
									{#if foto.status === 'terunggah'}
										<svg xmlns="http://www.w3.org/2000/svg" class="w-3.5 h-3.5 text-teal-600" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
											<path d="m9 15 2 2 4-4"/><circle cx="12" cy="12" r="10"/>
										</svg>
									{:else}
										<svg xmlns="http://www.w3.org/2000/svg" class="w-3.5 h-3.5 text-zinc-400 animate-spin" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
											<path d="M21 12a9 9 0 1 1-6.219-8.56"/>
										</svg>
									{/if}
								</div>
								<span class="text-[10px] text-zinc-600 truncate flex-1">{foto.namaFile}</span>
								<button type="button" onclick={() => hapusFotoPacking(transaksiTerpilih!.idTransaksi, foto.id)} class="text-[9px] text-rose-500 underline flex-shrink-0">
									Hapus
								</button>
							</div>
						{/each}
					</div>

					<label class="inline-flex items-center gap-1.5 text-[10px] font-medium text-slate-950 border border-zinc-300 rounded px-3 py-2 cursor-pointer hover:bg-zinc-50 transition-colors w-full justify-center">
						+ Tambah Foto Packing
						<input type="file" accept="image/*" class="hidden" onchange={(e) => tambahFotoPacking(e, transaksiTerpilih!.idTransaksi)} />
					</label>

					<button
						type="button"
						disabled={!fotoSiapUntukKirim}
						onclick={kirimBarang}
						class="w-full mt-3 px-4 py-2.5 bg-slate-950 text-white text-[10px] font-bold uppercase tracking-wider rounded-sm hover:bg-slate-800 transition-colors disabled:opacity-30 disabled:cursor-not-allowed"
					>
						Kirim Barang
					</button>
					{#if !fotoSiapUntukKirim}
						<p class="text-[9px] text-zinc-400 mt-2 text-center">Unggah foto packing dulu buat mengaktifkan tombol ini.</p>
					{/if}
				</div>
			{/if}
		</div>
	</div>
</section>