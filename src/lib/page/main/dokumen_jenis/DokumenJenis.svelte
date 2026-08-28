<script lang="ts">
	// ///////////////////////////////////////////////////////////////////////
	// Types
	// ///////////////////////////////////////////////////////////////////////

	type JenisSeller = 'personal' | 'distributor' | 'brand';
	type StatusPengajuan = 'Pending' | 'Disetujui' | 'Ditolak' | 'Belum Diajukan';
	type StatusDokumen = 'kosong' | 'mengunggah' | 'terunggah';

	interface FieldConfig {
		key: string;
		label: string;
		placeholder: string;
	}

	interface DocConfig {
		key: string;
		label: string;
		hint: string;
		accept: string;
	}

	interface DokumenState {
		status: StatusDokumen;
		namaFile: string | null;
	}

	// ///////////////////////////////////////////////////////////////////////
	// State
	// ///////////////////////////////////////////////////////////////////////

	let jenisSeller: JenisSeller = $state('distributor');

	let formDistributor = $state<Record<string, string>>({
		namaPerusahaan: 'PT Sinergi Distribusi Nusantara',
		nib: '1234567890123',
		npwp: '01.234.567.8-901.000'
	});

	let formBrand = $state<Record<string, string>>({
		namaPerusahaan: 'PT Kreasi Merek Indonesia',
		negaraAsal: 'Indonesia',
		lembagaPendaftaran: 'DJKI Kementerian Hukum & HAM',
		nomorPendaftaranMerek: 'IDM000123456',
		nib: '9876543210987',
		npwp: '02.345.678.9-012.000'
	});

	let statusDistributor = $state<StatusPengajuan>('Pending');
	let alasanDistributor = $state('');

	let statusBrand = $state<StatusPengajuan>('Ditolak');
	let alasanBrand = $state(
		'Sertifikat merek belum bisa diverifikasi keasliannya. Mohon unggah ulang dengan scan dokumen asli, bukan fotokopi.'
	);

	let dokumenDistributor: Record<string, DokumenState> = $state({
		dokumenIzin: { status: 'terunggah', namaFile: 'izin-distributor-2026.pdf' },
		npwpFoto: { status: 'terunggah', namaFile: 'npwp-scan.jpg' },
		nibFoto: { status: 'kosong', namaFile: null },
		suratKerjasama: { status: 'kosong', namaFile: null }
	});

	let dokumenBrand: Record<string, DokumenState> = $state({
		sertifikatMerek: { status: 'terunggah', namaFile: 'sertifikat-merek-lama.pdf' },
		dokumenPerwakilan: { status: 'terunggah', namaFile: 'surat-kuasa.pdf' },
		logoFoto: { status: 'terunggah', namaFile: 'logo-brand.png' },
		nibFoto: { status: 'terunggah', namaFile: 'nib-brand.jpg' },
		npwpFoto: { status: 'terunggah', namaFile: 'npwp-brand.jpg' },
		suratKerjasama: { status: 'kosong', namaFile: null }
	});

	// ///////////////////////////////////////////////////////////////////////
	// Config — field & dokumen per jenis seller
	// ///////////////////////////////////////////////////////////////////////

	const fieldsDistributor: FieldConfig[] = [
		{ key: 'namaPerusahaan', label: 'Nama Perusahaan', placeholder: 'PT Sinergi Distribusi Nusantara' },
		{ key: 'nib', label: 'Nomor Induk Berusaha (NIB)', placeholder: '1234567890123' },
		{ key: 'npwp', label: 'NPWP Perusahaan', placeholder: '01.234.567.8-901.000' }
	];

	const fieldsBrand: FieldConfig[] = [
		{ key: 'namaPerusahaan', label: 'Nama Perusahaan', placeholder: 'PT Kreasi Merek Indonesia' },
		{ key: 'negaraAsal', label: 'Negara Asal Brand', placeholder: 'Indonesia' },
		{ key: 'lembagaPendaftaran', label: 'Lembaga Pendaftaran Merek', placeholder: 'DJKI Kemenkumham' },
		{ key: 'nomorPendaftaranMerek', label: 'Nomor Pendaftaran Merek', placeholder: 'IDM000123456' },
		{ key: 'nib', label: 'Nomor Induk Berusaha (NIB)', placeholder: '9876543210987' },
		{ key: 'npwp', label: 'NPWP Perusahaan', placeholder: '02.345.678.9-012.000' }
	];

	const docsDistributor: DocConfig[] = [
		{ key: 'dokumenIzin', label: 'Dokumen Izin Distributor', hint: 'PDF, scan asli, maks 5MB', accept: '.pdf' },
		{ key: 'npwpFoto', label: 'Foto NPWP', hint: 'JPG/PNG, jelas & tidak buram', accept: '.jpg,.jpeg,.png' },
		{ key: 'nibFoto', label: 'Foto NIB', hint: 'JPG/PNG, jelas & tidak buram', accept: '.jpg,.jpeg,.png' },
		{ key: 'suratKerjasama', label: 'Surat Kerjasama Distribusi', hint: 'PDF, ditandatangani kedua pihak', accept: '.pdf' }
	];

	const docsBrand: DocConfig[] = [
		{ key: 'sertifikatMerek', label: 'Sertifikat Merek', hint: 'PDF, dari lembaga pendaftaran', accept: '.pdf' },
		{ key: 'dokumenPerwakilan', label: 'Dokumen Surat Kuasa / Perwakilan', hint: 'PDF, wajib jika bukan pemilik langsung', accept: '.pdf' },
		{ key: 'logoFoto', label: 'Logo Brand', hint: 'PNG transparan, resolusi tinggi', accept: '.png' },
		{ key: 'nibFoto', label: 'Foto NIB', hint: 'JPG/PNG, jelas & tidak buram', accept: '.jpg,.jpeg,.png' },
		{ key: 'npwpFoto', label: 'Foto NPWP', hint: 'JPG/PNG, jelas & tidak buram', accept: '.jpg,.jpeg,.png' },
		{ key: 'suratKerjasama', label: 'Surat Kerjasama (opsional)', hint: 'PDF, jika ada perjanjian tertulis', accept: '.pdf' }
	];

	// ///////////////////////////////////////////////////////////////////////
	// Derived — bundel config aktif berdasarkan tab terpilih
	// ///////////////////////////////////////////////////////////////////////

	let formAktif = $derived(jenisSeller === 'distributor' ? formDistributor : formBrand);
	let fieldsAktif = $derived(jenisSeller === 'distributor' ? fieldsDistributor : fieldsBrand);
	let docsAktif = $derived(jenisSeller === 'distributor' ? docsDistributor : docsBrand);
	let dokumenAktif = $derived(jenisSeller === 'distributor' ? dokumenDistributor : dokumenBrand);
	let statusAktif = $derived(jenisSeller === 'distributor' ? statusDistributor : statusBrand);
	let alasanAktif = $derived(jenisSeller === 'distributor' ? alasanDistributor : alasanBrand);

	let dokumenTerunggah = $derived(Object.values(dokumenAktif).filter((d) => d.status === 'terunggah').length);
	let totalDokumen = $derived(docsAktif.length);

	// ///////////////////////////////////////////////////////////////////////
	// Handlers
	// ///////////////////////////////////////////////////////////////////////

	function handleFileChange(e: Event, docKey: string) {
		const target = e.target as HTMLInputElement;
		const file = target.files?.[0];
		if (!file) return;

		const state = dokumenAktif[docKey];
		state.status = 'mengunggah';
		state.namaFile = file.name;

		setTimeout(() => {
			state.status = 'terunggah';
		}, 900);
	}

	function hapusDokumen(docKey: string) {
		const state = dokumenAktif[docKey];
		state.status = 'kosong';
		state.namaFile = null;
	}

	function statusPengajuanClass(status: StatusPengajuan): string {
		if (status === 'Disetujui') return 'bg-teal-50 text-teal-700 border-teal-200';
		if (status === 'Ditolak') return 'bg-rose-50 text-rose-600 border-rose-200';
		if (status === 'Pending') return 'bg-zinc-100 text-zinc-600 border-zinc-200';
		return 'bg-zinc-50 text-zinc-400 border-zinc-200';
	}
