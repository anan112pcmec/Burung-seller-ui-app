<script lang="ts">
    // State management menggunakan Svelte 5 Runes
    let selectedIndex = $state<number | null>(0); // Default memilih kategori pertama

    // Data Kategori Barang
    const daftarKategori = $state([
        {
            id_kategori_barang: 8821,
            id_seller_kategori_barang: 4091,
            id_barang_induk_kategori: 102,
            id_alamat_gudang_kategori_barang: 12,
            id_rekening_kategori_barang: 5,
            nama_kategori_barang: "Variasi Hitam XL",
            deskripsi_kategori_barang: "Kategori variasi ukuran besar dengan material linen murni premium.",
            warna_kategori_barang: "Hitam",
            stok_kategori_barang: 45,
            harga_kategori_barang: 149000,
            diskon: 26000,
            berat_gram_kategori_barang: 250,
            dimensi_panjang_cm_kategori_barang: 30,
            dimensi_tinggi_cm_kategori_barang: 20,
            sku_kategori: "KTG-8821",
            is_original_kategori_barang: true,
            created_at: "2026-08-15T10:30:00Z",
            updated_at: "2026-08-31T09:12:44Z",
            gudang_name: "Gudang Utama Jakarta",
            rekening_info: "BCA • 8829101",
            foto: ["/placeholder-black.jpg"]
        },
        {
            id_kategori_barang: 8822,
            id_seller_kategori_barang: 4091,
            id_barang_induk_kategori: 102,
            id_alamat_gudang_kategori_barang: 12,
            id_rekening_kategori_barang: 5,
            nama_kategori_barang: "Variasi Putih M",
            deskripsi_kategori_barang: "Kategori variasi warna putih bersih ukuran sedang.",
            warna_kategori_barang: "Putih",
            stok_kategori_barang: 12,
            harga_kategori_barang: 139000,
            diskon: 0,
            berat_gram_kategori_barang: 230,
            dimensi_panjang_cm_kategori_barang: 28,
            dimensi_tinggi_cm_kategori_barang: 18,
            sku_kategori: "KTG-8822",
            is_original_kategori_barang: false,
            created_at: "2026-08-16T11:00:00Z",
            updated_at: "2026-08-30T14:20:00Z",
            gudang_name: "Gudang Cabang Bandung",
            rekening_info: "Mandiri • 1310029",
            foto: ["/placeholder-white.jpg"]
        }
    ]);

    let selectedData = $derived(selectedIndex !== null ? daftarKategori[selectedIndex] : null);
</script>

