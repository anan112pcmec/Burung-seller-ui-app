<script lang="ts"></script>

<section id="details-barang-induk" class="w-full bg-white rounded-lg p-6 font-sans">
    <div class="border-b border-zinc-200 pb-3 mb-6 flex items-center justify-between">
        <div>
            <h3 class="text-lg font-bold uppercase tracking-tight text-zinc-900 leading-none">
                Detail Barang Induk
            </h3>
            <span class="text-[10px] font-mono text-zinc-400 mt-1 block">ID: #BI-80291</span>
        </div>
        <span class="px-2.5 py-1 text-xs font-semibold rounded-full bg-zinc-100 text-zinc-700 font-mono">
            Jenis: Pakaian
        </span>
    </div>

    <div class="grid grid-cols-1 lg:grid-cols-12 gap-6 items-start">
        
        <!-- SISI KIRI: MEDIA (FOTO & VIDEO BARANG INDUK) -->
        <div class="lg:col-span-5 space-y-4">
            <!-- Frame Utama (Preview Gambar / Video Switch) -->
            <div class="w-full aspect-square rounded-lg bg-zinc-100 border border-zinc-200 overflow-hidden relative group flex items-center justify-center">
                <svg xmlns="http://www.w3.org/2000/svg" class="w-12 h-12 text-zinc-300" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5">
                    <rect width="18" height="18" x="3" y="3" rx="2" ry="2"/>
                    <circle cx="9" cy="9" r="2"/>
                    <path d="m21 15-3.086-3.086a2 2 0 0 0-2.828 0L6 21"/>
                </svg>
                <span class="absolute top-3 left-3 bg-zinc-900/80 text-white text-[9px] font-mono px-2 py-0.5 rounded uppercase tracking-wider">
                    Foto Utama
                </span>
            </div>

            <!-- Thumbnail Grid Media (Foto Tambahan & Video) -->
          
        </div>

        <!-- SISI KANAN: FIELD INFORMASI STRUCT BARANG INDUK -->
        <div class="lg:col-span-7 space-y-4">
            <!-- Nama Barang & Harga Base -->
            <div>
                <span class="text-[10px] font-mono text-zinc-400 uppercase tracking-wider block mb-1">Nama Barang Induk</span>
                <h2 class="text-xl font-bold text-zinc-900 leading-tight">
                    Kemeja Oversize Linen Premium
                </h2>
                <div class="mt-2 flex items-baseline gap-2">
                    <span class="text-xs text-zinc-400">Harga Standar Kategori:</span>
                    <span class="text-lg font-bold font-mono text-zinc-900">Rp 149.000</span>
                </div>
            </div>

            <!-- Deskripsi Barang -->
            <div>
                <span class="text-[10px] font-mono text-zinc-400 uppercase tracking-wider block mb-1">Deskripsi</span>
                <p class="text-xs text-zinc-600 leading-relaxed bg-zinc-50 p-3 rounded-md border border-zinc-200">
                    Kemeja dengan potongan oversize berbahan 100% linen murni premium. Bahan adem, tidak menerawang, cocok digunakan untuk aktivitas kasual harian maupun formal.
                </p>
            </div>

            <!-- Metadata Struct GORM (Seller, Diskon, Kategori Original) -->
            <div class="grid grid-cols-2 sm:grid-cols-4 gap-2 bg-zinc-50 p-3 rounded-md border border-zinc-200 text-xs font-mono">
                <div>
                    <span class="block text-[9px] text-zinc-400 uppercase">Seller ID</span>
                    <span class="font-bold text-zinc-700">#SEL-4091</span>
                </div>
                <div>
                    <span class="block text-[9px] text-zinc-400 uppercase">ID Diskon</span>
                    <span class="font-bold text-zinc-700">#DSK-102</span>
                </div>
                <div>
                    <span class="block text-[9px] text-zinc-400 uppercase">Original Kategori</span>
                    <span class="font-bold text-zinc-700">#KTG-08</span>
                </div>
                <div>
                    <span class="block text-[9px] text-zinc-400 uppercase">Harga Kategori</span>
                    <span class="font-bold text-zinc-700">Rp 149k</span>
                </div>
            </div>

            <!-- Audit Timestamps (CreatedAt, UpdatedAt) -->
            <div class="pt-3 border-t border-zinc-100 flex items-center justify-between text-[10px] font-mono text-zinc-400">
                <span>Created At: 2026-08-15 10:30:00</span>
                <span>Updated At: 2026-08-31 09:12:44</span>
            </div>

              <div class="flex items-start gap-2 overflow-x-auto scrollbar-none">
                <!-- Photo Thumbnail Active -->
                <div class="h-40 w-48 aspect-square rounded-md bg-zinc-100 border-2 border-zinc-900 flex items-center justify-center cursor-pointer relative overflow-hidden">
                    <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5 text-zinc-400" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5">
                        <rect width="18" height="18" x="3" y="3" rx="2" ry="2"/>
                        <circle cx="9" cy="9" r="2"/>
                        <path d="m21 15-3.086-3.086a2 2 0 0 0-2.828 0L6 21"/>
                    </svg>
                </div>

                <!-- Photo Thumbnail -->
                <div class="h-40 w-48  aspect-square rounded-md bg-zinc-100 border border-zinc-200 hover:border-zinc-400 flex items-center justify-center cursor-pointer transition-colors">
                    <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5 text-zinc-400" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5">
                        <rect width="18" height="18" x="3" y="3" rx="2" ry="2"/>
                        <circle cx="9" cy="9" r="2"/>
                        <path d="m21 15-3.086-3.086a2 2 0 0 0-2.828 0L6 21"/>
                    </svg>
                </div>

                <!-- Video Thumbnail -->
                <div class="h-40 w-48  aspect-square rounded-md bg-zinc-900 text-white flex flex-col items-center justify-center cursor-pointer relative group">
                    <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5 text-white/90" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                        <polygon points="6 3 20 12 6 21 6 3"/>
                    </svg>
                    <span class="text-[8px] font-mono uppercase tracking-wider mt-0.5 text-zinc-300">Video</span>
                </div>

                <!-- Empty Media Slot -->
                <div class="h-40 w-48  aspect-square rounded-md bg-zinc-50 border border-dashed border-zinc-200 flex items-center justify-center text-zinc-300">
                    <span class="text-[10px]">+</span>
                </div>
            </div>
        </div>

    </div>
</section>