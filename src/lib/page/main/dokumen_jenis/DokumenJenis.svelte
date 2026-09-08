<script lang="ts">
	// State untuk menyimpan pilihan yang disorot oleh seller
	let selectedRole = $state<'distributor' | 'brand' | null>(null);

	const distributorRequirements = [
		'Surat Izin Usaha / Dokumen Izin Distributor yang sah',
		'Nomor Induk Berusaha (NIB) milik perusahaan',
		'NPWP Perusahaan yang aktif',
		'Surat Perjanjian Kerjasama Distribusi (jika ada)'
	];

	const brandRequirements = [
		'Sertifikat Pendaftaran Merek (DJKI / Lembaga Merek Resmi)',
		'Surat Kuasa / Dokumen Perwakilan (jika bukan pemilik langsung)',
		'Nomor Induk Berusaha (NIB) perusahaan',
		'NPWP Perusahaan yang aktif',
		'Asset Logo Brand resmi beresolusi tinggi'
	];
</script>

<section id="dokumen-jenis" class="min-h-screen  py-10 px-4 sm:px-6 lg:px-8 text-slate-900">
	<div class="space-y-5">
		
		<!-- 1. Header & Introduksi -->
		<div class="    space-y-4">
			<div class="inline-flex items-center gap-2 px-3 py-1 bg-slate-100 rounded-full text-[11px] font-mono font-semibold uppercase tracking-wider text-slate-700">
				<span class="w-2 h-2 rounded-full bg-slate-900"></span>
				Langkah Awal Penentuan Peran Seller
			</div>
			
			<h1 class="text-2xl sm:text-3xl font-extrabold uppercase tracking-tight text-slate-950">
				Verifikasi Legalitas & Kemitraan Toko
			</h1>
			
			<p class="text-sm leading-relaxed text-slate-600 max-w-3xl">
				Selamat datang di proses kemitraan resmi platform kami. Untuk menciptakan ekosistem bisnis yang aman, tepercaya, dan patuh terhadap regulasi perdagangan, kami memberlakukan proses standardisasi peran seller. 
				Sebelum melanjutkan, silakan pelajari ketentuan dan pilih kualifikasi usaha yang sesuai dengan operasional Anda saat ini.
			</p>

			<!-- Alert Informasi Penting -->
			<div class="border-l-4 border-slate-900 bg-slate-100/70 p-4 text-xs text-slate-700 rounded-r space-y-1">
				<p class="font-bold text-slate-950">Perhatian Sebelum Memilih:</p>
				<p class="leading-relaxed">
					Pilihan jenis usaha ini akan menentukan badge resmi, hak akses fitur grosir/katalog, serta dokumen legalitas yang wajib diunggah. Pastikan Anda memiliki wewenang hukum dan dokumen pendukung yang sah sebelum memulai.
				</p>
			</div>
		</div>

		<!-- 2. Perbandingan Peran & Ketetapan Bisnis -->
		<div class="grid grid-cols-1 md:grid-cols-2 gap-6">
			
			<!-- Kartu Option: Distributor -->
			<div 
				class="bg-white border-2 rounded-lg p-6 sm:p-7 flex flex-col justify-between transition-all duration-200 cursor-pointer relative shadow-sm hover:shadow-md {selectedRole === 'distributor' ? 'border-slate-950 bg-slate-50/50' : 'border-slate-200'}"
				onclick={() => (selectedRole = 'distributor')}
				onkeydown={(e) => e.key === 'Enter' && (selectedRole = 'distributor')}
				role="button"
				tabindex="0"
			>
				{#if selectedRole === 'distributor'}
					<div class="absolute top-4 right-4 bg-slate-950 text-white rounded-full p-1">
						<svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><polyline points="20 6 9 17 4 12"/></svg>
					</div>
				{/if}

				<div class="space-y-5">
					<div>
						<span class="text-[10px] font-mono font-bold tracking-widest text-slate-400 uppercase">Peran A</span>
						<h2 class="text-xl font-bold uppercase tracking-tight text-slate-950 mt-1">Distributor Resmi</h2>
						<p class="text-xs text-slate-600 mt-2 leading-relaxed">
							Diperuntukkan bagi badan usaha yang memegang hak distribusi produk dari berbagai merek, mengelola rantai pasok (*supply chain*), atau bertindak sebagai penyedia stok tangan pertama.
						</p>
					</div>

					<div class="border-t border-slate-100 pt-4 space-y-3">
						<p class="text-[11px] font-bold uppercase tracking-wider text-slate-900">Keuntungan Peran Distributor:</p>
						<ul class="text-xs text-slate-600 space-y-1.5 list-disc list-inside">
							<li>Akses ke fitur penetapan harga grosir (*tier pricing*)</li>
							<li>Badge verifikasi **Authorized Distributor** pada toko</li>
							<li>Kemudahan dalam distribusi pasokan skala besar ke *reseller*</li>
						</ul>
					</div>

					<div class="border-t border-slate-100 pt-4 space-y-3">
						<p class="text-[11px] font-bold uppercase tracking-wider text-slate-900">Dokumen Wajib Ditunjukkan:</p>
						<ul class="text-xs text-slate-600 space-y-1">
							{#each distributorRequirements as req}
								<li class="flex items-start gap-2">
									<span class="text-slate-400 font-bold">•</span>
									<span>{req}</span>
								</li>
							{/each}
						</ul>
					</div>
				</div>

				<div class="mt-8 pt-4 border-t border-slate-100">
					<button 
						type="button" 
						class="w-full py-2.5 text-xs font-bold uppercase tracking-wider transition-colors rounded-sm border {selectedRole === 'distributor' ? 'bg-slate-950 text-white border-slate-950' : 'bg-white text-slate-800 border-slate-300 hover:border-slate-950'}"
					>
						{selectedRole === 'distributor' ? 'Peran Distributor Dipilih' : 'Pilih Distributor'}
					</button>
				</div>
			</div>

			<!-- Kartu Option: Brand -->
			<div 
				class="bg-white border-2 rounded-lg p-6 sm:p-7 flex flex-col justify-between transition-all duration-200 cursor-pointer relative shadow-sm hover:shadow-md {selectedRole === 'brand' ? 'border-slate-950 bg-slate-50/50' : 'border-slate-200'}"
				onclick={() => (selectedRole = 'brand')}
				onkeydown={(e) => e.key === 'Enter' && (selectedRole = 'brand')}
				role="button"
				tabindex="0"
			>
				{#if selectedRole === 'brand'}
					<div class="absolute top-4 right-4 bg-slate-950 text-white rounded-full p-1">
						<svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><polyline points="20 6 9 17 4 12"/></svg>
					</div>
				{/if}

				<div class="space-y-5">
					<div>
						<span class="text-[10px] font-mono font-bold tracking-widest text-slate-400 uppercase">Peran B</span>
						<h2 class="text-xl font-bold uppercase tracking-tight text-slate-950 mt-1">Pemilik / Perwakilan Brand</h2>
						<p class="text-xs text-slate-600 mt-2 leading-relaxed">
							Diperuntukkan bagi entitas yang memiliki Sertifikat Merek (HKI) atau pemegang lisensi tunggal resmi yang memproduksi dan mengontrol hak cipta merek secara penuh.
						</p>
					</div>

					<div class="border-t border-slate-100 pt-4 space-y-3">
						<p class="text-[11px] font-bold uppercase tracking-wider text-slate-900">Keuntungan Peran Brand:</p>
						<ul class="text-xs text-slate-600 space-y-1.5 list-disc list-inside">
							<li>Akses status **Official Store / Official Brand**</li>
							<li>Hak untuk mendaftarkan nama katalog produk resmi baru</li>
							<li>Perlindungan Hak Cipta & Pelaporan Pelanggaran Merek</li>
						</ul>
					</div>

					<div class="border-t border-slate-100 pt-4 space-y-3">
						<p class="text-[11px] font-bold uppercase tracking-wider text-slate-900">Dokumen Wajib Ditunjukkan:</p>
						<ul class="text-xs text-slate-600 space-y-1">
							{#each brandRequirements as req}
								<li class="flex items-start gap-2">
									<span class="text-slate-400 font-bold">•</span>
									<span>{req}</span>
								</li>
							{/each}
						</ul>
					</div>
				</div>

				<div class="mt-8 pt-4 border-t border-slate-100">
					<button 
						type="button" 
						class="w-full py-2.5 text-xs font-bold uppercase tracking-wider transition-colors rounded-sm border {selectedRole === 'brand' ? 'bg-slate-950 text-white border-slate-950' : 'bg-white text-slate-800 border-slate-300 hover:border-slate-950'}"
					>
						{selectedRole === 'brand' ? 'Peran Brand Dipilih' : 'Pilih Brand'}
					</button>
				</div>
			</div>

		</div>

		<!-- 3. Jembatan Navigasi Aksi -->
		<div class="bg-white border border-slate-200 rounded-lg p-6 sm:p-8 flex flex-col sm:flex-row items-center justify-between gap-4 shadow-sm">
			<div class="text-center sm:text-left space-y-1">
				<p class="text-xs font-bold text-slate-900 uppercase tracking-wider">
					{#if selectedRole === 'distributor'}
						Anda Memilih: Pendaftaran Distributor
					{:else if selectedRole === 'brand'}
						Anda Memilih: Pendaftaran Brand / Merek
					{:else}
						Pilih Salah Satu Peran Di Atas
					{/if}
				</p>
				<p class="text-xs text-slate-500">
					{#if selectedRole}
						Klik tombol di samping untuk melanjutkan ke pengisian formulir & unggah dokumen legalitas.
					{:else}
						Silakan klik salah satu opsi peran di atas untuk mengaktifkan tombol pendaftaran.
					{/if}
				</p>
			</div>

			<div class="w-full sm:w-auto">
				{#if selectedRole === 'distributor'}
					<a
						href="/dokumen-jenis/distributor"
						class="inline-flex justify-center items-center gap-2 w-full sm:w-auto px-6 py-3 bg-slate-950 text-white text-xs font-bold uppercase tracking-wider rounded-sm hover:bg-slate-800 transition-colors shadow-sm"
					>
						<span>Lanjut ke Form Distributor</span>
						<svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="5" y1="12" x2="19" y2="12"/><polyline points="12 5 19 12 12 19"/></svg>
					</a>
				{:else if selectedRole === 'brand'}
					<a
						href="/dokumen-jenis/brand"
						class="inline-flex justify-center items-center gap-2 w-full sm:w-auto px-6 py-3 bg-slate-950 text-white text-xs font-bold uppercase tracking-wider rounded-sm hover:bg-slate-800 transition-colors shadow-sm"
					>
						<span>Lanjut ke Form Brand</span>
						<svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="5" y1="12" x2="19" y2="12"/><polyline points="12 5 19 12 12 19"/></svg>
					</a>
				{:else}
					<button
						disabled
						class="w-full sm:w-auto px-6 py-3 bg-slate-200 text-slate-400 text-xs font-bold uppercase tracking-wider rounded-sm cursor-not-allowed"
					>
						Pilih Peran Dahulu
					</button>
				{/if}
			</div>
		</div>

	</div>
</section>