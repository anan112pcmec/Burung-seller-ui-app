<script lang="ts">
	// Simulasi data diskon langsung di state
	let statusDiskon = $state('Draft'); // Coba ubah ke 'Aktif' atau 'Berakhir'
	let nama = $state('Promo Kilat Imlek 2026');
	let diskonPersen = $state(15.5);
	let berlakuMulai = $state('2026-02-10');
	let berlakuSampai = $state('2026-02-20');
	let deskripsi = $state('Potongan harga spesial awal tahun untuk item pilihan.');
	let createdAt = '2026-02-01';

	// Lock State: Hanya boleh diedit jika status "Draft"
	let isLocked = $derived(statusDiskon !== 'Draft');

	// Hitung Skor Kelengkapan
	let kelengkapanScore = $derived.by(() => {
		let score = 0;
		if (nama.trim().length >= 3) score += 25;
		if (diskonPersen > 0 && diskonPersen <= 100) score += 25;
		if (berlakuMulai && berlakuSampai) score += 25;
		if (deskripsi.trim().length >= 5) score += 25;
		return score;
	});

	// Simulasi kalkulator hemat
	let sampelHarga = 150000;
	let nominalPotongan = $derived((sampelHarga * (diskonPersen || 0)) / 100);
	let hargaAkhir = $derived(sampelHarga - nominalPotongan);

	// --- Helper Functions untuk UI Card ---
	function formatTanggal(tglStr: string) {
		if (!tglStr) return '—';
		const d = new Date(tglStr);
		if (isNaN(d.getTime())) return tglStr;
		return d.toLocaleDateString('id-ID', { day: '2-digit', month: 'short', year: 'numeric' });
	}

	function sisaHari(sampaiStr: string) {
		if (!sampaiStr) return 0;
		const target = new Date(sampaiStr).getTime();
		const now = new Date().getTime();
		const diffMs = target - now;
		return Math.ceil(diffMs / (1000 * 60 * 60 * 24));
	}

	function persenWaktuBerjalan(mulaiStr: string, sampaiStr: string) {
		if (!mulaiStr || !sampaiStr) return 0;
		const start = new Date(mulaiStr).getTime();
		const end = new Date(sampaiStr).getTime();
		const now = new Date().getTime();

		if (now <= start) return 0;
		if (now >= end) return 100;
		return Math.min(100, Math.max(0, ((now - start) / (end - start)) * 100));
	}

	function statusClass(status: string) {
		switch (status) {
			case 'Aktif':
				return 'bg-emerald-50 text-emerald-700 border border-emerald-200';
			case 'Draft':
				return 'bg-amber-50 text-amber-700 border border-amber-200';
			case 'Berakhir':
				return 'bg-zinc-100 text-zinc-500 border border-zinc-200';
			default:
				return 'bg-zinc-100 text-zinc-600';
		}
	}

    const persenWaktu = persenWaktuBerjalan(berlakuMulai, berlakuSampai)
	const sisa = sisaHari(berlakuSampai)
	const aktif = statusDiskon === "Aktif"
</script>