<section id="details-kategori-barang" class="w-full bg-white border border-zinc-200 rounded-lg overflow-hidden font-sans">
    <div class="grid grid-cols-1 lg:grid-cols-[320px_1fr] divide-y lg:divide-y-0 lg:divide-x divide-zinc-200">
        
        <!-- KOLOM KIRI: LIST SIMPEL VERTIKAL (SCROLL DOWN) -->
        <div class="p-3 flex flex-col gap-2 overflow-y-auto max-h-[38rem] scrollbar-thin scrollbar-thumb-zinc-200">
            <!-- Tombol Tambah Kategori -->
            <button
                type="button"
                class="w-full border border-dashed border-zinc-300 hover:border-zinc-400 rounded-md bg-zinc-50/70 hover:bg-zinc-100 p-2.5 flex items-center justify-center gap-2 transition-colors cursor-pointer group"
            >
                <div class="w-5 h-5 rounded-full bg-zinc-200 group-hover:bg-zinc-900 group-hover:text-white text-zinc-700 flex items-center justify-center transition-colors">
                    <svg xmlns="http://www.w3.org/2000/svg" class="w-3.5 h-3.5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
                        <path d="M5 12h14"/><path d="M12 5v14"/>
                    </svg>
                </div>
                <span class="text-xs font-bold text-zinc-700">Tambah Kategori Baru</span>
            </button>

            <!-- Render Loop List Simpel -->
            {#each daftarKategori as item, index}
                <button
                    type="button"
                    onclick={() => selectedIndex = index}
                    class="w-full text-left p-2.5 rounded-lg border transition-all duration-150 flex items-center gap-3 cursor-pointer select-none
                    {selectedIndex === index 
                        ? 'border-zinc-900 bg-zinc-900 text-white shadow-sm' 
                        : 'border-zinc-200 hover:border-zinc-300 hover:bg-zinc-50 text-zinc-800'}"
                >
                    <!-- Thumbnail Foto -->
                    <div class="w-11 h-11 rounded-md shrink-0 border overflow-hidden flex items-center justify-center relative
                        {selectedIndex === index ? 'bg-zinc-800 border-zinc-700' : 'bg-zinc-100 border-zinc-200'}">
                        <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5 {selectedIndex === index ? 'text-zinc-500' : 'text-zinc-400'}" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5">
                            <rect width="18" height="18" x="3" y="3" rx="2" ry="2"/>
                            <circle cx="9" cy="9" r="2"/>
                            <path d="m21 15-3.086-3.086a2 2 0 0 0-2.828 0L6 21"/>
                        </svg>
                    </div>

                    <!-- Informasi Nama & Identitas Ringkas -->
                    <div class="flex-1 min-w-0 leading-tight">
                        <div class="flex items-center gap-1.5">
                            <span class="text-xs font-bold truncate">{item.nama_kategori_barang}</span>
                            {#if item.is_original_kategori_barang}
                                <span class="text-[8px] font-mono px-1 py-0.2 rounded uppercase shrink-0 
                                    {selectedIndex === index ? 'bg-zinc-100 text-zinc-900 font-bold' : 'bg-zinc-900 text-white'}">
                                    Ori
                                </span>
                            {/if}
                        </div>

                        <div class="flex items-center gap-2 mt-1 text-[10px] font-mono 
                            {selectedIndex === index ? 'text-zinc-400' : 'text-zinc-400'}">
                            <span>{item.sku_kategori}</span>
                            <span>•</span>
                            <span>{item.warna_kategori_barang}</span>
                            <span>•</span>
                            <span class={selectedIndex === index ? 'text-teal-300 font-semibold' : 'text-teal-600 font-semibold'}>
                                Stok: {item.stok_kategori_barang}
                            </span>
                        </div>
                    </div>

                    <!-- Harga Ringkas -->
                    <div class="text-right shrink-0">
                        <span class="text-xs font-bold font-mono block">
                            Rp {(item.harga_kategori_barang / 1000).toFixed(0)}k
                        </span>
                    </div>
                </button>
            {/each}
        </div>

        <!-- KOLOM KANAN: DETAIL KATEGORI YANG DIPILIH -->
        <div class="p-6 bg-white min-h-[24rem]">
            {#if selectedData}
                <div class="space-y-6">
                    <!-- Header Informasi SKU & Nama -->
                    <div class="flex items-start justify-between border-b border-zinc-100 pb-4">
                        <div>
                            <div class="flex items-center gap-2 mb-1">
                                <span class="text-xs font-mono font-semibold px-2 py-0.5 rounded bg-zinc-100 text-zinc-700">
                                    SKU: {selectedData.sku_kategori}
                                </span>
                                {#if selectedData.is_original_kategori_barang}
                                    <span class="text-xs font-mono font-semibold px-2 py-0.5 rounded bg-zinc-900 text-white">
                                        Original Kategori
                                    </span>
                                {/if}
                            </div>
                            <h3 class="text-xl font-bold text-zinc-900">{selectedData.nama_kategori_barang}</h3>
                        </div>
                        <div class="text-right">
                            <span class="text-[10px] font-mono text-zinc-400 uppercase block">Harga Jual</span>
                            <span class="text-2xl font-bold font-mono text-zinc-900">
                                Rp {selectedData.harga_kategori_barang.toLocaleString('id-ID')}
                            </span>
                        </div>
                    </div>

                    <!-- Layout Foto & Detail Spec -->
                    <div class="grid grid-cols-1 md:grid-cols-12 gap-6">
                        <!-- Galeri Foto Kategori -->
                        <div class="md:col-span-5 space-y-3">
                            <span class="text-[10px] font-mono text-zinc-400 uppercase tracking-wider block">Foto Kategori</span>
                            <div class="aspect-square rounded-lg bg-zinc-100 border border-zinc-200 overflow-hidden relative flex items-center justify-center">
                                <svg xmlns="http://www.w3.org/2000/svg" class="w-10 h-10 text-zinc-300" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5">
                                    <rect width="18" height="18" x="3" y="3" rx="2" ry="2"/>
                                    <circle cx="9" cy="9" r="2"/>
                                    <path d="m21 15-3.086-3.086a2 2 0 0 0-2.828 0L6 21"/>
                                </svg>
                            </div>
                        </div>

                        <!-- Detail Atribut GORM -->
                        <div class="md:col-span-7 space-y-4">
                            <div>
                                <span class="text-[10px] font-mono text-zinc-400 uppercase tracking-wider block mb-1">Deskripsi Kategori</span>
                                <p class="text-xs text-zinc-600 bg-zinc-50 p-3 rounded border border-zinc-200 leading-relaxed">
                                    {selectedData.deskripsi_kategori_barang}
                                </p>
                            </div>

                            <!-- Spec Metrics Grid -->
                            <div class="grid grid-cols-2 gap-3 text-xs font-mono">
                                <div class="p-2.5 rounded bg-zinc-50 border border-zinc-200">
                                    <span class="block text-[9px] text-zinc-400 uppercase">Stok Tersedia</span>
                                    <span class="font-bold text-zinc-900 text-sm">{selectedData.stok_kategori_barang} pcs</span>
                                </div>
                                <div class="p-2.5 rounded bg-zinc-50 border border-zinc-200">
                                    <span class="block text-[9px] text-zinc-400 uppercase">Warna Produk</span>
                                    <span class="font-bold text-zinc-900 text-sm">{selectedData.warna_kategori_barang}</span>
                                </div>
                                <div class="p-2.5 rounded bg-zinc-50 border border-zinc-200">
                                    <span class="block text-[9px] text-zinc-400 uppercase">Berat (Gram)</span>
                                    <span class="font-bold text-zinc-900 text-sm">{selectedData.berat_gram_kategori_barang} gr</span>
                                </div>
                                <div class="p-2.5 rounded bg-zinc-50 border border-zinc-200">
                                    <span class="block text-[9px] text-zinc-400 uppercase">Dimensi (P x L)</span>
                                    <span class="font-bold text-zinc-900 text-sm">{selectedData.dimensi_panjang_cm_kategori_barang} x {selectedData.dimensi_tinggi_cm_kategori_barang} cm</span>
                                </div>
                            </div>

                            <!-- Relasi Metadata Foreign Keys -->
                            <div class="space-y-4 pt-2 text-xs font-mono">
                                <div class="flex border border-slate-900 justify-between p-2 rounded bg-zinc-100/60 text-zinc-700">
                                    <span class="text-zinc-500">Gudang Penyimpanan -> </span>
                                    <span class="font-bold">{selectedData.gudang_name}</span>
                                </div>
                                <div class="flex border border-slate-900 justify-between p-2 rounded bg-zinc-100/60 text-zinc-700">
                                    <span class="text-zinc-500">Rekening Tujuan -> </span>
                                    <span class="font-bold">{selectedData.rekening_info}</span>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- Metadata Audit -->
                    <div class="pt-4 border-t border-zinc-100 flex items-center justify-between text-[10px] font-mono text-zinc-400">
                        <span>Created: {new Date(selectedData.created_at).toLocaleString('id-ID')}</span>
                        <span>Updated: {new Date(selectedData.updated_at).toLocaleString('id-ID')}</span>
                    </div>
                </div>
            {:else}
                <div class="h-full flex flex-col items-center justify-center text-zinc-400 text-xs py-12">
                    <svg xmlns="http://www.w3.org/2000/svg" class="w-8 h-8 mb-2 stroke-1" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <circle cx="12" cy="12" r="10"/>
                        <path d="M12 8v4m0 4h.01"/>
                    </svg>
                    Pilih salah satu kategori di sebelah kiri untuk melihat detail lengkap.
                </div>
            {/if}
        </div>

    </div>
</section>