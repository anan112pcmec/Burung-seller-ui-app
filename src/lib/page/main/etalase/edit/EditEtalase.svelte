<script lang="ts">
	import { goto } from '$app/navigation';

	// Tipe data sesuai struct Go
	type Etalase = {
		id_etalase: number;
		id_seller_etalase: number;
		nama_etalase: string;
		deskripsi_etalase: string;
		jumlah_barang: number;
		foto_url?: string;
		created_at: string;
		updated_at: string;
	};

	// Props untuk menerima data etalase yang sedang diedit
	let { etalaseData }: { etalaseData?: Etalase } = $props();

	// State Form (Default value dari props jika ada)
	let idEtalase = $state(etalaseData?.id_etalase ?? 0);
	let namaEtalase = $state(etalaseData?.nama_etalase ?? '');
	let deskripsiEtalase = $state(etalaseData?.deskripsi_etalase ?? '');
	let fotoUrl = $state(etalaseData?.foto_url ?? '');
	let fotoFile = $state<File | null>(null);
	let previewImage = $state<string | null>(etalaseData?.foto_url || null);

	// Limit simulasi kapasitas etalase
	const maxJumlahBarang = 100;
	let jumlahBarang = $state(etalaseData?.jumlah_barang ?? 0);

	// Handle upload foto dari microservice gambar
	function handleFileChange(event: Event) {
		const target = event.target as HTMLInputElement;
		if (target.files && target.files[0]) {
			const file = target.files[0];
			fotoFile = file;
			previewImage = URL.createObjectURL(file);
		}
	}

	// Kalkulasi Skor Kelengkapan Data
	let kelengkapanScore = $derived.by(() => {
		let score = 0;
		if (namaEtalase.trim().length >= 3) score += 40;
		if (deskripsiEtalase.trim().length >= 10) score += 40;
		if (previewImage) score += 20;
		return score;
	});

	let isFormValid = $derived(namaEtalase.trim().length >= 3 && deskripsiEtalase.trim().length >= 10);

	// Handler Submit Form
	async function handleSubmit(e: SubmitEvent) {
		e.preventDefault();
		if (!isFormValid) return;

		// Payload JSON untuk Go microservice utama
		const payload = {
			id_etalase: idEtalase,
			nama_etalase: namaEtalase,
			deskripsi_etalase: deskripsiEtalase
		};

		// Jika fotoFile ada, unggah ke microservice media/foto terlebih dahulu
		console.log('Submit payload:', payload, 'Foto File:', fotoFile);
	}

    const persenIsi:number = Math.round((jumlahBarang / maxJumlahBarang) * 100)
    const aktif:boolean = jumlahBarang > 0
</script>

