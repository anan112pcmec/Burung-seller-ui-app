<script lang="ts">
	// ///////////////////////////////////////////////////////////////////////
	// Types
	// ///////////////////////////////////////////////////////////////////////

	type LangkahPassword = 'form' | 'otp' | 'selesai';
	type AksiRekening = 'tambah' | 'edit' | 'hapus' | null;

	interface Rekening {
		id: number;
		namaBank: string;
		nomorRekening: string;
		pemilikRekening: string;
		isDefault: boolean;
	}

	// ///////////////////////////////////////////////////////////////////////
	// Konstanta — daftar bank harus cocok whitelist backend (nama_bank.BankMap)
	// ganti/lengkapi sesuai isi enum aslinya, ini contoh bank umum Indonesia
	// ///////////////////////////////////////////////////////////////////////

	const DAFTAR_BANK: string[] = [
		'BCA',
		'Mandiri',
		'BNI',
		'BRI',
		'CIMB Niaga',
		'Permata',
		'Danamon',
		'BTN',
		'Bank Syariah Indonesia (BSI)',
		'OCBC NISP'
	];

	const DURASI_OTP_DETIK = 180; // 3 menit — sesuai TTL Redis di backend

	// ///////////////////////////////////////////////////////////////////////
	// State — Ubah Password
	// ///////////////////////////////////////////////////////////////////////

	let langkahPassword = $state<LangkahPassword>('form');
	let passwordLama = $state('');
	let passwordBaru = $state('');
	let passwordKonfirmasi = $state('');
	let kodeOtp = $state('');
	let sisaDetikOtp = $state(DURASI_OTP_DETIK);
	let otpKadaluarsa = $derived(sisaDetikOtp <= 0);

	let cocokKonfirmasi = $derived(passwordBaru.length > 0 && passwordBaru === passwordKonfirmasi);
	let panjangCukup = $derived(passwordBaru.length >= 8);
	let formPasswordValid = $derived(passwordLama.length > 0 && panjangCukup && cocokKonfirmasi);

	let mmssOtp = $derived(() => {
		const m = Math.floor(sisaDetikOtp / 60);
		const s = sisaDetikOtp % 60;
		return `${m}:${s.toString().padStart(2, '0')}`;
	});

	let timerHandle: ReturnType<typeof setInterval> | undefined;

	function mulaiTimerOtp() {
		sisaDetikOtp = DURASI_OTP_DETIK;
		clearInterval(timerHandle);
		timerHandle = setInterval(() => {
			if (sisaDetikOtp > 0) sisaDetikOtp -= 1;
			else clearInterval(timerHandle);
		}, 1000);
	}

	function ajukanUbahPassword() {
		// TODO: panggil PreUbahPasswordSeller — verifikasi password lama di server,
		// server generate hash password baru & simpan sementara di Redis, lalu
		// kirim OTP ke email seller. Jangan ubah password di sisi client sama sekali.
		langkahPassword = 'otp';
		kodeOtp = '';
		mulaiTimerOtp();
	}

	function validasiOtpPassword() {
		// TODO: panggil ValidateUbahPasswordSeller dengan kodeOtp.
		// Password baru BENERAN diterapkan di sini, bukan di langkah sebelumnya.
		clearInterval(timerHandle);
		langkahPassword = 'selesai';
	}

	function batalUbahPassword() {
		clearInterval(timerHandle);
		langkahPassword = 'form';
		passwordLama = '';
		passwordBaru = '';
		passwordKonfirmasi = '';
		kodeOtp = '';
	}

	// ///////////////////////////////////////////////////////////////////////
	// State — Rekening Bank
	// ///////////////////////////////////////////////////////////////////////

	let rekeningList = $state<Rekening[]>([
		{ id: 1, namaBank: 'BCA', nomorRekening: '4520019283', pemilikRekening: 'Ahmad Fauzan', isDefault: true },
		{ id: 2, namaBank: 'Mandiri', nomorRekening: '1330008812345', pemilikRekening: 'Ahmad Fauzan', isDefault: false }
	]);

	let aksiRekening = $state<AksiRekening>(null);
	let idRekeningAktif = $state<number | null>(null);

	let formRekening = $state<Record<string, string>>({
		namaBank: DAFTAR_BANK[0],
		nomorRekening: '',
		pemilikRekening: ''
	});

	let konfirmasiHapus = $state('');

	function maskNorek(nomor: string): string {
		if (nomor.length <= 4) return nomor;
		return `${'•'.repeat(Math.max(nomor.length - 4, 0))}${nomor.slice(-4)}`;
	}

	function bukaTambahRekening() {
		aksiRekening = 'tambah';
		idRekeningAktif = null;
		formRekening = { namaBank: DAFTAR_BANK[0], nomorRekening: '', pemilikRekening: '' };
	}

	function bukaEditRekening(rek: Rekening) {
		aksiRekening = 'edit';
		idRekeningAktif = rek.id;
		formRekening = { namaBank: rek.namaBank, nomorRekening: rek.nomorRekening, pemilikRekening: rek.pemilikRekening };
	}

	function bukaHapusRekening(rek: Rekening) {
		aksiRekening = 'hapus';
		idRekeningAktif = rek.id;
		konfirmasiHapus = '';
	}

	function tutupPanelRekening() {
		aksiRekening = null;
		idRekeningAktif = null;
		konfirmasiHapus = '';
	}

	function simpanRekening() {
		// TODO: panggil TambahRekeningSeller / EditRekeningSeller sesuai aksiRekening.
		// nama bank WAJIB salah satu dari whitelist backend, kalau tidak backend
		// akan menolak dengan 406 "nama bank tidak diterima".
		if (aksiRekening === 'tambah') {
			const rekeningPertama = rekeningList.length === 0;
			rekeningList.push({
				id: Date.now(),
				namaBank: formRekening.namaBank,
				nomorRekening: formRekening.nomorRekening,
				pemilikRekening: formRekening.pemilikRekening,
				isDefault: rekeningPertama
			});
		} else if (aksiRekening === 'edit' && idRekeningAktif !== null) {
			const rek = rekeningList.find((r) => r.id === idRekeningAktif);
			if (rek) {
				rek.namaBank = formRekening.namaBank;
				rek.nomorRekening = formRekening.nomorRekening;
				rek.pemilikRekening = formRekening.pemilikRekening;
			}
		}
		tutupPanelRekening();
	}

	function jadikanDefault(id: number) {
		// TODO: panggil SetDefaultRekeningSeller — operasi ini transactional di
		// backend (unset default lama, set default baru), jadi jangan optimis-update
		// dua-duanya di client tanpa fallback kalau request gagal.
		for (const rek of rekeningList) rek.isDefault = rek.id === id;
	}

	function hapusRekeningTerkonfirmasi() {
		if (idRekeningAktif === null) return;
		const rek = rekeningList.find((r) => r.id === idRekeningAktif);
		if (!rek || konfirmasiHapus !== rek.nomorRekening) return;
		// TODO: panggil HapusRekeningSeller — backend re-verifikasi nomor_rekening
		// ini lagi di sisi server, jadi ini bukan cuma validasi kosmetik di client.
		rekeningList = rekeningList.filter((r) => r.id !== idRekeningAktif);
		tutupPanelRekening();
	}

	let rekeningYangDihapus = $derived(rekeningList.find((r) => r.id === idRekeningAktif) ?? null);
	let konfirmasiHapusCocok = $derived(rekeningYangDihapus !== null && konfirmasiHapus === rekeningYangDihapus.nomorRekening);
