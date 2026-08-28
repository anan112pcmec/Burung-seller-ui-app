<script lang="ts">
	// ///////////////////////////////////////////////////////////////////////
	// Types — sama seperti struct NotificationSeller
	// ///////////////////////////////////////////////////////////////////////

	interface NotificationSeller {
		id_seller: number;
		pengirim: string;
		judul: string;
		pesan: string;
		pop: number;
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

	// ///////////////////////////////////////////////////////////////////////
	// Mock data
	// ///////////////////////////////////////////////////////////////////////

	function todayAt(jam: number, menit: number): string {
		const d = new Date();
		d.setHours(jam, menit, 0, 0);
		return d.toISOString();
	}

	const notifikasi: NotificationSeller[] = [
		{
			id_seller: 1,
			pengirim: 'Sistem Transaksi',
			judul: 'Order #BRG-8823 menunggu approve',
			pesan: 'Pesanan baru untuk Kemeja Linen Oversize (2 pcs) masuk dan menunggu persetujuanmu.',
			pop: 0.85,
			archive: false,
			inbox: true,
			activity: true,
			created_at: todayAt(9, 12),
			expired_at: todayAt(23, 59),
			data: { metadata: { order_id: 'BRG-8823', jumlah: '2 pcs' }, special: null }
		},
        
		{
			id_seller: 1,
			pengirim: 'Sistem Transaksi',
			judul: 'Dana Rp1.240.000 berhasil dicairkan',
			pesan: 'Pencairan dana ke rekening BCA sudah diproses dan masuk dalam 1x24 jam kerja.',
			pop: 0.5,
			archive: false,
			inbox: true,
			activity: true,
			created_at: todayAt(10, 45),
			expired_at: '',
			data: { metadata: { rekening: 'BCA •••• 1123', jumlah: 'Rp1.240.000' }, special: null }
		},
		{
			id_seller: 1,
			pengirim: 'Kurir Ekspedisi',
			judul: 'Kamu diberi rating 5 oleh kurir',
			pesan: 'Merpati Express memberi rating 5 bintang atas kecepatan proses packing pesananmu.',
			pop: 0.25,
			archive: false,
			inbox: true,
			activity: true,
			created_at: todayAt(13, 30),
			expired_at: '',
			data: { metadata: { kurir: 'Merpati Express', rating: '5.0' }, special: null }
		},
		{
			id_seller: 1,
			pengirim: 'Sistem Transaksi',
			judul: 'Order #BRG-8830 sudah dikirim',
			pesan: 'Paket Rok Plisket Katun sudah diserahkan ke kurir dan dalam perjalanan ke pembeli.',
			pop: 0.4,
			archive: false,
			inbox: true,
			activity: true,
			created_at: todayAt(15, 5),
			expired_at: '',
			data: { metadata: { order_id: 'BRG-8830', kurir: 'Reguler' }, special: null }
		}
	];

	// ///////////////////////////////////////////////////////////////////////
	// Derived
	// ///////////////////////////////////////////////////////////////////////

	const hariIni = new Date();

	let aktivitasHariIni = $derived(
		notifikasi
			.filter((n) => n.activity && new Date(n.created_at).toDateString() === hariIni.toDateString())
			.sort((a, b) => new Date(b.created_at).getTime() - new Date(a.created_at).getTime())
	);

	// ///////////////////////////////////////////////////////////////////////
	// Helpers
	// ///////////////////////////////////////////////////////////////////////

	function formatJam(dateStr: string): string {
		return new Date(dateStr).toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' });
	}

	function refCode(n: NotificationSeller): string {
		return `AKT-${n.id_seller.toString().padStart(4, '0')}-${formatJam(n.created_at).replace(':', '')}`;
	}

	// Menentukan warna badge/aksen berdasarkan pengirim
	function getSenderAccent(pengirim: string): { bg: string; text: string; border: string } {
		if (pengirim.toLowerCase().includes('transaksi')) {
			return { bg: 'bg-emerald-50', text: 'text-emerald-700', border: 'border-emerald-200' };
		}
		if (pengirim.toLowerCase().includes('kurir')) {
			return { bg: 'bg-sky-50', text: 'text-sky-700', border: 'border-sky-200' };
		}
		return { bg: 'bg-zinc-100', text: 'text-zinc-700', border: 'border-zinc-200' };
	}
</script>

{#snippet ActivityCard(n: NotificationSeller)}
	{@const kode = refCode(n)}
	{@const accent = getSenderAccent(n.pengirim)}
	<div class="group relative w-full  bg-white border border-zinc-200/80 rounded-xl p-5 shadow-xs hover:shadow-md hover:border-zinc-300 transition-all duration-200 flex flex-col justify-between">
		
		<!-- Bagian Atas: Pengirim & Waktu -->
		<div>
			<div class="flex items-center justify-between gap-2 mb-3">
				<span class={`text-[10px] font-semibold tracking-wide uppercase px-2.5 py-1 rounded-md border ${accent.bg} ${accent.text} ${accent.border}`}>
					{n.pengirim}
				</span>
				<span class="text-xs font-mono text-zinc-400 font-medium">
					{formatJam(n.created_at)}
				</span>
			</div>

			<!-- Judul & Pesan -->
			<h3 class="text-sm font-bold text-zinc-900 group-hover:text-zinc-950 leading-snug">
				{n.judul}
			</h3>
			<p class="text-xs text-zinc-600 font-normal mt-1.5 leading-relaxed">
				{n.pesan}
			</p>
		</div>

		<!-- Bagian Metadata (jika ada) -->
		{#if Object.keys(n.data.metadata).length > 0}
			<div class="mt-4 pt-3 border-t border-zinc-100 space-y-1.5 bg-zinc-50/60 -mx-5 px-5 py-3">
				{#each Object.entries(n.data.metadata) as [key, value]}
					<div class="flex items-center justify-between text-[11px]">
						<span class="text-zinc-400 capitalize">{key.replace(/_/g, ' ')}</span>
						<span class="text-zinc-800 font-mono font-medium">{String(value)}</span>
					</div>
				{/each}
			</div>
		{/if}

		<!-- Bagian Bawah: Kode Referensi & Badge Prioritas -->
		<div class="mt-4 pt-3 border-t border-zinc-100 flex items-center justify-between text-[10px] text-zinc-400 font-mono">
			<span>{kode}</span>
			{#if n.pop >= 0.7}
				<span class="bg-zinc-900 text-white font-sans text-[9px] font-bold tracking-wider uppercase px-2 py-0.5 rounded-sm">
					Prioritas
				</span>
			{/if}
		</div>
	</div>
{/snippet}

<section id="activity-page" class="w-full min-h-screen bg-zinc-50/50 text-zinc-900 p-4 sm:p-6 lg:p-8">
	<!-- HEADER -->
	<div class="pb-6 mb-8 border-b border-zinc-200">
    <div class="flex space-x-4 items-center">
        <!-- Tombol kembali -->
        <a href="#back" class="w-10 h-10 flex items-center justify-center text-zinc-700 ">
            <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="m12 19-7-7 7-7"/><path d="M19 12H5"/>
            </svg>
        </a>
    

        <!-- Teks header -->
        <div>
            <div class="flex items-center gap-2">
                <span class="w-2 h-2 rounded-full bg-emerald-500 animate-pulse"></span>
                <span class="text-[10px] font-bold tracking-[0.18em] text-zinc-400 uppercase font-mono">
                    Live Feed Aktivitas
                </span>
            </div>
            <h1 class="mt-2 text-2xl sm:text-3xl font-bold tracking-tight text-zinc-900">
                Aktivitas Hari Ini
            </h1>
            <p class="text-xs sm:text-sm text-zinc-500 font-light mt-1">
                {hariIni.toLocaleDateString('id-ID', { weekday: 'long', day: '2-digit', month: 'long', year: 'numeric' })}
                · <span class="font-medium text-zinc-700">{aktivitasHariIni.length} aktivitas tercatat</span>
            </p>
        </div>
    </div>
</div>

	<!-- LIST KARTU -->
	{#if aktivitasHariIni.length === 0}
		<div class="flex flex-col items-center justify-center py-20 text-center bg-white border border-zinc-200 rounded-xl p-8">
			<p class="text-sm font-semibold text-zinc-800">Belum ada aktivitas hari ini</p>
			<p class="text-xs text-zinc-400 font-light mt-1">Aktivitas baru akan muncul di sini secara otomatis.</p>
		</div>
	{:else}
		<div class="space-y-4 overflow-y-auto scrollbar-none gap-5">
			{#each aktivitasHariIni as n (n.id_seller + n.created_at + n.judul)}
				{@render ActivityCard(n)}
			{/each}
		</div>
	{/if}
</section>