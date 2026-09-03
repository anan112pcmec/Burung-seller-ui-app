<script lang="ts">
	// ///////////////////////////////////////////////////////////////////////
	// Types
	// ///////////////////////////////////////////////////////////////////////

	type StatusSimpan = 'idle' | 'menyimpan' | 'selesai';
	type StatusFoto = 'kosong' | 'mengunggah' | 'terunggah';

	interface PesanField {
		teks: string;
		tipe: 'sukses' | 'info' | 'gagal';
	}

	// ///////////////////////////////////////////////////////////////////////
	// Konstanta — Dedication/kategori toko
	// TODO: isi asli ada di seller_dedication_enums.CategoryMap, belum
	// ditunjukkan filenya. Ganti daftar ini begitu enum aslinya kebaca.
	// ///////////////////////////////////////////////////////////////////////

	const DAFTAR_DEDICATION: string[] = [
		'Semua Barang',
		'Fashion',
		'Elektronik',
		'Kecantikan & Perawatan',
		'Rumah Tangga',
		'Olahraga & Outdoor',
		'Makanan & Minuman'
	];

	// ///////////////////////////////////////////////////////////////////////
	// Data awal — ganti dengan fetch profil seller yang sedang login
	// ///////////////////////////////////////////////////////////////////////

	const dataAwal = {
		username: 'tokoahmadfauzan',
		nama: 'Ahmad Fauzan',
		email: 'ahmad.fauzan@gmail.com',
		punchline: 'Fashion nyaman, harga bersahabat.',
		deskripsi: 'Toko fashion pria & wanita dengan bahan berkualitas dan pengiriman cepat ke seluruh Indonesia.',
		jamOperasionalMulai: '08:00',
		jamOperasionalSelesai: '20:00',
		dedication: 'Fashion'
	};

	// ///////////////////////////////////////////////////////////////////////
	// State — Foto Profil
	// TODO: field foto TIDAK ADA di struct Seller manapun yang ditunjukkan,
	// kemungkinan besar ini lewat media_services (pola sama seperti dokumen
	// legalitas & banner etalase sebelumnya).
	// ///////////////////////////////////////////////////////////////////////

	let statusFoto = $state<StatusFoto>('terunggah');
	let fotoProfilUrl = $state('https://picsum.photos/seed/toko-ahmad-fauzan/200/200');

	function ubahFotoProfil(e: Event) {
		const target = e.target as HTMLInputElement;
		const file = target.files?.[0];
		if (!file) return;

		statusFoto = 'mengunggah';
		const previewSementara = URL.createObjectURL(file);

		// TODO: ganti dengan alur upload nyata ke media_services (2 tahap:
		// minta slot/ekstensi, lalu PUT file), baru pakai URL final dari server.
		setTimeout(() => {
			fotoProfilUrl = previewSementara;
			statusFoto = 'terunggah';
		}, 700);
	}

	// ///////////////////////////////////////////////////////////////////////
	// State — Data Personal (username, nama, email)
	// ///////////////////////////////////////////////////////////////////////

	let username = $state(dataAwal.username);
	let nama = $state(dataAwal.nama);
	let email = $state(dataAwal.email);

	let statusPersonal = $state<StatusSimpan>('idle');
	let pesanUsername = $state<PesanField | null>(null);
	let pesanNama = $state<PesanField | null>(null);
	let pesanEmail = $state<PesanField | null>(null);
	let saranUsername = $state<string[]>([]);

	let adaPerubahanPersonal = $derived(
		username.trim() !== dataAwal.username || nama.trim() !== dataAwal.nama || email.trim() !== dataAwal.email
	);

	function nilaiAtauNot(baru: string, lama: string): string {
		const dipangkas = baru.trim();
		if (dipangkas === '' || dipangkas === lama) return 'not';
		return dipangkas;
	}

	async function simpanPersonal() {
		if (!adaPerubahanPersonal) return;
		statusPersonal = 'menyimpan';
		pesanUsername = null;
		pesanNama = null;
		pesanEmail = null;
		saranUsername = [];

		// TODO: panggil UpdatePersonalSeller. Field yang gak berubah WAJIB
		// dikirim literal "not", bukan dihapus dari payload (lihat nilaiAtauNot).

		await new Promise((r) => setTimeout(r, 600));

		if (nilaiAtauNot(username, dataAwal.username) !== 'not') {
			if (username.toLowerCase() === 'admin') {
				pesanUsername = { teks: 'Gagal, coba gunakan nama yang disarankan', tipe: 'gagal' };
				saranUsername = [`${username}23`, `${username}88`, `toko_${username}`];
			} else {
				pesanUsername = { teks: 'Berhasil', tipe: 'sukses' };
				dataAwal.username = username.trim();
			}
		}
		if (nilaiAtauNot(nama, dataAwal.nama) !== 'not') {
			pesanNama = { teks: 'Berhasil', tipe: 'sukses' };
			dataAwal.nama = nama.trim();
		}
		if (nilaiAtauNot(email, dataAwal.email) !== 'not') {
			pesanEmail = { teks: 'Berhasil — notifikasi perubahan dikirim ke email lama', tipe: 'sukses' };
			dataAwal.email = email.trim();
		}

		statusPersonal = 'selesai';
		setTimeout(() => (statusPersonal = 'idle'), 2000);
	}

	function pakaiSaranUsername(saran: string) {
		username = saran;
		saranUsername = [];
		pesanUsername = null;
	}

	// ///////////////////////////////////////////////////////////////////////
	// State — Info Publik Toko (punchline, deskripsi, jam operasional, dedication)
	// ///////////////////////////////////////////////////////////////////////

	let punchline = $state(dataAwal.punchline);
	let deskripsi = $state(dataAwal.deskripsi);
	let jamMulai = $state(dataAwal.jamOperasionalMulai);
	let jamSelesai = $state(dataAwal.jamOperasionalSelesai);
	let dedication = $state(dataAwal.dedication);

	let statusPublik = $state<StatusSimpan>('idle');
	let pesanPunchline = $state<PesanField | null>(null);
	let pesanDeskripsi = $state<PesanField | null>(null);
	let pesanJamOperasional = $state<PesanField | null>(null);
	let pesanDedication = $state<PesanField | null>(null);

	let jamOperasionalString = $derived(`${jamMulai} - ${jamSelesai}`);
	let jamOperasionalAwalString = $derived(`${dataAwal.jamOperasionalMulai} - ${dataAwal.jamOperasionalSelesai}`);

	let adaPerubahanPublik = $derived(
		punchline.trim() !== dataAwal.punchline ||
			deskripsi.trim() !== dataAwal.deskripsi ||
			jamOperasionalString !== jamOperasionalAwalString ||
			dedication !== dataAwal.dedication
	);

	async function simpanPublik() {
		if (!adaPerubahanPublik) return;
		statusPublik = 'menyimpan';
		pesanPunchline = null;
		pesanDeskripsi = null;
		pesanJamOperasional = null;
		pesanDedication = null;

		// TODO: panggil UpdateInfoGeneralPublic, sama aturan sentinel "not".

		await new Promise((r) => setTimeout(r, 600));

		if (nilaiAtauNot(punchline, dataAwal.punchline) !== 'not') {
			pesanPunchline = { teks: 'Berhasil', tipe: 'sukses' };
			dataAwal.punchline = punchline.trim();
		}
		if (nilaiAtauNot(deskripsi, dataAwal.deskripsi) !== 'not') {
			pesanDeskripsi = { teks: 'Berhasil', tipe: 'sukses' };
			dataAwal.deskripsi = deskripsi.trim();
		}
		if (jamOperasionalString !== jamOperasionalAwalString) {
			pesanJamOperasional = { teks: 'Berhasil', tipe: 'sukses' };
			dataAwal.jamOperasionalMulai = jamMulai;
			dataAwal.jamOperasionalSelesai = jamSelesai;
		}
		if (dedication !== dataAwal.dedication) {
			pesanDedication = { teks: 'Berhasil mengubah dedication', tipe: 'sukses' };
			dataAwal.dedication = dedication;
		}

		statusPublik = 'selesai';
		setTimeout(() => (statusPublik = 'idle'), 2000);
	}

	function kelasPesan(pesan: PesanField | null): string {
		if (!pesan) return '';
		if (pesan.tipe === 'sukses') return 'text-teal-600';
		if (pesan.tipe === 'gagal') return 'text-rose-500';
		return 'text-zinc-400';
	}

	// ///////////////////////////////////////////////////////////////////////
	// Skor kelengkapan profil — dorongan buat seller ngelengkapin data
	// ///////////////////////////////////////////////////////////////////////

	let itemKelengkapan = $derived([
		{ label: 'Foto profil toko', selesai: statusFoto === 'terunggah' },
		{ label: 'Punchline diisi', selesai: punchline.trim().length > 0 },
		{ label: 'Deskripsi minimal 40 karakter', selesai: deskripsi.trim().length >= 40 },
		{ label: 'Jam operasional diatur', selesai: jamMulai !== '' && jamSelesai !== '' },
		{ label: 'Dedication spesifik dipilih', selesai: dedication !== 'Semua Barang' }
	]);

	let skorPersen = $derived(
		Math.round((itemKelengkapan.filter((i) => i.selesai).length / itemKelengkapan.length) * 100)
	);

	let adaPerubahanBelumTersimpan = $derived(adaPerubahanPersonal || adaPerubahanPublik);