</script>

<section id="legalitas-seller" class="w-full bg-white p-4 sm:p-6 lg:p-8 text-slate-950">
	<div class="pb-6 mb-6 border-b border-zinc-800/10">
		<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
			LEGALITAS TOKO
		</span>
		<h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
			Jenis Seller &amp; Verifikasi Dokumen
		</h1>
	</div>

	<div class="inline-flex border border-zinc-800/20 rounded-xs overflow-hidden mb-6">
		{#each [
			{ value: 'personal', label: 'Personal' },
			{ value: 'distributor', label: 'Distributor' },
			{ value: 'brand', label: 'Brand' }
		] as opt}
			<button
				type="button"
				onclick={() => (jenisSeller = opt.value as JenisSeller)}
				class="px-4 py-2 text-[10px] sm:text-[11px] font-medium uppercase tracking-wider transition duration-200 {jenisSeller === opt.value
					? 'bg-slate-950 text-white'
					: 'text-slate-600 hover:bg-slate-50'}"
			>
				{opt.label}
			</button>
		{/each}
	</div>

	{#if jenisSeller === 'personal'}
		<div class="border border-zinc-800/20 rounded-sm p-6 sm:p-8 flex flex-col items-start gap-3">
			<span class="px-2 py-1 rounded text-[9px] font-medium uppercase tracking-wider bg-zinc-100 text-zinc-600">
				Tanpa Dokumen Tambahan
			</span>
			<p class="text-sm text-slate-700 max-w-md leading-relaxed">
				Sebagai <strong>Personal Seller</strong>, kamu tidak perlu mengisi data perusahaan atau mengunggah dokumen legalitas apa pun. Kamu bisa langsung mulai jualan.
			</p>
			<p class="text-[11px] text-slate-400 leading-relaxed">
				Kalau toko kamu adalah distributor resmi atau pemilik/pemegang lisensi sebuah brand, kamu bisa ajukan upgrade jenis seller lewat tab di atas untuk membuka fitur & badge tambahan.
			</p>
		</div>
	{:else}
		<div class="border rounded-sm p-4 sm:p-5 mb-6 flex flex-col sm:flex-row sm:items-center sm:justify-between gap-3 {statusPengajuanClass(statusAktif)}">
			<div class="flex items-center gap-3">
				<svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4 flex-shrink-0" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
					{#if statusAktif === 'Disetujui'}
						<path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><path d="m9 11 3 3L22 4"/>
					{:else if statusAktif === 'Ditolak'}
						<circle cx="12" cy="12" r="10"/><path d="m15 9-6 6M9 9l6 6"/>
					{:else}
						<circle cx="12" cy="12" r="10"/><path d="M12 6v6l4 2"/>
					{/if}
				</svg>
				<div>
					<p class="text-xs font-bold uppercase tracking-wider">Status Pengajuan: {statusAktif}</p>
					{#if statusAktif === 'Ditolak' && alasanAktif}
						<p class="text-[11px] mt-0.5 leading-relaxed opacity-90">{alasanAktif}</p>
					{:else if statusAktif === 'Pending'}
						<p class="text-[11px] mt-0.5 leading-relaxed opacity-80">Sedang ditinjau oleh tim kami, biasanya 1-3 hari kerja.</p>
					{:else if statusAktif === 'Disetujui'}
						<p class="text-[11px] mt-0.5 leading-relaxed opacity-80">Badge {jenisSeller === 'distributor' ? 'Distributor' : 'Brand'} resmi sudah aktif di toko kamu.</p>
					{/if}
				</div>
			</div>
			<span class="text-[10px] font-mono flex-shrink-0">{dokumenTerunggah}/{totalDokumen} dokumen terunggah</span>
		</div>

		<div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5 mb-4">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				Data Perusahaan
			</span>

			<div class="mt-4 grid grid-cols-1 sm:grid-cols-2 gap-4">
				{#each fieldsAktif as field}
					<label class="flex flex-col gap-1.5">
						<span class="text-[10px] font-medium text-slate-600">{field.label}</span>
						<input
							type="text"
							bind:value={formAktif[field.key]}
							placeholder={field.placeholder}
							class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 placeholder-zinc-400 focus:outline-none focus:border-slate-950 transition-colors"
						/>
					</label>
				{/each}
			</div>
		</div>

		<div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5">
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
				Dokumen Legalitas
			</span>

			<div class="mt-4 grid grid-cols-1 sm:grid-cols-2 gap-3">
				{#each docsAktif as doc}
					{@const state = dokumenAktif[doc.key]}
					<div class="border border-dashed border-zinc-300 rounded-md p-3 flex items-start gap-3">
						<div class="w-9 h-9 rounded-md flex items-center justify-center flex-shrink-0 {state.status === 'terunggah' ? 'bg-teal-50' : 'bg-zinc-100'}">
							{#if state.status === 'terunggah'}
								<svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4 text-teal-600" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
									<path d="M14.5 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V7.5L14.5 2Z"/><path d="M14 2v6h6"/><path d="m9 15 2 2 4-4"/>
								</svg>
							{:else if state.status === 'mengunggah'}
								<svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4 text-zinc-400 animate-spin" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
									<path d="M21 12a9 9 0 1 1-6.219-8.56"/>
								</svg>
							{:else}
								<svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4 text-zinc-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
									<path d="M12 13v8"/><path d="m8 17 4-4 4 4"/><path d="M20.39 18.39A5 5 0 0 0 18 9h-1.26A8 8 0 1 0 3 16.3"/>
								</svg>
							{/if}
						</div>

						<div class="flex-1 min-w-0">
							<p class="text-[11px] font-semibold text-zinc-800 truncate">{doc.label}</p>

							{#if state.status === 'kosong'}
								<p class="text-[9px] text-zinc-400 mt-0.5">{doc.hint}</p>
								<label class="inline-flex items-center gap-1 mt-2 text-[10px] font-medium text-slate-950 border border-zinc-300 rounded px-2 py-1 cursor-pointer hover:bg-zinc-50 transition-colors">
									Pilih File
									<input type="file" accept={doc.accept} class="hidden" onchange={(e) => handleFileChange(e, doc.key)} />
								</label>
							{:else if state.status === 'mengunggah'}
								<p class="text-[9px] text-zinc-400 mt-0.5 truncate">Mengunggah {state.namaFile}...</p>
							{:else}
								<p class="text-[9px] text-teal-700 mt-0.5 truncate font-medium">{state.namaFile}</p>
								<div class="flex items-center gap-2 mt-1.5">
									<label class="text-[9px] font-medium text-slate-600 underline cursor-pointer">
										Ganti
										<input type="file" accept={doc.accept} class="hidden" onchange={(e) => handleFileChange(e, doc.key)} />
									</label>
									<button type="button" onclick={() => hapusDokumen(doc.key)} class="text-[9px] font-medium text-rose-500 underline">
										Hapus
									</button>
								</div>
							{/if}
						</div>
					</div>
				{/each}
			</div>
		</div>

		<div class="flex justify-end mt-5">
			<button
				type="button"
				class="px-5 py-2.5 bg-slate-950 text-white text-[11px] font-bold uppercase tracking-wider rounded-sm hover:bg-slate-800 transition-colors"
			>
				{statusAktif === 'Belum Diajukan' ? 'Ajukan Verifikasi' : 'Simpan & Ajukan Ulang'}
			</button>
		</div>
	{/if}
</section>