</script>

<section id="keamanan-kredensial" class="w-full bg-white p-4 sm:p-6 lg:p-8 text-slate-950">
	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- HEADER -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="pb-5 mb-6 border-b border-zinc-800/10">
		<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
			KEAMANAN &amp; KREDENSIAL
		</span>
		<h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
			Password &amp; Rekening Pencairan Dana
		</h1>
		<p class="mt-2 text-[11px] text-slate-500 leading-relaxed max-w-2xl">
			Halaman ini menyangkut akses akun dan tujuan pencairan dana hasil penjualan kamu. Sebagian perubahan di sini memerlukan verifikasi tambahan lewat email dan tidak dapat langsung dibatalkan setelah diproses. Mohon dibaca dengan teliti sebelum melanjutkan.
		</p>
	</div>

	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- SECTION: UBAH PASSWORD -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5 mb-6">
		<div class="flex items-center justify-between flex-wrap gap-2">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				Ubah Password
			</span>
			<div class="flex items-center gap-1.5 text-[9px] font-mono text-slate-400 uppercase tracking-wider">
				<span class={langkahPassword === 'form' ? 'text-slate-950 font-bold' : ''}>1. Verifikasi</span>
				<span>→</span>
				<span class={langkahPassword === 'otp' ? 'text-slate-950 font-bold' : ''}>2. Kode OTP</span>
				<span>→</span>
				<span class={langkahPassword === 'selesai' ? 'text-teal-700 font-bold' : ''}>3. Selesai</span>
			</div>
		</div>

		{#if langkahPassword === 'form'}
			<div class="mt-3 bg-zinc-50 border border-zinc-200 rounded-md p-3 text-[10px] text-zinc-500 leading-relaxed">
				<strong class="text-zinc-700">Perhatian:</strong> setelah kamu mengajukan perubahan di bawah ini, password
				<strong>belum langsung berubah</strong>. Kami akan mengirimkan kode verifikasi 6 digit ke alamat email
				yang terdaftar pada akun ini, dan kode tersebut hanya berlaku selama 3 menit sejak dikirim. Password
				lama kamu tetap aktif sampai proses verifikasi selesai.
			</div>

			<div class="mt-4 grid grid-cols-1 sm:grid-cols-2 gap-4 max-w-xl">
				<label class="flex flex-col gap-1.5 sm:col-span-2">
					<span class="text-[10px] font-medium text-slate-600">Password Saat Ini</span>
					<input
						type="password"
						bind:value={passwordLama}
						placeholder="Masukkan password yang sedang aktif"
						class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 placeholder-zinc-400 focus:outline-none focus:border-slate-950 transition-colors"
					/>
				</label>

				<label class="flex flex-col gap-1.5">
					<span class="text-[10px] font-medium text-slate-600">Password Baru</span>
					<input
						type="password"
						bind:value={passwordBaru}
						placeholder="Minimal 8 karakter"
						class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 placeholder-zinc-400 focus:outline-none focus:border-slate-950 transition-colors"
					/>
					{#if passwordBaru.length > 0 && !panjangCukup}
						<span class="text-[9px] text-rose-500">Password baru minimal 8 karakter.</span>
					{/if}
				</label>

				<label class="flex flex-col gap-1.5">
					<span class="text-[10px] font-medium text-slate-600">Konfirmasi Password Baru</span>
					<input
						type="password"
						bind:value={passwordKonfirmasi}
						placeholder="Ulangi password baru"
						class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 placeholder-zinc-400 focus:outline-none focus:border-slate-950 transition-colors"
					/>
					{#if passwordKonfirmasi.length > 0 && !cocokKonfirmasi}
						<span class="text-[9px] text-rose-500">Konfirmasi belum sama dengan password baru.</span>
					{/if}
				</label>
			</div>

			<div class="mt-4 flex justify-end">
				<button
					type="button"
					disabled={!formPasswordValid}
					onclick={ajukanUbahPassword}
					class="px-5 py-2.5 bg-slate-950 text-white text-[11px] font-bold uppercase tracking-wider rounded-sm hover:bg-slate-800 transition-colors disabled:opacity-30 disabled:cursor-not-allowed"
				>
					Kirim Kode Verifikasi
				</button>
			</div>
		{:else if langkahPassword === 'otp'}
			<div class="mt-3 bg-zinc-50 border border-zinc-200 rounded-md p-3 text-[10px] text-zinc-500 leading-relaxed">
				Kode verifikasi 6 digit telah dikirim ke email yang terdaftar pada akun ini. Masukkan kode tersebut
				di bawah untuk menyelesaikan perubahan password. Kode yang sudah kadaluarsa tidak dapat digunakan
				lagi dan proses harus diulang dari awal. Jangan bagikan kode ini kepada siapa pun, termasuk pihak
				yang mengaku sebagai staf resmi kami — kami tidak akan pernah meminta kode OTP kamu lewat telepon,
				chat, maupun media sosial.
			</div>

			<div class="mt-4 flex flex-col sm:flex-row sm:items-end gap-4 max-w-xl">
				<label class="flex flex-col gap-1.5 flex-1">
					<span class="text-[10px] font-medium text-slate-600">Kode OTP</span>
					<input
						type="text"
						inputmode="numeric"
						maxlength="6"
						bind:value={kodeOtp}
						disabled={otpKadaluarsa}
						placeholder="6 digit dari email"
						class="border border-zinc-300 rounded-md px-3 py-2 text-sm font-mono tracking-[0.3em] text-slate-900 placeholder-zinc-400 focus:outline-none focus:border-slate-950 transition-colors disabled:bg-zinc-100 disabled:text-zinc-400"
					/>
				</label>
				<div class="text-[11px] font-mono {otpKadaluarsa ? 'text-rose-500' : 'text-slate-500'} pb-2.5">
					{otpKadaluarsa ? 'Kode kadaluarsa' : `Berlaku ${mmssOtp()}`}
				</div>
			</div>

			{#if otpKadaluarsa}
				<p class="mt-2 text-[10px] text-rose-500">
					Kode verifikasi sudah tidak berlaku. Silakan ulangi proses dari langkah pertama untuk mendapatkan kode baru.
				</p>
			{/if}

			<div class="mt-4 flex flex-wrap items-center gap-3">
				<button
					type="button"
					disabled={otpKadaluarsa || kodeOtp.length !== 6}
					onclick={validasiOtpPassword}
					class="px-5 py-2.5 bg-slate-950 text-white text-[11px] font-bold uppercase tracking-wider rounded-sm hover:bg-slate-800 transition-colors disabled:opacity-30 disabled:cursor-not-allowed"
				>
					Validasi &amp; Ubah Password
				</button>
				<button
					type="button"
					onclick={batalUbahPassword}
					class="text-[10px] font-medium text-slate-500 underline"
				>
					Batalkan &amp; Kembali
				</button>
			</div>
		{:else}
			<div class="mt-4 flex items-start gap-3 bg-teal-50 border border-teal-200 rounded-md p-4">
				<svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4 text-teal-600 flex-shrink-0 mt-0.5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
					<path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><path d="m9 11 3 3L22 4"/>
				</svg>
				<div>
					<p class="text-xs font-bold text-teal-800">Password berhasil diubah</p>
					<p class="text-[11px] text-teal-700 mt-1 leading-relaxed">
						Kamu akan tetap login di perangkat ini, tapi disarankan untuk login ulang di perangkat lain
						yang mungkin masih menggunakan sesi lama, demi keamanan akun kamu.
					</p>
					<button type="button" onclick={batalUbahPassword} class="mt-2 text-[10px] font-medium text-teal-800 underline">
						Ubah password lagi
					</button>
				</div>
			</div>
		{/if}
	</div>

	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- SECTION: REKENING BANK -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5">
		<div class="flex items-center justify-between flex-wrap gap-2">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				Rekening Pencairan Dana
			</span>
			<button
				type="button"
				onclick={bukaTambahRekening}
				class="text-[10px] font-bold uppercase tracking-wider text-slate-950 border border-zinc-300 rounded px-3 py-1.5 hover:bg-zinc-50 transition-colors"
			>
				+ Tambah Rekening
			</button>
		</div>

		<div class="mt-3 bg-zinc-50 border border-zinc-200 rounded-md p-3 text-[10px] text-zinc-500 leading-relaxed">
			Rekening yang terdaftar di sini akan digunakan sebagai tujuan pencairan dana hasil penjualan kamu.
			Pastikan nama pemilik rekening sesuai dengan identitas yang terdaftar pada akun — kesalahan nama atau
			nomor rekening dapat menyebabkan pencairan dana gagal atau tertunda, dan proses koreksinya bisa memakan
			waktu beberapa hari kerja.
		</div>

		<div class="mt-4 flex flex-col gap-3">
			{#each rekeningList as rek (rek.id)}
				<div class="border border-zinc-200 rounded-md p-3">
					<div class="flex items-start justify-between gap-3 flex-wrap">
						<div>
							<div class="flex items-center gap-2 flex-wrap">
								<p class="text-xs font-bold text-zinc-800">{rek.namaBank}</p>
								{#if rek.isDefault}
									<span class="px-1.5 py-0.5 rounded text-[8px] font-medium uppercase tracking-wider bg-teal-50 text-teal-700">Default</span>
								{/if}
							</div>
							<p class="text-[11px] font-mono text-zinc-500 mt-0.5">{maskNorek(rek.nomorRekening)}</p>
							<p class="text-[10px] text-zinc-400 mt-0.5">a.n. {rek.pemilikRekening}</p>
						</div>

						<div class="flex items-center gap-3 flex-wrap">
							{#if !rek.isDefault}
								<button type="button" onclick={() => jadikanDefault(rek.id)} class="text-[9px] font-medium text-slate-600 underline">
									Jadikan Default
								</button>
							{/if}
							<button type="button" onclick={() => bukaEditRekening(rek)} class="text-[9px] font-medium text-slate-600 underline">
								Edit
							</button>
							<button type="button" onclick={() => bukaHapusRekening(rek)} class="text-[9px] font-medium text-rose-500 underline">
								Hapus
							</button>
						</div>
					</div>

					{#if aksiRekening === 'hapus' && idRekeningAktif === rek.id}
						<div class="mt-3 pt-3 border-t border-dashed border-zinc-200">
							<div class="bg-rose-50 border border-rose-200 rounded-md p-3 text-[10px] text-rose-700 leading-relaxed">
								Tindakan ini <strong>tidak dapat dibatalkan</strong>. Untuk melanjutkan, ketik ulang nomor
								rekening <strong class="font-mono">{rek.nomorRekening}</strong> persis seperti yang tertera
								di kolom di bawah ini sebagai bentuk konfirmasi.
							</div>
							<input
								type="text"
								bind:value={konfirmasiHapus}
								placeholder="Ketik ulang nomor rekening"
								class="mt-2 w-full max-w-xs border border-zinc-300 rounded-md px-3 py-2 text-xs font-mono text-slate-900 placeholder-zinc-400 focus:outline-none focus:border-rose-500 transition-colors"
							/>
							<div class="mt-2 flex items-center gap-3">
								<button
									type="button"
									disabled={!konfirmasiHapusCocok}
									onclick={hapusRekeningTerkonfirmasi}
									class="px-4 py-2 bg-rose-600 text-white text-[10px] font-bold uppercase tracking-wider rounded-sm hover:bg-rose-700 transition-colors disabled:opacity-30 disabled:cursor-not-allowed"
								>
									Hapus Rekening Ini
								</button>
								<button type="button" onclick={tutupPanelRekening} class="text-[10px] font-medium text-slate-500 underline">
									Batal
								</button>
							</div>
						</div>
					{/if}

					{#if aksiRekening === 'edit' && idRekeningAktif === rek.id}
						<div class="mt-3 pt-3 border-t border-dashed border-zinc-200 grid grid-cols-1 sm:grid-cols-3 gap-3">
							<label class="flex flex-col gap-1.5">
								<span class="text-[10px] font-medium text-slate-600">Nama Bank</span>
								<select
									bind:value={formRekening.namaBank}
									class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 focus:outline-none focus:border-slate-950 transition-colors"
								>
									{#each DAFTAR_BANK as bank}
										<option value={bank}>{bank}</option>
									{/each}
								</select>
							</label>
							<label class="flex flex-col gap-1.5">
								<span class="text-[10px] font-medium text-slate-600">Nomor Rekening</span>
								<input
									type="text"
									inputmode="numeric"
									bind:value={formRekening.nomorRekening}
									class="border border-zinc-300 rounded-md px-3 py-2 text-xs font-mono text-slate-900 focus:outline-none focus:border-slate-950 transition-colors"
								/>
							</label>
							<label class="flex flex-col gap-1.5">
								<span class="text-[10px] font-medium text-slate-600">Nama Pemilik Rekening</span>
								<input
									type="text"
									bind:value={formRekening.pemilikRekening}
									class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 focus:outline-none focus:border-slate-950 transition-colors"
								/>
							</label>
							<div class="sm:col-span-3 flex items-center gap-3">
								<button type="button" onclick={simpanRekening} class="px-4 py-2 bg-slate-950 text-white text-[10px] font-bold uppercase tracking-wider rounded-sm hover:bg-slate-800 transition-colors">
									Simpan Perubahan
								</button>
								<button type="button" onclick={tutupPanelRekening} class="text-[10px] font-medium text-slate-500 underline">
									Batal
								</button>
							</div>
						</div>
					{/if}
				</div>
			{/each}

			{#if rekeningList.length === 0}
				<p class="text-[11px] text-zinc-400 text-center py-6">Belum ada rekening terdaftar. Tambahkan minimal satu rekening agar dana penjualan bisa dicairkan.</p>
			{/if}
		</div>

		{#if aksiRekening === 'tambah'}
			<div class="mt-4 border border-zinc-200 rounded-md p-3">
				<p class="text-[10px] font-bold uppercase tracking-wider text-zinc-500 mb-3">Tambah Rekening Baru</p>
				<div class="bg-zinc-50 border border-zinc-200 rounded-md p-3 mb-3 text-[10px] text-zinc-500 leading-relaxed">
					Nama bank hanya dapat dipilih dari daftar yang tersedia. Nama pemilik rekening wajib sama persis
					dengan identitas terdaftar — rekening atas nama pihak lain (misalnya keluarga atau rekan) berisiko
					ditolak sistem verifikasi pencairan dana.
				</div>
				<div class="grid grid-cols-1 sm:grid-cols-3 gap-3">
					<label class="flex flex-col gap-1.5">
						<span class="text-[10px] font-medium text-slate-600">Nama Bank</span>
						<select
							bind:value={formRekening.namaBank}
							class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 focus:outline-none focus:border-slate-950 transition-colors"
						>
							{#each DAFTAR_BANK as bank}
								<option value={bank}>{bank}</option>
							{/each}
						</select>
					</label>
					<label class="flex flex-col gap-1.5">
						<span class="text-[10px] font-medium text-slate-600">Nomor Rekening</span>
						<input
							type="text"
							inputmode="numeric"
							bind:value={formRekening.nomorRekening}
							placeholder="cth. 1234567890"
							class="border border-zinc-300 rounded-md px-3 py-2 text-xs font-mono text-slate-900 placeholder-zinc-400 focus:outline-none focus:border-slate-950 transition-colors"
						/>
					</label>
					<label class="flex flex-col gap-1.5">
						<span class="text-[10px] font-medium text-slate-600">Nama Pemilik Rekening</span>
						<input
							type="text"
							bind:value={formRekening.pemilikRekening}
							placeholder="Sesuai identitas terdaftar"
							class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 placeholder-zinc-400 focus:outline-none focus:border-slate-950 transition-colors"
						/>
					</label>
				</div>
				<div class="mt-3 flex items-center gap-3">
					<button
						type="button"
						disabled={!formRekening.nomorRekening || !formRekening.pemilikRekening}
						onclick={simpanRekening}
						class="px-4 py-2 bg-slate-950 text-white text-[10px] font-bold uppercase tracking-wider rounded-sm hover:bg-slate-800 transition-colors disabled:opacity-30 disabled:cursor-not-allowed"
					>
						Simpan Rekening
					</button>
					<button type="button" onclick={tutupPanelRekening} class="text-[10px] font-medium text-slate-500 underline">
						Batal
					</button>
				</div>
			</div>
		{/if}
	</div>
</section>