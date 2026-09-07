<script lang="ts">
    // Simulasi Data Barang Induk di State (Svelte 5 Runes)
    let idBarangInduk = $state(101);
    let sellerID = $state(1);
    let namaBarang = $state('Kabel Data Type-C Fast Charge');
    let jenisBarang = $state('Semua Barang');
    let deskripsi = $state(
        'Kabel premium braided nylon panjang 1 meter output 65W dengan proteksi arus pendek.'
    );
    let createdAt = '2026-01-15T08:30:00Z';
    let updatedAt = $state('2026-02-01T10:00:00Z');

    // State Media
    let fotoUrl = $state<string | null>(null);
    let videoUrl = $state<string | null>(null);

    // State Form UI
    let isSaving = $state(false);
    let showSuccessToast = $state(false);

    const opsiJenisBarang = [
        'Semua Barang',
        'Khusus Offline',
        'Khusus Online',
        'Promosi Event'
    ];

    // Hitung Skor Kelengkapan
    let kelengkapanScore = $derived(
        (namaBarang.trim().length >= 3 ? 30 : 0) +
        (jenisBarang.trim().length > 0 ? 10 : 0) +
        (deskripsi.trim().length >= 10 ? 30 : 0) +
        (fotoUrl ? 20 : 0) +
        (videoUrl ? 10 : 0)
    );

    function handleFotoUpload(e: Event) {
        const input = e.target as HTMLInputElement;
        if (input.files?.[0]) {
            fotoUrl = URL.createObjectURL(input.files[0]);
        }
    }

    function handleVideoUpload(e: Event) {
        const input = e.target as HTMLInputElement;
        if (input.files?.[0]) {
            videoUrl = URL.createObjectURL(input.files[0]);
        }
    }

    function removeFoto() { fotoUrl = null; }
    function removeVideo() { videoUrl = null; }

    function handleSimpan(e: SubmitEvent) {
        e.preventDefault();
        isSaving = true;

        setTimeout(() => {
            isSaving = false;
            updatedAt = new Date().toISOString();
            showSuccessToast = true;

            setTimeout(() => {
                showSuccessToast = false;
            }, 3000);
        }, 500);
    }

    function formatTanggal(tglStr: string) {
        if (!tglStr) return '—';
        const d = new Date(tglStr);
        if (isNaN(d.getTime())) return tglStr;
        return d.toLocaleDateString('id-ID', {
            day: '2-digit',
            month: 'short',
            year: 'numeric'
        });
    }
</script>

