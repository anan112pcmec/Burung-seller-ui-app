<script lang="ts">
	// ///////////////////////////////////////////////////////////////////////
	// Types — mengikuti struct NotificationSeller (json tag snake_case)
	// ///////////////////////////////////////////////////////////////////////

	interface NotificationSeller {
		id_seller: number;
		pengirim: string;
		judul: string;
		pesan: string;
		pop: number; // skor prioritas 0.0 - 1.0
		archive: boolean;
		inbox: boolean;
		activity: boolean;
		created_at: string;
		expired_at: string;
		data: {
			metadata: Record<string, unknown>;
			special: unknown;
		};
	}

	type TabFilter = 'semua' | 'inbox' | 'aktivitas' | 'arsip';
	type SortMode = 'terbaru' | 'terlama' | 'prioritas';

	// ///////////////////////////////////////////////////////////////////////
	// Mock data — ganti dengan hasil fetch endpoint notifikasi seller asli
	// ///////////////////////////////////////////////////////////////////////

	const notifikasi: NotificationSeller[] = [
		{
			id_seller: 1,
			pengirim: 'Burung Official',
			judul: 'Toko kamu resmi terverifikasi',
			pesan:
				'Selamat! Toko Baju Modis sudah lolos verifikasi dan sekarang tampil dengan badge terpercaya di halaman produk.',
			pop: 0.9,
			archive: false,
			inbox: true,
			activity: false,
			created_at: '2026-08-27T09:15:00',
			expired_at: '',
			data: { metadata: { badge: 'terverifikasi' }, special: null }
		},
		{
			id_seller: 1,
			pengirim: 'Sistem Transaksi',
			judul: 'Order #BRG-8823 menunggu approve',
			pesan: 'Ada pesanan baru untuk Kemeja Linen Oversize (2 pcs). Segera approve sebelum batas waktu pengiriman.',
			pop: 0.8,
			archive: false,
			inbox: true,
			activity: true,
			created_at: '2026-08-27T07:40:00',
			expired_at: '2026-08-29T07:40:00',
			data: { metadata: { order_id: 'BRG-8823' }, special: null }
		},
		{
			id_seller: 1,
			pengirim: 'Tim Promo',
			judul: 'Voucher diskon iklan gratis 20%',
			pesan: 'Pakai kode PROMO20 buat naikin jangkauan produkmu minggu ini. Berlaku terbatas, jangan sampai kelewat.',
			pop: 0.55,
			archive: false,
			inbox: true,
			activity: false,
			created_at: '2026-08-25T14:00:00',
			expired_at: '2026-08-26T23:59:00',
			data: { metadata: { kode: 'PROMO20' }, special: null }
		},
		{
			id_seller: 1,
			pengirim: 'Sistem Transaksi',
			judul: 'Dana Rp1.240.000 berhasil dicairkan',
			pesan: 'Pencairan dana ke rekening BCA •••• 1123 sudah diproses dan akan masuk dalam 1x24 jam kerja.',
			pop: 0.4,
			archive: false,
			inbox: true,
			activity: true,
			created_at: '2026-08-24T11:20:00',
			expired_at: '',
			data: { metadata: { jumlah: 'Rp1.240.000' }, special: null }
		},
		{
			id_seller: 1,
			pengirim: 'Kurir Ekspedisi',
			judul: 'Kamu diberi rating 5 oleh kurir',
			pesan: 'Merpati Express memberi rating 5 bintang atas kecepatan proses packing pesananmu. Pertahankan!',
			pop: 0.2,
			archive: false,
			inbox: true,
			activity: true,
			created_at: '2026-08-22T16:05:00',
			expired_at: '',
			data: { metadata: {}, special: null }
		},
		{
			id_seller: 1,
			pengirim: 'Burung Official',
			judul: 'Pembaruan kebijakan komisi seller',
			pesan: 'Mulai bulan depan, potongan komisi untuk kategori Elektronik disesuaikan. Baca detail lengkapnya di sini.',
			pop: 0.3,
			archive: true,
			inbox: false,
			activity: false,
			created_at: '2026-08-10T08:00:00',
			expired_at: '',
			data: { metadata: { kategori: 'Elektronik' }, special: null }
		},
		{
			id_seller: 1,
			pengirim: 'Tim Promo',
			judul: 'Flash sale akhir bulan sudah berakhir',
			pesan: 'Terima kasih sudah berpartisipasi. Total 42 barang kamu terjual selama periode flash sale kemarin.',
			pop: 0.15,
			archive: true,
			inbox: false,
			activity: false,
			created_at: '2026-08-05T12:00:00',
			expired_at: '2026-08-06T00:00:00',
			data: { metadata: {}, special: null }
		}
	];

	// ///////////////////////////////////////////////////////////////////////
	// State — sorter & filter tab
	// ///////////////////////////////////////////////////////////////////////

	let activeTab: TabFilter = $state('semua');
	let sortBy: SortMode = $state('terbaru');
	let expandedId: string | null = $state(null);

	const tabs: { value: TabFilter; label: string }[] = [
		{ value: 'semua', label: 'Semua' },
		{ value: 'inbox', label: 'Kotak Masuk' },
		{ value: 'aktivitas', label: 'Aktivitas' },
		{ value: 'arsip', label: 'Arsip' }
	];

	// ///////////////////////////////////////////////////////////////////////
	// Helpers
	// ///////////////////////////////////////////////////////////////////////

	function initials(nama: string): string {
		return nama
			.split(' ')
			.slice(0, 2)
			.map((s) => s[0])
			.join('')
			.toUpperCase();
	}

	function isExpired(expiredAt: string): boolean {
		if (!expiredAt) return false;
		return new Date(expiredAt).getTime() < Date.now();
	}

	function formatWaktu(dateStr: string): string {
		const diffMs = Date.now() - new Date(dateStr).getTime();
		const menit = Math.floor(diffMs / 60_000);
		const jam = Math.floor(menit / 60);
		const hari = Math.floor(jam / 24);

		if (menit < 60) return `${Math.max(menit, 1)}m lalu`;
		if (jam < 24) return `${jam}j lalu`;
		if (hari < 7) return `${hari}h lalu`;
		return new Date(dateStr).toLocaleDateString('id-ID', { day: '2-digit', month: 'short' });
	}

	function rowKey(n: NotificationSeller): string {
		return `${n.id_seller}-${n.created_at}-${n.judul}`;
	}

	// ///////////////////////////////////////////////////////////////////////
	// Derived — filter + sort
	// ///////////////////////////////////////////////////////////////////////

	let filtered = $derived(
		notifikasi.filter((n) => {
			if (activeTab === 'inbox') return n.inbox && !n.archive;
			if (activeTab === 'aktivitas') return n.activity;
			if (activeTab === 'arsip') return n.archive;
			return true;
		})
	);

	let sorted = $derived(
		[...filtered].sort((a, b) => {
			if (sortBy === 'prioritas') return b.pop - a.pop;
			const ta = new Date(a.created_at).getTime();
			const tb = new Date(b.created_at).getTime();
			return sortBy === 'terlama' ? ta - tb : tb - ta;
		})
	);

	let jumlahInbox = $derived(notifikasi.filter((n) => n.inbox && !n.archive).length);
	let jumlahAktivitas = $derived(notifikasi.filter((n) => n.activity).length);
	let jumlahArsip = $derived(notifikasi.filter((n) => n.archive).length);

	function jumlahTab(tab: TabFilter): number {
		if (tab === 'inbox') return jumlahInbox;
		if (tab === 'aktivitas') return jumlahAktivitas;
		if (tab === 'arsip') return jumlahArsip;
		return notifikasi.length;
	}