</script>

<section id="profiling" class="w-full bg-white p-4 sm:p-6 lg:p-8 text-slate-950 pb-20">
	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- HEADER -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="flex items-center justify-between flex-wrap gap-3 pb-6 mb-6 border-b border-zinc-800/10">
		<div>
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
				PROFIL SELLER
			</span>
			<h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
				Profiling
			</h1>
		</div>
		<div class="flex items-center gap-2">
			<div class="relative w-8 h-8">
				<svg class="w-full h-full -rotate-90" viewBox="0 0 36 36">
					<circle cx="18" cy="18" r="15.915" fill="none" class="stroke-zinc-100" stroke-width="3.5" />
					<circle
						cx="18" cy="18" r="15.915" fill="none"
						class={skorPersen === 100 ? 'stroke-teal-600' : 'stroke-slate-950'}
						stroke-width="3.5"
						stroke-dasharray="{skorPersen} {100 - skorPersen}"
						stroke-linecap="round"
					/>
				</svg>
				<span class="absolute inset-0 flex items-center justify-center text-[8px] font-bold font-mono">{skorPersen}%</span>
			</div>
			<div class="leading-tight">
				<p class="text-[10px] font-bold text-zinc-800">Profil {skorPersen === 100 ? 'Lengkap' : 'Belum Lengkap'}</p>
				<p class="text-[9px] text-zinc-400">Toko lebih meyakinkan kalau profilnya lengkap</p>
			</div>
		</div>
	</div>

	<div class="grid grid-cols-1 lg:grid-cols-12 gap-6">
		<!-- ///////////////////////////////////////////////////////////// -->
		<!-- KOLOM KIRI — form -->
		<!-- ///////////////////////////////////////////////////////////// -->
		<div class="lg:col-span-7 flex flex-col gap-6">
			<!-- FOTO PROFIL -->
			<div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					Foto Profil Toko
				</span>
				<div class="mt-3 flex items-center gap-4">
					<div class="relative w-16 h-16 sm:w-20 sm:h-20 flex-shrink-0 rounded-full overflow-hidden bg-zinc-100 border border-zinc-200">
						{#if statusFoto === 'mengunggah'}
							<div class="absolute inset-0 flex items-center justify-center bg-white/70">
								<svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5 text-zinc-400 animate-spin" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
									<path d="M21 12a9 9 0 1 1-6.219-8.56"/>
								</svg>
							</div>
						{:else}
							<img src={fotoProfilUrl} alt="Foto profil toko" class="w-full h-full object-cover" />
						{/if}
					</div>
					<div>
						<label class="inline-flex items-center gap-1.5 text-[10px] font-bold uppercase tracking-wider text-slate-950 border border-zinc-300 rounded px-3 py-1.5 cursor-pointer hover:bg-zinc-50 transition-colors">
							Ganti Foto
							<input type="file" accept="image/*" class="hidden" onchange={ubahFotoProfil} />
						</label>
						<p class="text-[9px] text-zinc-400 mt-1.5 leading-relaxed">
							Format JPG/PNG, disarankan rasio 1:1 biar gak terpotong aneh di berbagai tempat.
						</p>
					</div>
				</div>
			</div>

			<!-- DATA PERSONAL -->
			<div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					Data Personal
				</span>
				<p class="text-[10px] text-zinc-400 mt-1 leading-relaxed">
					Username, nama, dan email akun kamu. Ganti email akan mengirim notifikasi ke alamat email lama.
				</p>

				<div class="mt-4 flex flex-col gap-4">
					<label class="flex flex-col gap-1.5">
						<span class="text-[10px] font-medium text-slate-600">Username</span>
						<input
							type="text"
							bind:value={username}
							class="border border-zinc-300 rounded-md px-3 py-2 text-xs font-mono text-slate-900 focus:outline-none focus:border-slate-950 transition-colors"
						/>
						{#if pesanUsername}
							<span class="text-[9px] {kelasPesan(pesanUsername)}">{pesanUsername.teks}</span>
						{/if}
						{#if saranUsername.length > 0}
							<div class="flex flex-wrap gap-1.5 mt-1">
								{#each saranUsername as saran}
									<button
										type="button"
										onclick={() => pakaiSaranUsername(saran)}
										class="px-2 py-1 rounded bg-zinc-100 hover:bg-zinc-200 text-[10px] font-mono text-zinc-700 transition-colors"
									>
										{saran}
									</button>
								{/each}
							</div>
						{/if}
					</label>

					<label class="flex flex-col gap-1.5">
						<span class="text-[10px] font-medium text-slate-600">Nama</span>
						<input
							type="text"
							bind:value={nama}
							class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 focus:outline-none focus:border-slate-950 transition-colors"
						/>
						{#if pesanNama}
							<span class="text-[9px] {kelasPesan(pesanNama)}">{pesanNama.teks}</span>
						{/if}
					</label>

					<label class="flex flex-col gap-1.5">
						<span class="text-[10px] font-medium text-slate-600">Email</span>
						<input
							type="email"
							bind:value={email}
							class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 focus:outline-none focus:border-slate-950 transition-colors"
						/>
						{#if pesanEmail}
							<span class="text-[9px] {kelasPesan(pesanEmail)}">{pesanEmail.teks}</span>
						{/if}
					</label>
				</div>

				<div class="flex items-center gap-3 mt-4">
					<button
						type="button"
						disabled={!adaPerubahanPersonal || statusPersonal === 'menyimpan'}
						onclick={simpanPersonal}
						class="px-4 py-2 bg-slate-950 text-white text-[10px] font-bold uppercase tracking-wider rounded-sm hover:bg-slate-800 transition-colors disabled:opacity-30 disabled:cursor-not-allowed"
					>
						{statusPersonal === 'menyimpan' ? 'Menyimpan...' : 'Simpan Data Personal'}
					</button>
				</div>
			</div>

			<!-- INFO PUBLIK TOKO -->
			<div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5">
				<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
					Info Publik Toko
				</span>
				<p class="text-[10px] text-zinc-400 mt-1 leading-relaxed">
					Informasi ini yang dilihat pembeli saat mengunjungi halaman toko kamu — lihat pratinjau di sisi kanan.
				</p>

				<div class="mt-4 flex flex-col gap-4">
					<label class="flex flex-col gap-1.5">
						<div class="flex items-center justify-between">
							<span class="text-[10px] font-medium text-slate-600">Punchline</span>
							<span class="text-[9px] font-mono text-zinc-400">{punchline.length}/100</span>
						</div>
						<input
							type="text"
							bind:value={punchline}
							maxlength="100"
							placeholder="Tagline singkat toko kamu"
							class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 placeholder-zinc-400 focus:outline-none focus:border-teal-600 transition-colors"
						/>
						{#if pesanPunchline}
							<span class="text-[9px] {kelasPesan(pesanPunchline)}">{pesanPunchline.teks}</span>
						{/if}
					</label>

					<label class="flex flex-col gap-1.5">
						<div class="flex items-center justify-between">
							<span class="text-[10px] font-medium text-slate-600">Deskripsi Toko</span>
							<span class="text-[9px] font-mono {deskripsi.trim().length >= 40 ? 'text-teal-600' : 'text-zinc-400'}">
								{deskripsi.trim().length} karakter
							</span>
						</div>
						<textarea
							bind:value={deskripsi}
							rows={3}
							class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 focus:outline-none focus:border-teal-600 transition-colors resize-none"
						></textarea>
						{#if deskripsi.trim().length < 40}
							<span class="text-[9px] text-zinc-400">Tambah sedikit lagi biar pembeli lebih yakin sama tokomu.</span>
						{/if}
						{#if pesanDeskripsi}
							<span class="text-[9px] {kelasPesan(pesanDeskripsi)}">{pesanDeskripsi.teks}</span>
						{/if}
					</label>

					<div class="flex flex-col gap-1.5">
						<span class="text-[10px] font-medium text-slate-600">Jam Operasional</span>
						<div class="flex items-center gap-2">
							<input
								type="time"
								bind:value={jamMulai}
								class="border border-zinc-300 rounded-md px-3 py-2 text-xs font-mono text-slate-900 focus:outline-none focus:border-teal-600 transition-colors"
							/>
							<span class="text-zinc-400 text-xs">—</span>
							<input
								type="time"
								bind:value={jamSelesai}
								class="border border-zinc-300 rounded-md px-3 py-2 text-xs font-mono text-slate-900 focus:outline-none focus:border-teal-600 transition-colors"
							/>
						</div>
						{#if pesanJamOperasional}
							<span class="text-[9px] {kelasPesan(pesanJamOperasional)}">{pesanJamOperasional.teks}</span>
						{/if}
					</div>

					<label class="flex flex-col gap-1.5">
						<span class="text-[10px] font-medium text-slate-600">Dedication (Fokus Toko)</span>
						<select
							bind:value={dedication}
							class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 focus:outline-none focus:border-teal-600 transition-colors"
						>
							{#each DAFTAR_DEDICATION as opt}
								<option value={opt}>{opt}</option>
							{/each}
						</select>
						{#if pesanDedication}
							<span class="text-[9px] {kelasPesan(pesanDedication)}">{pesanDedication.teks}</span>
						{/if}
					</label>
				</div>

				<div class="flex items-center gap-3 mt-4">
					<button
						type="button"
						disabled={!adaPerubahanPublik || statusPublik === 'menyimpan'}
						onclick={simpanPublik}
						class="px-4 py-2 bg-slate-950 text-white text-[10px] font-bold uppercase tracking-wider rounded-sm hover:bg-slate-800 transition-colors disabled:opacity-30 disabled:cursor-not-allowed"
					>
						{statusPublik === 'menyimpan' ? 'Menyimpan...' : 'Simpan Info Toko'}
					</button>
				</div>
			</div>
		</div>

		<!-- ///////////////////////////////////////////////////////////// -->
		<!-- KOLOM KANAN — preview live & kelengkapan (sticky) -->
		<!-- ///////////////////////////////////////////////////////////// -->
		<div class="lg:col-span-5">
			<div class="lg:sticky lg:top-6 flex flex-col gap-4">
				<!-- PREVIEW TOKO -->
				<div class="border border-zinc-800/20 rounded-sm overflow-hidden">
					<div class="px-4 pt-3">
						<span class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
							Pratinjau Toko
						</span>
					</div>
					<div class="p-4 sm:p-5">
						<div class="flex items-center gap-3">
							<div class="w-12 h-12 rounded-full overflow-hidden bg-zinc-100 border border-zinc-200 flex-shrink-0">
								<img src={fotoProfilUrl} alt="Preview foto profil" class="w-full h-full object-cover" />
							</div>
							<div class="min-w-0">
								<p class="text-sm font-bold text-zinc-900 truncate">{nama || 'Nama Toko'}</p>
								<p class="text-[10px] text-zinc-400">@{username || 'username'}</p>
							</div>
						</div>

						<p class="text-xs text-zinc-700 mt-3 leading-snug italic">
							"{punchline || 'Belum ada punchline...'}"
						</p>

						<p class="text-[11px] text-zinc-500 mt-2 leading-relaxed">
							{deskripsi || 'Belum ada deskripsi toko...'}
						</p>

						<div class="flex items-center gap-2 mt-3 pt-3 border-t border-zinc-100 flex-wrap">
							<span class="px-2 py-0.5 rounded text-[9px] font-medium uppercase tracking-wider bg-teal-50 text-teal-700">
								{dedication}
							</span>
							<span class="text-[9px] text-zinc-400 font-mono">
								Buka {jamMulai || '--:--'} - {jamSelesai || '--:--'}
							</span>
						</div>
					</div>
				</div>

				<!-- SKOR KELENGKAPAN -->
				<div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5">
					<div class="flex items-center justify-between mb-2">
						<span class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
							Kelengkapan Profil
						</span>
						<span class="text-[10px] font-bold font-mono {skorPersen === 100 ? 'text-teal-600' : 'text-zinc-700'}">
							{skorPersen}%
						</span>
					</div>
					<div class="w-full h-1.5 bg-zinc-100 rounded-full overflow-hidden mb-3">
						<div
							class="h-full rounded-full transition-all duration-300 {skorPersen === 100 ? 'bg-teal-600' : 'bg-slate-950'}"
							style:width="{skorPersen}%"
						></div>
					</div>
					<div class="flex flex-col gap-1.5">
						{#each itemKelengkapan as item}
							<div class="flex items-center gap-2 text-[10px]">
								{#if item.selesai}
									<svg xmlns="http://www.w3.org/2000/svg" class="w-3.5 h-3.5 text-teal-600 flex-shrink-0" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
										<path d="M20 6 9 17l-5-5"/>
									</svg>
									<span class="text-zinc-400 line-through">{item.label}</span>
								{:else}
									<span class="w-3.5 h-3.5 rounded-full border-2 border-zinc-300 flex-shrink-0"></span>
									<span class="text-zinc-700">{item.label}</span>
								{/if}
							</div>
						{/each}
					</div>
				</div>
			</div>
		</div>
	</div>

	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- BANNER PERUBAHAN BELUM TERSIMPAN -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	{#if adaPerubahanBelumTersimpan}
		<div class="fixed bottom-0 left-0 right-0 bg-slate-950 text-white px-4 sm:px-6 lg:px-8 py-3 flex items-center justify-between gap-3 z-10">
			<span class="text-[11px]">Ada perubahan yang belum disimpan di formulir.</span>
			<span class="text-[9px] text-zinc-400 hidden sm:inline">Gunakan tombol simpan di masing-masing bagian.</span>
		</div>
	{/if}
</section>