<section id="edit-diskon">
<div class="max-w-6xl mx-auto p-6">
	<!-- Top Bar -->
	<div class="mb-6 flex items-center justify-between">
		<div class="flex items-center gap-3">
			<h1 class="text-xl font-bold text-zinc-800">Edit Diskon Produk</h1>
			<span class="px-2.5 py-0.5 text-[10px] font-mono font-bold uppercase rounded-full {statusClass(statusDiskon)}">
				{statusDiskon}
			</span>
		</div>
		<button onclick={() => history.back()} class="px-3 py-1.5 text-xs font-medium text-zinc-600 bg-zinc-100 hover:bg-zinc-200 rounded-md">
			Batal
		</button>
	</div>

	<!-- BANNER JIKA LOCKED -->
	{#if isLocked}
		<div class="mb-6 p-4 rounded-xl bg-amber-50 border border-amber-200 flex items-start gap-3">
			<svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5 text-amber-600 shrink-0 mt-0.5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
				<rect width="18" height="11" x="3" y="11" rx="2" ry="2"/><path d="M7 11V7a5 5 0 0 1 10 0v4"/>
			</svg>
			<div class="text-xs text-amber-900">
				<p class="font-bold">Diskon Terkunci (Read-Only)</p>
				<p class="mt-0.5 text-amber-800">Status promo saat ini <strong>{statusDiskon}</strong>. Hanya promo berstatus <strong>Draft</strong> yang dapat diubah.</p>
			</div>
		</div>
	{/if}

	<div class="grid grid-cols-1 lg:grid-cols-12 gap-8 items-start">
		<!-- KIRI: FORM EDIT -->
		<div class="lg:col-span-7 bg-white border border-zinc-200 rounded-xl p-6 shadow-sm space-y-4">
			<div>
				<label for="nama-promo" class="block text-xs font-semibold text-zinc-700 mb-1">Nama Promo</label>
				<input
					id="nama-promo"
					type="text"
					bind:value={nama}
					disabled={isLocked}
					class="w-full px-3 py-2 text-xs border border-zinc-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-teal-500/20 focus:border-teal-600 disabled:bg-zinc-100 disabled:text-zinc-500"
				/>
			</div>

			<div>
				<label for="diskon-persen" class="block text-xs font-semibold text-zinc-700 mb-1">Besar Diskon (%)</label>
				<input
					id="diskon-persen"
					type="number"
					bind:value={diskonPersen}
					disabled={isLocked}
					min="1"
					max="100"
					class="w-28 px-3 py-2 text-xs font-mono font-bold border border-zinc-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-teal-500/20 focus:border-teal-600 disabled:bg-zinc-100 disabled:text-zinc-500"
				/>
			</div>

			<div class="grid grid-cols-2 gap-4">
				<div>
					<label for="berlaku-mulai" class="block text-xs font-semibold text-zinc-700 mb-1">Tanggal Mulai</label>
					<input
						id="berlaku-mulai"
						type="date"
						bind:value={berlakuMulai}
						disabled={isLocked}
						class="w-full px-3 py-2 text-xs font-mono border border-zinc-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-teal-500/20 focus:border-teal-600 disabled:bg-zinc-100 disabled:text-zinc-500"
					/>
				</div>
				<div>
					<label for="berlaku-sampai" class="block text-xs font-semibold text-zinc-700 mb-1">Tanggal Selesai</label>
					<input
						id="berlaku-sampai"
						type="date"
						bind:value={berlakuSampai}
						disabled={isLocked}
						class="w-full px-3 py-2 text-xs font-mono border border-zinc-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-teal-500/20 focus:border-teal-600 disabled:bg-zinc-100 disabled:text-zinc-500"
					/>
				</div>
			</div>

			<div>
				<label for="deskripsi-promo" class="block text-xs font-semibold text-zinc-700 mb-1">Syarat & Ketentuan</label>
				<textarea
					id="deskripsi-promo"
					bind:value={deskripsi}
					disabled={isLocked}
					rows="3"
					class="w-full px-3 py-2 text-xs border border-zinc-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-teal-500/20 focus:border-teal-600 disabled:bg-zinc-100 disabled:text-zinc-500 resize-none"
				></textarea>
			</div>

			<div class="pt-2 flex justify-end">
				<button
					type="button"
					disabled={isLocked}
					class="px-5 py-2 text-xs font-semibold text-white bg-teal-700 hover:bg-teal-800 disabled:bg-zinc-300 disabled:cursor-not-allowed rounded-lg shadow-sm"
				>
					{isLocked ? 'Terkunci' : 'Simpan Perubahan'}
				</button>
			</div>
		</div>

		<!-- KANAN: PREVIEW CARD & KELENGKAPAN -->
		<div class="lg:col-span-5 space-y-6">
			<!-- Live Card Diskon Tiket Preview -->
			<div>
				<div class="relative w-full max-w-[23rem] h-[9.5rem] flex-shrink-0 border border-zinc-200 rounded-lg bg-white shadow-sm overflow-hidden flex transition-colors duration-150">
					<!-- SISI KIRI — Stub Kupon -->
					<div class="w-24 flex-shrink-0 flex flex-col items-center justify-center gap-1 border-r border-dashed border-zinc-200 bg-zinc-50/60 py-4">
						<span class="text-[9px] font-mono text-zinc-400 uppercase tracking-wider">Diskon</span>
						<span class="text-2xl font-bold font-mono tracking-tight {aktif ? 'text-teal-700' : 'text-zinc-800'}">
							{diskonPersen || 0}%
						</span>
						<span class="px-1.5 py-0.5 rounded text-[8px] font-medium uppercase tracking-wider {statusClass(statusDiskon)}">
							{statusDiskon}
						</span>
					</div>

					<!-- Notch Efek Tiket -->
					<div class="absolute left-[5.5rem] -top-1.5 w-3 h-3 rounded-full bg-zinc-100 border border-zinc-200"></div>
					<div class="absolute left-[5.5rem] -bottom-1.5 w-3 h-3 rounded-full bg-zinc-100 border border-zinc-200"></div>

					<!-- SISI KANAN — Detail Promo -->
					<div class="flex-1 p-3 flex flex-col justify-between min-w-0">
						<div class="space-y-1.5">
							<div class="flex items-start justify-start gap-1">
								<span class="text-[9px] text-zinc-400 font-mono">Dibuat {formatTanggal(createdAt)}</span>
							</div>

							<div class="flex justify-between items-start">
								<h3 class="text-xs font-semibold text-start text-zinc-800 leading-tight truncate">
									{nama || 'Nama Promo'}
								</h3>
							</div>
							
							<p class="text-[10px] text-zinc-400 text-start line-clamp-2 mt-0.5 leading-tight">
								{deskripsi || 'Belum ada deskripsi.'}
							</p>
						</div>

						<div>
							<div class="flex items-center justify-between text-[9px] font-mono text-zinc-500 mb-1">
								<span>{formatTanggal(berlakuMulai)}</span>
								<span>{formatTanggal(berlakuSampai)}</span>
							</div>
							<div class="w-full h-1 bg-slate-100 rounded-full overflow-hidden">
								<div
									class="h-full rounded-full {statusDiskon === 'Berakhir' ? 'bg-slate-300' : 'bg-teal-600'}"
									style:width="{persenWaktu}%"
								></div>
							</div>
							<p class="text-[9px] text-zinc-400 font-light mt-1">
								{#if statusDiskon === "Berakhir"}
									sudah berakhir {Math.abs(sisa)} hari lalu
								{:else if statusDiskon === "Draft"}
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
			</div>

			<!-- Progress Kelengkapan -->
			<div class="bg-white border border-zinc-200 rounded-xl p-5 shadow-sm">
				<div class="flex items-center justify-between mb-2">
					<span class="text-xs font-semibold text-zinc-700">Kelengkapan Promo</span>
					<span class="text-xs font-mono font-bold text-teal-700">{kelengkapanScore}%</span>
				</div>
				<div class="w-full h-2 bg-zinc-100 rounded-full overflow-hidden">
					<div class="h-full bg-teal-600 transition-all duration-300 rounded-full" style:width="{kelengkapanScore}%"></div>
				</div>
			</div>

			<!-- Simulasi Kalkulator -->
			<div class="bg-white border border-zinc-200 rounded-xl p-4 shadow-sm space-y-2 text-xs">
				<span class="block font-semibold text-zinc-700 mb-1">Simulasi Potongan Harga</span>
				<div class="flex justify-between text-zinc-500">
					<span>Harga Produk Misal:</span>
					<span class="font-mono">Rp 150.000</span>
				</div>
				<div class="flex justify-between text-emerald-600">
					<span>Potongan ({diskonPersen || 0}%):</span>
					<span class="font-mono">- Rp {nominalPotongan.toLocaleString('id-ID')}</span>
				</div>
				<div class="pt-2 border-t border-zinc-100 font-bold text-zinc-800 flex justify-between">
					<span>Harga Akhir:</span>
					<span class="font-mono text-teal-700">Rp {hargaAkhir.toLocaleString('id-ID')}</span>
				</div>
			</div>
		</div>
	</div>
</div>
</section>