<div class="max-w-6xl mx-auto p-6">
	<!-- Header Page -->
	<div class="mb-6 flex items-center justify-between">
		<div>
			<h1 class="text-xl font-bold text-zinc-800">Edit Etalase</h1>
			<p class="text-xs text-zinc-500 mt-0.5">Perbarui informasi dan tampilan etalase toko Anda.</p>
		</div>
		<button
			type="button"
			onclick={() => history.back()}
			class="px-3 py-1.5 text-xs font-medium text-zinc-600 bg-zinc-100 hover:bg-zinc-200 rounded-md transition-colors"
		>
			Batal
		</button>
	</div>

	<div class="grid grid-cols-1 lg:grid-cols-12 gap-8 items-start">
		<!-- KIRI: FORM EDITING -->
		<div class="lg:col-span-7 bg-white border border-zinc-200 rounded-xl p-6 shadow-sm">
			<form onsubmit={handleSubmit} class="space-y-5">
				<!-- Upload Foto Etalase (Microservice terpisah) -->
				<div>
					<label for="foto-etalase" class="block text-xs font-semibold text-zinc-700 mb-2">
						Foto / Icon Etalase
					</label>
					<div class="flex items-center gap-4">
						<div class="w-16 h-16 rounded-full bg-zinc-100 border border-zinc-200 flex items-center justify-center overflow-hidden shrink-0">
							{#if previewImage}
								<img src={previewImage} alt="Preview Foto" class="w-full h-full object-cover" />
							{:else}
								<svg xmlns="http://www.w3.org/2000/svg" class="w-7 h-7 text-zinc-400 stroke-1" viewBox="0 0 24 24" fill="none" stroke="currentColor">
									<path d="M2 7h20v3a2 2 0 0 1-2 2h-1a2 2 0 0 1-2-2 2 2 0 0 1-2 2h-1a2 2 0 0 1-2-2 2 2 0 0 1-2 2H9a2 2 0 0 1-2-2 2 2 0 0 1-2 2H4a2 2 0 0 1-2-2Z"/>
									<path d="M4 12v6a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-6"/>
									<path d="M9 21v-5a1 1 0 0 1 1-1h4a1 1 0 0 1 1 1v5"/>
								</svg>
							{/if}
						</div>

						<div class="flex-1">
							<input
								type="file"
								id="foto-etalase"
								accept="image/*"
								onchange={handleFileChange}
								class="hidden"
							/>
							<label
								for="foto-etalase"
								class="inline-block px-3 py-1.5 bg-white border border-zinc-300 hover:border-zinc-400 text-zinc-700 text-xs font-medium rounded-lg cursor-pointer transition-colors"
							>
								{previewImage ? 'Ganti Foto' : 'Unggah Foto'}
							</label>
							<p class="text-[10px] text-zinc-400 mt-1">Format JPG, PNG. Maksimal 2MB.</p>
						</div>
					</div>
				</div>

				<hr class="border-zinc-100" />

				<!-- Nama Etalase -->
				<div>
					<label for="nama_etalase" class="block text-xs font-semibold text-zinc-700 mb-1">
						Nama Etalase <span class="text-rose-500">*</span>
					</label>
					<input
						type="text"
						id="nama_etalase"
						bind:value={namaEtalase}
						maxlength="100"
						placeholder="Contoh: Pakaian Pria, Promo Spesial"
						class="w-full px-3 py-2 text-xs border border-zinc-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-teal-500/20 focus:border-teal-600 transition-all"
					/>
					<div class="flex justify-between items-center mt-1">
						<span class="text-[10px] text-zinc-400">Minimal 3 karakter.</span>
						<span class="text-[10px] text-zinc-400 font-mono">{namaEtalase.length}/100</span>
					</div>
				</div>

				<!-- Deskripsi Etalase -->
				<div>
					<label for="deskripsi_etalase" class="block text-xs font-semibold text-zinc-700 mb-1">
						Deskripsi Etalase <span class="text-rose-500">*</span>
					</label>
					<textarea
						id="deskripsi_etalase"
						bind:value={deskripsiEtalase}
						rows="4"
						placeholder="Jelaskan jenis barang atau penawaran dalam etalase ini..."
						class="w-full px-3 py-2 text-xs border border-zinc-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-teal-500/20 focus:border-teal-600 transition-all resize-none"
					></textarea>
					<span class="text-[10px] text-zinc-400">Minimal 10 karakter agar informasi jelas bagi pembeli.</span>
				</div>

				<!-- Submit Button -->
				<div class="pt-2 flex justify-end">
					<button
						type="submit"
						disabled={!isFormValid}
						class="px-5 py-2 text-xs font-semibold text-white bg-teal-700 hover:bg-teal-800 disabled:bg-zinc-300 disabled:cursor-not-allowed rounded-lg shadow-sm transition-colors"
					>
						Simpan Perubahan
					</button>
				</div>
			</form>
		</div>

		<!-- KANAN: PREVIEW & INDICATOR KELENGKAPAN DATA -->
		<div class="lg:col-span-5 space-y-6">
			<!-- Tracker Kelengkapan -->
			<div class="bg-white border border-zinc-200 rounded-xl p-5 shadow-sm">
				<div class="flex items-center justify-between mb-2">
					<span class="text-xs font-semibold text-zinc-700">Kelengkapan Data Etalase</span>
					<span class="text-xs font-mono font-bold text-teal-700">{kelengkapanScore}%</span>
				</div>
				<div class="w-full h-2 bg-zinc-100 rounded-full overflow-hidden mb-4">
					<div
						class="h-full bg-teal-600 transition-all duration-300 rounded-full"
						style:width="{kelengkapanScore}%"
					></div>
				</div>

				<!-- Checklists -->
				<div class="space-y-2 text-xs">
					<div class="flex items-center gap-2">
						{#if namaEtalase.trim().length >= 3}
							<span class="w-4 h-4 rounded-full bg-teal-100 text-teal-700 flex items-center justify-center text-[10px] font-bold">✓</span>
							<span class="text-zinc-700">Nama etalase valid</span>
						{:else}
							<span class="w-4 h-4 rounded-full bg-zinc-100 text-zinc-400 flex items-center justify-center text-[10px]">○</span>
							<span class="text-zinc-400">Nama etalase min. 3 karakter</span>
						{/if}
					</div>

					<div class="flex items-center gap-2">
						{#if deskripsiEtalase.trim().length >= 10}
							<span class="w-4 h-4 rounded-full bg-teal-100 text-teal-700 flex items-center justify-center text-[10px] font-bold">✓</span>
							<span class="text-zinc-700">Deskripsi etalase terisi jelas</span>
						{:else}
							<span class="w-4 h-4 rounded-full bg-zinc-100 text-zinc-400 flex items-center justify-center text-[10px]">○</span>
							<span class="text-zinc-400">Deskripsi etalase min. 10 karakter</span>
						{/if}
					</div>

					<div class="flex items-center gap-2">
						{#if previewImage}
							<span class="w-4 h-4 rounded-full bg-teal-100 text-teal-700 flex items-center justify-center text-[10px] font-bold">✓</span>
							<span class="text-zinc-700">Foto etalase terunggah</span>
						{:else}
							<span class="w-4 h-4 rounded-full bg-zinc-100 text-zinc-400 flex items-center justify-center text-[10px]">○</span>
							<span class="text-zinc-400">Foto etalase (Opsional)</span>
						{/if}
					</div>
				</div>
			</div>

			<!-- Live Preview Kartu Etalase -->
			<div>
				<span class="block text-xs font-semibold text-zinc-500 uppercase tracking-wider mb-2">Live Preview Kartu</span>
				
				

				<div class="h-[15.5rem] w-full border border-zinc-200 rounded-lg bg-white overflow-hidden shadow-sm flex flex-col pointer-events-none">
					<!-- BANNER -->
					<div class="relative h-[4.75rem] w-full shrink-0 overflow-hidden bg-gradient-to-br from-teal-700 via-teal-800 to-zinc-900">
						<span class="absolute top-2 left-2 px-1.5 py-0.5 bg-black/30 backdrop-blur-xs text-white text-[9px] font-mono rounded">
							#Preview
						</span>

						<span class="absolute top-2 right-2 px-1.5 py-0.5 {aktif ? 'bg-teal-500/20 text-teal-50' : 'bg-white/15 text-white/70'} backdrop-blur-xs text-[9px] font-medium rounded uppercase tracking-wider">
							{aktif ? 'Aktif' : 'Kosong'}
						</span>
					</div>

					<!-- KONTEN -->
					<div class="relative px-4 pb-3.5 flex-1 flex flex-col min-h-0 text-left">
						<div class="-mt-6 mb-2 flex items-end justify-between">
							<div class="w-12 h-12 rounded-full bg-white border-[3px] border-white ring-1 ring-zinc-200 shadow-sm flex items-center justify-center shrink-0 overflow-hidden">
								{#if previewImage}
									<img src={previewImage} alt="Avatar Etalase" class="w-full h-full object-cover" />
								{:else}
									<svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5 text-zinc-400 stroke-1" viewBox="0 0 24 24" fill="none" stroke="currentColor">
										<path d="M2 7h20v3a2 2 0 0 1-2 2h-1a2 2 0 0 1-2-2 2 2 0 0 1-2 2h-1a2 2 0 0 1-2-2 2 2 0 0 1-2 2H9a2 2 0 0 1-2-2 2 2 0 0 1-2 2H4a2 2 0 0 1-2-2Z"/>
										<path d="M4 12v6a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-6"/>
										<path d="M9 21v-5a1 1 0 0 1 1-1h4a1 1 0 0 1 1 1v5"/>
									</svg>
								{/if}
							</div>
							<span class="text-[9px] text-zinc-400 font-mono mb-0.5">dibuat Hari ini</span>
						</div>

						<h3 class="text-xs font-semibold text-zinc-800 leading-tight truncate">
							{namaEtalase.trim() || 'Nama Etalase'}
						</h3>
						<p class="text-[10px] text-zinc-400 line-clamp-2 mt-0.5 leading-tight">
							{deskripsiEtalase.trim() || 'Deskripsi etalase akan tampil di bagian ini secara otomatis...'}
						</p>

						<!-- Progress Bar -->
						<div class="mt-auto pt-2.5 border-t border-zinc-100">
							<div class="flex items-center justify-between mb-1">
								<span class="text-[9px] font-semibold uppercase tracking-wider text-zinc-400">Isi Etalase</span>
								<span class="text-[10px] font-mono font-bold text-zinc-700">{persenIsi}%</span>
							</div>
							<div class="w-full h-1.5 bg-zinc-100 rounded-full overflow-hidden">
								<div
									class="h-full rounded-full {aktif ? 'bg-teal-600' : 'bg-zinc-300'}"
									style:width="{persenIsi}%"
								></div>
							</div>

							<div class="mt-2 flex items-center justify-between text-[10px] text-zinc-500 font-mono">
								<span class="flex items-center gap-1">
									<svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-zinc-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
										<path d="M21 8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16Z"/>
										<path d="m3.3 7 8.7 5 8.7-5"/><path d="M12 22V12"/>
									</svg>
									{jumlahBarang} barang
								</span>
								<span class="flex items-center gap-1">
									<svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-zinc-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
										<path d="M12 8v4l3 3"/><circle cx="12" cy="12" r="9"/>
									</svg>
									baru saja
								</span>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</div>