</script>

{#snippet MailBox(n: NotificationSeller)}
	{@const expired = isExpired(n.expired_at)}
	{@const isOpen = expandedId === rowKey(n)}
	<div
		role="button"
		tabindex="0"
		onclick={() => (expandedId = isOpen ? null : rowKey(n))}
		onkeydown={(e) => e.key === 'Enter' && (expandedId = isOpen ? null : rowKey(n))}
		class="w-full border-b border-zinc-100 px-4 sm:px-5 py-3.5 flex gap-3 cursor-pointer hover:bg-zinc-50/80 transition-colors {expired
			? 'opacity-50'
			: ''}"
	>
		<!-- avatar -->
		<div class="w-8 h-8 sm:w-9 sm:h-9 rounded-full bg-zinc-100 border border-zinc-200 flex items-center justify-center flex-shrink-0 text-[10px] font-bold text-zinc-700">
			{initials(n.pengirim)}
		</div>

		<!-- konten -->
		<div class="min-w-0 flex-1">
			<div class="flex items-start justify-between gap-2">
				<div class="min-w-0">
					<p class="text-[10px] sm:text-[11px] text-zinc-400 truncate">{n.pengirim}</p>
					<p class="text-xs sm:text-sm font-semibold text-zinc-900 truncate">{n.judul}</p>
				</div>
				<span class="text-[9px] sm:text-[10px] font-mono text-zinc-400 flex-shrink-0 mt-0.5">
					{formatWaktu(n.created_at)}
				</span>
			</div>

			<p class="text-[11px] sm:text-xs text-zinc-600 font-light mt-1 {isOpen ? '' : 'truncate'}">
				{n.pesan}
			</p>

			{#if isOpen && Object.keys(n.data.metadata).length > 0}
				<div class="mt-2 flex flex-wrap gap-1.5">
					{#each Object.entries(n.data.metadata) as [key, value]}
						<span class="text-[9px] font-mono text-zinc-600 bg-zinc-100 border border-zinc-200 rounded-sm px-1.5 py-0.5">
							{key}: {String(value)}
						</span>
					{/each}
				</div>
			{/if}

			<div class="flex items-center gap-1.5 mt-2">
				{#if n.activity}
					<span class="text-[8px] sm:text-[9px] font-bold uppercase tracking-wider text-zinc-600 border border-zinc-200 rounded-sm px-1.5 py-0.5">
						Aktivitas
					</span>
				{/if}
				{#if n.pop >= 0.7}
					<span class="text-[8px] sm:text-[9px] font-bold uppercase tracking-wider text-white bg-zinc-900 rounded-sm px-1.5 py-0.5">
						Prioritas Tinggi
					</span>
				{/if}
				{#if expired}
					<span class="text-[8px] sm:text-[9px] font-bold uppercase tracking-wider text-zinc-400 border border-zinc-200 rounded-sm px-1.5 py-0.5">
						Kedaluwarsa
					</span>
				{:else if n.expired_at}
					<span class="text-[8px] sm:text-[9px] font-mono text-zinc-400">
						Berlaku s.d. {new Date(n.expired_at).toLocaleDateString('id-ID', { day: '2-digit', month: 'short' })}
					</span>
				{/if}
			</div>
		</div>
	</div>
{/snippet}

<section id="mail-box" class="w-full bg-white space-y-5 text-zinc-900 border border-zinc-200 rounded-lg shadow-xs flex flex-col max-h-[42rem]">
	<!-- SORTER + FILTER TAB -->
     <div class="px-6 pt-4 max-w-120">
			<div class="grid grid-cols-[10%_2%_88%]">
                <div class="flex justify-around items-end">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-arrow-left-icon lucide-arrow-left"><path d="m12 19-7-7 7-7"/><path d="M19 12H5"/></svg>
                </div>
                <div></div>
                <div>
                    <span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
                    MailBox
                    </span>
                    <h1 class="mt-1text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
                        Informasi Pesan Masuk
                    </h1>
                </div>
            </div>
		</div>

	<div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-3 px-4 sm:px-5 py-3.5 border-b border-zinc-100 bg-zinc-50/50 rounded-t-lg">
		<div class="inline-flex border border-zinc-200 bg-white rounded-md overflow-x-auto shadow-2xs">
			{#each tabs as tab}
				<button
					type="button"
					onclick={() => (activeTab = tab.value)}
					class="px-2.5 sm:px-3 py-1.5 text-[9px] sm:text-[10px] font-medium uppercase tracking-wider whitespace-nowrap transition-colors {activeTab === tab.value
						? 'bg-zinc-900 text-white'
						: 'text-zinc-600 hover:bg-zinc-100'}"
				>
					{tab.label} <span class="font-mono opacity-70">({jumlahTab(tab.value)})</span>
				</button>
			{/each}
		</div>

		<div class="relative self-start sm:self-auto">
			<select
				bind:value={sortBy}
				class="appearance-none border border-zinc-200 rounded-md pl-3 pr-8 py-1.5 text-[10px] sm:text-xs text-zinc-700 bg-white shadow-2xs focus:outline-none focus:border-zinc-400 cursor-pointer transition-colors"
			>
				<option value="terbaru">Urutkan: Terbaru</option>
				<option value="terlama">Urutkan: Terlama</option>
				<option value="prioritas">Urutkan: Prioritas</option>
			</select>
			<svg
				xmlns="http://www.w3.org/2000/svg"
				class="w-3.5 h-3.5 absolute right-2.5 top-1/2 -translate-y-1/2 text-zinc-400 pointer-events-none"
				fill="none"
				viewBox="0 0 24 24"
				stroke="currentColor"
				stroke-width="1.75"
			>
				<path d="m6 9 6 6 6-6" />
			</svg>
		</div>
	</div>

	<!-- MAIL LIST VERTICAL -->
	<div class="flex-1 overflow-y-auto">
		{#if sorted.length === 0}
			<div class="flex flex-col items-center justify-center py-16 text-center px-4">
				<p class="text-xs sm:text-sm font-semibold text-zinc-800">Tidak ada notifikasi</p>
				<p class="text-[10px] sm:text-[11px] text-zinc-400 font-light mt-1">
					Belum ada apa-apa di kategori ini.
				</p>
			</div>
		{:else}
			{#each sorted as n (rowKey(n))}
				{@render MailBox(n)}
			{/each}
		{/if}
	</div>
</section>