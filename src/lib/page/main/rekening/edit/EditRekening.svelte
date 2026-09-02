<script lang="ts">
	// ///////////////////////////////////////////////////////////////////////
	// Types
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

	// ///////////////////////////////////////////////////////////////////////
	// Data rekening yang sedang diedit — HANYA field non-rahasia yang boleh
	// di-prefill. NomorRekening SENGAJA tidak dimuat ke form sama sekali,
	// bahkan dalam bentuk masked, biar gak ada versi lama yang "kebocor" ke UI.
	// ///////////////////////////////////////////////////////////////////////

	const idRekeningSeller = 1;

	let namaBank = $state('BCA');
	let pemilikRekening = $state('Ahmad Fauzan');
	let isDefault = $state(true);

	let nomorRekeningBaru = $state('');
	let konfirmasiNomorRekening = $state('');

	// ///////////////////////////////////////////////////////////////////////
	// Validasi
	// ///////////////////////////////////////////////////////////////////////

	let nomorRekeningDiisi = $derived(nomorRekeningBaru.trim().length > 0);
	let nomorRekeningCocok = $derived(nomorRekeningDiisi && nomorRekeningBaru === konfirmasiNomorRekening);
	let konfirmasiTidakKosongTapiBeda = $derived(konfirmasiNomorRekening.length > 0 && !nomorRekeningCocok);

	let formValid = $derived(
		namaBank.length > 0 && pemilikRekening.trim().length > 0 && nomorRekeningCocok
	);

	// ///////////////////////////////////////////////////////////////////////
	// Submit
	// ///////////////////////////////////////////////////////////////////////

	let statusSimpan = $state<'idle' | 'menyimpan' | 'tersimpan'>('idle');

	function simpanPerubahan() {
		if (!formValid) return;
		// TODO: panggil EditRekeningSeller dengan namaBank, nomorRekeningBaru,
		// pemilikRekening. Backend akan re-validasi namaBank terhadap whitelist
		// BankMap, jadi validasi di sini cuma buat UX, bukan satu-satunya lapis.
		statusSimpan = 'menyimpan';
		setTimeout(() => {
			statusSimpan = 'tersimpan';
			nomorRekeningBaru = '';
			konfirmasiNomorRekening = '';
			setTimeout(() => (statusSimpan = 'idle'), 2000);
		}, 700);
	}
</script>

<section id="edit-rekening" class="w-full bg-white p-4 sm:p-6 lg:p-8 text-slate-950">
	<!-- ///////////////////////////////////////////////////////////////// -->
	<!-- HEADER -->
	<!-- ///////////////////////////////////////////////////////////////// -->
	<div class="pb-5 mb-6 border-b border-zinc-800/10">
		<span class="text-[9px] sm:text-[10px] font-mono text-slate-400 uppercase tracking-wider">
			Rekening Toko / Edit
		</span>
		<h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
			Edit Rekening
		</h1>
	</div>

	<div class="max-w-xl">
		<!-- ///////////////////////////////////////////////////////////// -->
		<!-- BANNER PENJELASAN -->
		<!-- ///////////////////////////////////////////////////////////// -->
		<div class="bg-zinc-50 border border-zinc-200 rounded-md p-3 mb-5 text-[10px] text-zinc-500 leading-relaxed">
			Demi keamanan, nomor rekening yang sebelumnya terdaftar <strong class="text-zinc-700">tidak ditampilkan</strong>
			di halaman ini, sekalipun disamarkan. Kalau kamu tidak berniat mengganti nomor rekening, tetap masukkan
			nomor yang sama persis seperti sebelumnya. Kalau lupa nomor lamanya, cek dari buku tabungan atau aplikasi
			mobile banking kamu sebelum lanjut.
		</div>

		<!-- ///////////////////////////////////////////////////////////// -->
		<!-- FORM -->
		<!-- ///////////////////////////////////////////////////////////// -->
		<div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col gap-4">
			<label class="flex flex-col gap-1.5">
				<span class="text-[10px] font-medium text-slate-600">Nama Bank</span>
				<select
					bind:value={namaBank}
					class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 focus:outline-none focus:border-slate-950 transition-colors"
				>
					{#each DAFTAR_BANK as bank}
						<option value={bank}>{bank}</option>
					{/each}
				</select>
			</label>

			<label class="flex flex-col gap-1.5">
				<span class="text-[10px] font-medium text-slate-600">Nama Pemilik Rekening</span>
				<input
					type="text"
					bind:value={pemilikRekening}
					class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 focus:outline-none focus:border-slate-950 transition-colors"
				/>
			</label>

			<div class="pt-2 border-t border-dashed border-zinc-200">
				<label class="flex flex-col gap-1.5">
					<span class="text-[10px] font-medium text-slate-600">Nomor Rekening Baru</span>
					<input
						type="text"
						inputmode="numeric"
						bind:value={nomorRekeningBaru}
						placeholder="Masukkan nomor rekening baru"
						class="border border-zinc-300 rounded-md px-3 py-2 text-xs font-mono text-slate-900 placeholder-zinc-400 focus:outline-none focus:border-slate-950 transition-colors"
					/>
				</label>

				<label class="flex flex-col gap-1.5 mt-3">
					<span class="text-[10px] font-medium text-slate-600">Konfirmasi Nomor Rekening Baru</span>
					<input
						type="text"
						inputmode="numeric"
						bind:value={konfirmasiNomorRekening}
						placeholder="Ketik ulang nomor rekening di atas"
						class="border rounded-md px-3 py-2 text-xs font-mono text-slate-900 placeholder-zinc-400 focus:outline-none transition-colors {konfirmasiTidakKosongTapiBeda
							? 'border-rose-400 focus:border-rose-500'
							: 'border-zinc-300 focus:border-slate-950'}"
					/>
					{#if konfirmasiTidakKosongTapiBeda}
						<span class="text-[9px] text-rose-500">Belum sama dengan nomor rekening di atas.</span>
					{:else if nomorRekeningCocok}
						<span class="text-[9px] text-teal-600">Nomor rekening cocok.</span>
					{/if}
				</label>
			</div>

			<label class="flex items-center gap-2 text-[10px] text-zinc-600 pt-2 border-t border-dashed border-zinc-200">
				<input type="checkbox" bind:checked={isDefault} class="accent-slate-950" />
				Jadikan rekening default untuk pencairan dana
			</label>
		</div>

		<!-- ///////////////////////////////////////////////////////////// -->
		<!-- AKSI -->
		<!-- ///////////////////////////////////////////////////////////// -->
		<div class="flex items-center gap-3 mt-5">
			<button
				type="button"
				disabled={!formValid || statusSimpan === 'menyimpan'}
				onclick={simpanPerubahan}
				class="px-5 py-2.5 bg-slate-950 text-white text-[11px] font-bold uppercase tracking-wider rounded-sm hover:bg-slate-800 transition-colors disabled:opacity-30 disabled:cursor-not-allowed"
			>
				{statusSimpan === 'menyimpan' ? 'Menyimpan...' : 'Simpan Perubahan'}
			</button>
			<button type="button" class="text-[10px] font-medium text-slate-500 underline">
				Batal
			</button>
			{#if statusSimpan === 'tersimpan'}
				<span class="text-[10px] font-medium text-teal-700">Tersimpan ✓</span>
			{/if}
		</div>
	</div>
</section>