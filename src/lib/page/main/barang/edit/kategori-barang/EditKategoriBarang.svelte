<script lang="ts">
    // Interface disesuaikan dengan struct Go KategoriBarang
    interface KategoriBarang {
        id_kategori_barang: number;
        id_seller_kategori_barang: number;
        id_barang_induk_kategori: number;
        id_alamat_gudang_kategori_barang: number | null;
        id_rekening_kategori_barang: number | null;
        nama_kategori_barang: string;
        deskripsi_kategori_barang: string;
        warna_kategori_barang: string;
        stok_kategori_barang: number;
        harga_kategori_barang: number;
        berat_gram_kategori_barang: number;
        dimensi_panjang_cm_kategori_barang: number;
        dimensi_tinggi_cm_kategori_barang: number;
        sku_kategori: string;
        is_original_kategori_barang: boolean;
    }

    let form: KategoriBarang = $state({
        id_kategori_barang: 101,
        id_seller_kategori_barang: 1,
        id_barang_induk_kategori: 50,
        id_alamat_gudang_kategori_barang: 12,
        id_rekening_kategori_barang: 4,
        nama_kategori_barang: "Kaos Oversize Cotton Combed 30s - Hitam XL",
        deskripsi_kategori_barang: "Bahan nyaman, adem, tidak panas saat dipakai seharian. Jahitan rapi standar distro.",
        warna_kategori_barang: "#18181b",
        stok_kategori_barang: 150,
        harga_kategori_barang: 85000,
        berat_gram_kategori_barang: 250,
        dimensi_panjang_cm_kategori_barang: 30,
        dimensi_tinggi_cm_kategori_barang: 20,
        sku_kategori: "TSHIRT-BLK-XL",
        is_original_kategori_barang: true
    });

    const opsiAlamatGudang = [
        { id: 12, nama: "Gudang Utama - Jakarta Barat" },
        { id: 14, nama: "Gudang Cabang - Surabaya" }
    ];

    const opsiRekening = [
        { id: 4, nama: "BCA - 882019203 (a.n Toko Utama)" },
        { id: 7, nama: "Mandiri - 140001928301 (a.n Toko Utama)" }
    ];

    let kelengkapanScore = $derived.by(() => {
        let total = 0;
        const fields = [
            form.nama_kategori_barang,
            form.sku_kategori,
            form.harga_kategori_barang > 0,
            form.stok_kategori_barang >= 0,
            form.deskripsi_kategori_barang,
            form.warna_kategori_barang,
            form.berat_gram_kategori_barang > 0,
            form.dimensi_panjang_cm_kategori_barang > 0 && form.dimensi_tinggi_cm_kategori_barang > 0,
            form.id_alamat_gudang_kategori_barang,
            form.id_rekening_kategori_barang
        ];

        fields.forEach(field => {
            if (field) total += 10;
        });

        return total;
    });

    function handleSubmit(e: SubmitEvent) {
        e.preventDefault();
        console.log("Payload data dikirim ke backend Go:", JSON.stringify(form, null, 2));
        alert("Kategori barang berhasil diperbarui!");
    }
</script>