<section id="edit-barang-induk" class=" p-6 font-sans text-slate-800 antialiased">
    <!-- Header Page -->
    <header class="mb-8 flex items-center justify-between pb-4 border-b border-zinc-200">
        <div>
            <div class="flex items-center gap-2 mb-1">
                <span class="text-xs font-mono text-zinc-400">ID #{idBarangInduk}</span>
                <span class="text-zinc-300">•</span>
                <span class="text-xs font-mono text-zinc-400">Seller #{sellerID}</span>
            </div>
            <h1 class="text-lg font-semibold tracking-tight text-zinc-900">Edit Barang Induk</h1>
        </div>
        <button
            type="button"
            onclick={() => history.back()}
            class="px-3 py-1.5 text-xs font-medium text-zinc-600 hover:text-zinc-900 bg-white border border-zinc-200 hover:border-zinc-300 rounded transition-colors"
        >
            Batal
        </button>
    </header>

    <!-- Toast Notification (Minimalist Teal Accent) -->
    {#if showSuccessToast}
        <div class="mb-6 p-3 bg-teal-50/50 border border-teal-200 rounded flex items-center justify-between text-xs text-teal-900">
            <div class="flex items-center gap-2">
                <span class="w-1.5 h-1.5 rounded-full bg-teal-600"></span>
                <span>Perubahan berhasil disimpan.</span>
            </div>
            <span class="font-mono text-[11px] text-teal-700">{formatTanggal(updatedAt)}</span>
        </div>
    {/if}

    <div class="grid grid-cols-1 lg:grid-cols-12 gap-10 items-start">
        <!-- FORM UTAMA -->
        <form onsubmit={handleSimpan} class="lg:col-span-7 space-y-6">
            <!-- Basic Fields -->
            <div class="space-y-4">
                <div>
                    <label for="nama-barang" class="block text-xs font-medium text-zinc-700 mb-1.5">
                        Nama Barang <span class="text-teal-600">*</span>
                    </label>
                    <input
                        id="nama-barang"
                        type="text"
                        bind:value={namaBarang}
                        required
                        placeholder="Nama barang..."
                        class="w-full px-3 py-2 text-xs bg-white border border-zinc-200 rounded focus:outline-none focus:border-teal-600 transition-colors"
                    />
                </div>

                <div>
                    <label for="jenis-barang" class="block text-xs font-medium text-zinc-700 mb-1.5">
                        Jenis Barang (Dedikasi)
                    </label>
                    <select
                        id="jenis-barang"
                        bind:value={jenisBarang}
                        class="w-full px-3 py-2 text-xs bg-white border border-zinc-200 rounded focus:outline-none focus:border-teal-600 transition-colors text-zinc-800"
                    >
                        {#each opsiJenisBarang as opsi}
                            <option value={opsi}>{opsi}</option>
                        {/each}
                    </select>
                </div>

                <div>
                    <label for="deskripsi-barang" class="block text-xs font-medium text-zinc-700 mb-1.5">
                        Deskripsi
                    </label>
                    <textarea
                        id="deskripsi-barang"
                        bind:value={deskripsi}
                        rows="4"
                        placeholder="Deskripsi barang..."
                        class="w-full px-3 py-2 text-xs bg-white border border-zinc-200 rounded focus:outline-none focus:border-teal-600 transition-colors resize-none"
                    ></textarea>
                </div>
            </div>

            <!-- Upload Media Section -->
            <div class="pt-4 border-t border-zinc-200 space-y-3">
                <span class="block text-xs font-medium text-zinc-700">Media</span>
                <div class="grid grid-cols-2 gap-3">
                    <!-- Foto Slot -->
                    <div>
                        {#if fotoUrl}
                            <div class="relative w-full h-28 border border-zinc-200 rounded overflow-hidden bg-zinc-50 group">
                                <img src={fotoUrl} alt="Foto Utama" class="w-full h-full object-cover" />
                                <button
                                    type="button"
                                    onclick={removeFoto}
                                    class="absolute top-1.5 right-1.5 px-2 py-0.5 bg-zinc-900/80 text-white text-[10px] rounded hover:bg-zinc-900 transition-colors"
                                >
                                    Hapus
                                </button>
                            </div>
                        {:else}
                            <label class="flex flex-col items-center justify-center w-full h-28 border border-dashed border-zinc-200 hover:border-teal-600 rounded cursor-pointer bg-zinc-50/50 hover:bg-white transition-colors">
                                <span class="text-xs text-zinc-500">Upload Foto</span>
                                <input type="file" accept="image/*" onchange={handleFotoUpload} class="hidden" />
                            </label>
                        {/if}
                    </div>

                    <!-- Video Slot -->
                    <div>
                        {#if videoUrl}
                            <div class="relative w-full h-28 border border-zinc-200 rounded overflow-hidden bg-black group">
                                <video src={videoUrl} class="w-full h-full object-cover"></video>
                                <button
                                    type="button"
                                    onclick={removeVideo}
                                    class="absolute top-1.5 right-1.5 px-2 py-0.5 bg-zinc-900/80 text-white text-[10px] rounded hover:bg-zinc-900 transition-colors"
                                >
                                    Hapus
                                </button>
                            </div>
                        {:else}
                            <label class="flex flex-col items-center justify-center w-full h-28 border border-dashed border-zinc-200 hover:border-teal-600 rounded cursor-pointer bg-zinc-50/50 hover:bg-white transition-colors">
                                <span class="text-xs text-zinc-500">Upload Video</span>
                                <input type="file" accept="video/*" onchange={handleVideoUpload} class="hidden" />
                            </label>
                        {/if}
                    </div>
                </div>
            </div>

            <!-- Meta Timestamps & Action -->
            <div class="pt-4 border-t border-zinc-200 flex items-center justify-between">
                <div class="text-[11px] font-mono text-zinc-400 space-y-0.5">
                    <div>Dibuat: {formatTanggal(createdAt)}</div>
                    <div>Diperbarui: {formatTanggal(updatedAt)}</div>
                </div>

                <button
                    type="submit"
                    disabled={isSaving}
                    class="px-4 py-2 text-xs font-medium text-white bg-zinc-900 hover:bg-zinc-800 disabled:bg-zinc-300 rounded transition-colors"
                >
                    {isSaving ? 'Menyimpan...' : 'Simpan Perubahan'}
                </button>
            </div>
        </form>

        <!-- SIDEBAR / PREVIEW & STATS -->
        <aside class="lg:col-span-5 space-y-6">
            <!-- Kelengkapan Score Bar -->
            <div class="p-4 bg-white border border-zinc-200 rounded space-y-2">
                <div class="flex items-center justify-between text-xs">
                    <span class="font-medium text-zinc-700">Kelengkapan Data</span>
                    <span class="font-mono text-teal-600 font-semibold">{kelengkapanScore}%</span>
                </div>
                <div class="w-full h-1.5 bg-zinc-100 rounded-full overflow-hidden">
                    <div
                        class="h-full bg-teal-600 transition-all duration-300"
                        style="width: {kelengkapanScore}%"
                    ></div>
                </div>
            </div>

            <!-- Preview Component Minimalis -->
            <div class="bg-white border border-zinc-200 rounded p-4 space-y-4">
                <div class="text-xs font-mono text-zinc-400 uppercase tracking-wider">Preview</div>
                
                <div class="flex gap-4">
                    <!-- Photo Container -->
                    <div class="w-20 h-20 bg-zinc-100 border border-zinc-200 rounded shrink-0 overflow-hidden flex items-center justify-center">
                        {#if fotoUrl}
                            <img src={fotoUrl} alt="Preview" class="w-full h-full object-cover" />
                        {:else}
                            <span class="text-[10px] text-zinc-400 font-mono">NO MEDIA</span>
                        {/if}
                    </div>

                    <!-- Detail Text -->
                    <div class="flex-1 min-w-0 space-y-1">
                        <div class="flex items-center gap-2">
                            <span class="text-[10px] font-mono uppercase px-1.5 py-0.5 bg-zinc-100 text-zinc-600 rounded">
                                {jenisBarang || 'Umum'}
                            </span>
                            <span class="inline-flex items-center gap-1 text-[10px] text-teal-700 font-medium">
                                <span class="w-1 h-1 rounded-full bg-teal-600"></span>
                                Aktif
                            </span>
                        </div>
                        <h3 class="text-xs font-medium text-zinc-900 truncate">
                            {namaBarang || 'Tanpa Nama'}
                        </h3>
                        <p class="text-[11px] text-zinc-500 line-clamp-2 leading-relaxed">
                            {deskripsi || 'Belum ada deskripsi.'}
                        </p>
                    </div>
                </div>

                <!-- Bottom Metric Preview -->
                <div class="pt-3 border-t border-zinc-100 flex items-center justify-between text-[11px] font-mono text-zinc-500">
                    <span>12 Varian</span>
                    <span>Stok: 450</span>
                </div>
            </div>
        </aside>
    </div>
</section>