<section id="edit-kategori-barang" class="max-w-6xl mx-auto px-6 py-10 text-slate-800">
    <!-- Header Minimalis -->
    <div class="mb-10 flex flex-col sm:flex-row sm:items-center justify-between gap-4 border-b border-zinc-200 pb-5">
        <div>
            <h1 class="text-xl font-semibold text-zinc-900 tracking-tight">Edit Kategori Barang</h1>
            <p class="text-xs text-slate-500 mt-1">Kelola informasi varian, inventaris, dan spesifikasi pengiriman.</p>
        </div>
        <div class="flex items-center gap-2.5">
            <button 
                type="button" 
                class="px-4 py-2 text-xs font-medium text-slate-600 bg-white border border-zinc-200 rounded-lg hover:bg-zinc-50 transition-colors"
            >
                Batal
            </button>
            <button 
                type="submit" 
                form="form-kategori"
                class="px-4 py-2 text-xs font-medium text-white bg-teal-600 rounded-lg hover:bg-teal-700 active:bg-teal-800 transition-colors"
            >
                Simpan Perubahan
            </button>
        </div>
    </div>

    <!-- Layout Utama Grid -->
    <div class="grid grid-cols-1 lg:grid-cols-12 gap-10 items-start">
        
        <!-- Form Utama (7 Cols) -->
        <div class="lg:col-span-7 space-y-8">
            <form id="form-kategori" onsubmit={handleSubmit} class="space-y-8">
                
                <!-- Section 1: Informasi Dasar -->
                <div class="space-y-4">
                    <h2 class="text-xs font-semibold text-zinc-400 uppercase tracking-wider">Informasi Varian</h2>

                    <div class="space-y-4">
                        <div>
                            <label for="nama_kategori" class="block text-xs font-medium text-slate-700 mb-1.5">Nama Varian <span class="text-teal-600">*</span></label>
                            <input 
                                type="text" 
                                id="nama_kategori"
                                bind:value={form.nama_kategori_barang}
                                maxlength="120"
                                required
                                class="w-full px-3 py-2 text-xs text-zinc-900 bg-white border border-zinc-200 rounded-lg focus:outline-none focus:border-teal-600 transition-colors placeholder:text-zinc-400"
                                placeholder="Contoh: Hitam - XL"
                            />
                        </div>

                        <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                            <div>
                                <label for="sku" class="block text-xs font-medium text-slate-700 mb-1.5">Kode SKU <span class="text-teal-600">*</span></label>
                                <input 
                                    type="text" 
                                    id="sku"
                                    bind:value={form.sku_kategori}
                                    maxlength="20"
                                    required
                                    class="w-full px-3 py-2 text-xs font-mono text-zinc-900 bg-white border border-zinc-200 rounded-lg focus:outline-none focus:border-teal-600 transition-colors uppercase placeholder:text-zinc-400"
                                    placeholder="SKU-VAR-001"
                                />
                            </div>

                            <div>
                                <label for="warna" class="block text-xs font-medium text-slate-700 mb-1.5">Warna Atribut</label>
                                <div class="flex gap-2">
                                    <input 
                                        type="color" 
                                        id="warna_picker"
                                        bind:value={form.warna_kategori_barang}
                                        class="h-8 w-10 rounded border border-zinc-200 bg-white cursor-pointer p-0.5"
                                    />
                                    <input 
                                        type="text" 
                                        id="warna"
                                        bind:value={form.warna_kategori_barang}
                                        class="w-full px-3 py-2 text-xs text-zinc-900 bg-white border border-zinc-200 rounded-lg focus:outline-none focus:border-teal-600 transition-colors placeholder:text-zinc-400"
                                        placeholder="#18181b / Hitam"
                                    />
                                </div>
                            </div>
                        </div>

                        <div>
                            <label for="deskripsi" class="block text-xs font-medium text-slate-700 mb-1.5">Deskripsi Varian</label>
                            <textarea 
                                id="deskripsi"
                                bind:value={form.deskripsi_kategori_barang}
                                rows="3"
                                class="w-full px-3 py-2 text-xs text-zinc-900 bg-white border border-zinc-200 rounded-lg focus:outline-none focus:border-teal-600 transition-colors placeholder:text-zinc-400 resize-none"
                                placeholder="Catatan spesifik varian..."
                            ></textarea>
                        </div>

                        <div class="pt-1">
                            <label class="inline-flex items-center gap-2.5 cursor-pointer">
                                <input 
                                    type="checkbox" 
                                    bind:checked={form.is_original_kategori_barang}
                                    class="w-3.5 h-3.5 text-teal-600 rounded border-zinc-300 focus:ring-0 focus:ring-offset-0"
                                />
                                <span class="text-xs text-slate-700">Produk Terverifikasi Original</span>
                            </label>
                        </div>
                    </div>
                </div>

                <div class="border-t border-zinc-100"></div>

                <!-- Section 2: Harga & Stok -->
                <div class="space-y-4">
                    <h2 class="text-xs font-semibold text-zinc-400 uppercase tracking-wider">Harga & Stok</h2>

                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                        <div>
                            <label for="harga" class="block text-xs font-medium text-slate-700 mb-1.5">Harga Jual (Rp) <span class="text-teal-600">*</span></label>
                            <input 
                                type="number" 
                                id="harga"
                                bind:value={form.harga_kategori_barang}
                                min="0"
                                required
                                class="w-full px-3 py-2 text-xs font-mono text-zinc-900 bg-white border border-zinc-200 rounded-lg focus:outline-none focus:border-teal-600 transition-colors"
                            />
                        </div>

                        <div>
                            <label for="stok" class="block text-xs font-medium text-slate-700 mb-1.5">Jumlah Stok <span class="text-teal-600">*</span></label>
                            <input 
                                type="number" 
                                id="stok"
                                bind:value={form.stok_kategori_barang}
                                min="0"
                                required
                                class="w-full px-3 py-2 text-xs font-mono text-zinc-900 bg-white border border-zinc-200 rounded-lg focus:outline-none focus:border-teal-600 transition-colors"
                            />
                        </div>
                    </div>
                </div>

                <div class="border-t border-zinc-100"></div>

                <!-- Section 3: Dimensi & Logistik -->
                <div class="space-y-4">
                    <h2 class="text-xs font-semibold text-zinc-400 uppercase tracking-wider">Spesifikasi Pengiriman</h2>

                    <div class="grid grid-cols-3 gap-3">
                        <div>
                            <label for="berat" class="block text-xs font-medium text-slate-700 mb-1.5">Berat (Gram)</label>
                            <input 
                                type="number" 
                                id="berat"
                                bind:value={form.berat_gram_kategori_barang}
                                min="0"
                                class="w-full px-3 py-2 text-xs font-mono text-zinc-900 bg-white border border-zinc-200 rounded-lg focus:outline-none focus:border-teal-600 transition-colors"
                            />
                        </div>

                        <div>
                            <label for="panjang" class="block text-xs font-medium text-slate-700 mb-1.5">Panjang (cm)</label>
                            <input 
                                type="number" 
                                id="panjang"
                                bind:value={form.dimensi_panjang_cm_kategori_barang}
                                min="0"
                                class="w-full px-3 py-2 text-xs font-mono text-zinc-900 bg-white border border-zinc-200 rounded-lg focus:outline-none focus:border-teal-600 transition-colors"
                            />
                        </div>

                        <div>
                            <label for="tinggi" class="block text-xs font-medium text-slate-700 mb-1.5">Lebar/Tinggi (cm)</label>
                            <input 
                                type="number" 
                                id="tinggi"
                                bind:value={form.dimensi_tinggi_cm_kategori_barang}
                                min="0"
                                class="w-full px-3 py-2 text-xs font-mono text-zinc-900 bg-white border border-zinc-200 rounded-lg focus:outline-none focus:border-teal-600 transition-colors"
                            />
                        </div>
                    </div>
                </div>

                <div class="border-t border-zinc-100"></div>

                <!-- Section 4: Relasi Gudang & Rekening -->
                <div class="space-y-4">
                    <h2 class="text-xs font-semibold text-zinc-400 uppercase tracking-wider">Pemetaan Lokasi & Finansial</h2>

                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                        <div>
                            <label for="gudang" class="block text-xs font-medium text-slate-700 mb-1.5">Lokasi Gudang</label>
                            <select 
                                id="gudang"
                                bind:value={form.id_alamat_gudang_kategori_barang}
                                class="w-full px-3 py-2 text-xs text-zinc-900 bg-white border border-zinc-200 rounded-lg focus:outline-none focus:border-teal-600 transition-colors"
                            >
                                <option value={null}>-- Pilih Gudang --</option>
                                {#each opsiAlamatGudang as g}
                                    <option value={g.id}>{g.nama}</option>
                                {/each}
                            </select>
                        </div>

                        <div>
                            <label for="rekening" class="block text-xs font-medium text-slate-700 mb-1.5">Rekening Penampung</label>
                            <select 
                                id="rekening"
                                bind:value={form.id_rekening_kategori_barang}
                                class="w-full px-3 py-2 text-xs text-zinc-900 bg-white border border-zinc-200 rounded-lg focus:outline-none focus:border-teal-600 transition-colors"
                            >
                                <option value={null}>-- Pilih Rekening --</option>
                                {#each opsiRekening as r}
                                    <option value={r.id}>{r.nama}</option>
                                {/each}
                            </select>
                        </div>
                    </div>
                </div>

            </form>
        </div>

        <!-- Kolom Kanan: Preview & Progress (5 Cols) -->
        <div class="lg:col-span-5 lg:sticky lg:top-8 space-y-6">
            
            <!-- Minimal Progress Bar -->
            <div class="bg-white border border-zinc-200 rounded-xl p-4 space-y-2">
                <div class="flex items-center justify-between text-xs">
                    <span class="font-medium text-slate-700">Kelengkapan Atribut</span>
                    <span class="font-mono text-teal-600 font-semibold">{kelengkapanScore}%</span>
                </div>
                <div class="w-full h-1.5 bg-zinc-100 rounded-full overflow-hidden">
                    <div
                        class="h-full bg-teal-600 rounded-full transition-all duration-300"
                        style="width: {kelengkapanScore}%"
                    ></div>
                </div>
            </div>

            <!-- Pratinjau Ringkas -->
            <div class="bg-white border border-zinc-200 rounded-xl p-5 space-y-4">
                <div class="flex items-center justify-between border-b border-zinc-100 pb-3">
                    <span class="text-xs font-semibold text-zinc-400 uppercase tracking-wider">Pratinjau</span>
                    <span class="text-[11px] font-mono text-slate-400">ID #{form.id_kategori_barang}</span>
                </div>

                <div class="space-y-3">
                    <div class="flex items-center justify-between gap-2">
                        <span class="text-[11px] font-mono font-medium text-slate-600 bg-zinc-100 px-2 py-0.5 rounded">
                            {form.sku_kategori || 'SKU-NONE'}
                        </span>

                        {#if form.is_original_kategori_barang}
                            <span class="text-[11px] font-medium text-teal-700 bg-teal-50 px-2 py-0.5 rounded border border-teal-100">
                                Original
                            </span>
                        {/if}
                    </div>

                    <div>
                        <h3 class="text-xs font-semibold text-zinc-900 leading-snug">
                            {form.nama_kategori_barang || 'Nama varian belum diisi'}
                        </h3>
                        {#if form.deskripsi_kategori_barang}
                            <p class="text-xs text-slate-500 line-clamp-2 mt-1">{form.deskripsi_kategori_barang}</p>
                        {/if}
                    </div>

                    {#if form.warna_kategori_barang}
                        <div class="flex items-center gap-2 text-xs text-slate-600 pt-1">
                            <span class="w-3 h-3 rounded-full border border-zinc-300" style="background-color: {form.warna_kategori_barang}"></span>
                            <span class="font-mono text-[11px] text-slate-500">{form.warna_kategori_barang}</span>
                        </div>
                    {/if}

                    <div class="pt-3 border-t border-zinc-100 flex items-end justify-between">
                        <div>
                            <span class="text-[10px] text-slate-400 block uppercase">Harga</span>
                            <span class="text-sm font-semibold text-zinc-900 font-mono">
                                Rp {form.harga_kategori_barang.toLocaleString('id-ID')}
                            </span>
                        </div>
                        <div class="text-right">
                            <span class="text-[10px] text-slate-400 block uppercase">Stok</span>
                            <span class="text-xs font-medium text-slate-700 font-mono">
                                {form.stok_kategori_barang} unit
                            </span>
                        </div>
                    </div>

                    <div class="pt-2 flex items-center justify-between text-[11px] font-mono text-slate-500 bg-zinc-50 p-2 rounded">
                        <span>Dimensi: {form.dimensi_panjang_cm_kategori_barang}x{form.dimensi_tinggi_cm_kategori_barang} cm</span>
                        <span>{form.berat_gram_kategori_barang} gr</span>
                    </div>
                </div>
            </div>

        </div>

    </div>
</section>