<script lang="ts"></script>

<section id="barang-induk" class="p-6  mx-auto font-sans bg-white text-zinc-800 h-screen">
    <div class="border-b border-zinc-200 pb-3 mb-6">
        <h1 class="text-2xl font-bold uppercase tracking-tight text-zinc-900 leading-none">
            Barang Induk
        </h1>
    </div>

    <div class="grid grid-cols-1 md:grid-cols-[20%_80%] gap-6 items-start">
        
        <!-- SISI KIRI: MEDIA (GAMBAR & VIDEO) -->
        <div class="space-y-4">
            <!-- Box Gambar -->
            <div>
                <span class="block text-[10px] font-bold uppercase tracking-wider text-zinc-600 mb-1">
                    Foto Produk <span class="text-rose-600">*</span>
                </span>
                <label class="w-full aspect-square border-2 border-dashed border-zinc-300 hover:border-zinc-400 rounded-lg bg-zinc-50/60 flex flex-col items-center justify-center p-3 cursor-pointer transition-colors group">
                    <div class="w-8 h-8 rounded-full bg-zinc-200 group-hover:bg-zinc-900 group-hover:text-white text-zinc-600 flex items-center justify-center transition-colors mb-2">
                        <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                            <rect width="18" height="18" x="3" y="3" rx="2" ry="2"/>
                            <circle cx="9" cy="9" r="2"/>
                            <path d="m21 15-3.086-3.086a2 2 0 0 0-2.828 0L6 21"/>
                        </svg>
                    </div>
                    <span class="text-xs font-semibold text-zinc-700">Upload Gambar</span>
                    <span class="text-[9px] text-zinc-400 mt-0.5 text-center">PNG, JPG, WEBP</span>
                    <input type="file" accept="image/*" class="hidden" />
                </label>
            </div>

            <!-- Box Video -->
            <div>
                <span class="block text-[10px]  font-bold uppercase tracking-wider text-zinc-600 mb-1">
                    Video Produk
                </span>
                <label class="w-full h-[11rem] border-2 border-dashed border-zinc-300 hover:border-zinc-400 rounded-lg bg-zinc-50/60 flex flex-col items-center justify-center p-3 cursor-pointer transition-colors group">
                    <div class="w-7 h-7 rounded-full bg-zinc-200 group-hover:bg-zinc-900 group-hover:text-white text-zinc-600 flex items-center justify-center transition-colors mb-1.5">
                        <svg xmlns="http://www.w3.org/2000/svg" class="w-3.5 h-3.5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                            <path d="m22 8-6 4 6 4V8Z"/>
                            <rect width="14" height="12" x="2" y="6" rx="2" ry="2"/>
                        </svg>
                    </div>
                    <span class="text-xs font-semibold text-zinc-700">Upload Video</span>
                    <span class="text-[9px] text-zinc-400 mt-0.5 text-center">MP4, WEBM</span>
                    <input type="file" accept="video/*" class="hidden" />
                </label>
            </div>
        </div>

        <!-- SISI KANAN: FORM INPUTS -->
        <div class="space-y-2 h-full">
            <!-- Nama Barang -->
            <div>
                <label for="nama_barang" class="block text-xs font-bold uppercase tracking-wider text-zinc-700 mb-1">
                    Nama Barang <span class="text-rose-600">*</span>
                </label>
                <input
                    id="nama_barang"
                    type="text"
                    maxlength="200"
                    required
                    placeholder="Contoh: Kemeja Oversize Linen Premium"
                    class="w-full bg-white border border-zinc-300 rounded-md px-3 py-2 text-xs text-zinc-800 placeholder-zinc-400 focus:outline-none focus:border-zinc-900 shadow-2xs transition-colors"
                />
            </div>

            <!-- Jenis Barang -->
            <div>
                <label for="jenis_barang" class="block text-xs font-bold uppercase tracking-wider text-zinc-700 mb-1">
                    Jenis Barang
                </label>
                <select
                    id="jenis_barang"
                    class="w-full bg-white border border-zinc-300 rounded-md px-3 py-2 text-xs text-zinc-800 focus:outline-none focus:border-zinc-900 shadow-2xs transition-colors cursor-pointer"
                >
                    <option value="Semua Barang">Semua Barang</option>
                    <option value="Pakaian">Pakaian</option>
                    <option value="Elektronik">Elektronik</option>
                    <option value="Aksesoris">Aksesoris</option>
                </select>
            </div>

            <!-- Deskripsi Barang -->
            <div>
                <label for="deskripsi" class="block text-xs font-bold uppercase tracking-wider text-zinc-700 mb-1">
                    Deskripsi Barang
                </label>
                <textarea
                    id="deskripsi"
                    rows="5"
                    placeholder="..."
                    class="w-full bg-white border border-zinc-300 rounded-md py-0 text-xs text-zinc-800 placeholder-zinc-400 focus:outline-none focus:border-zinc-900 shadow-2xs transition-colors resize-none"
                ></textarea>
            </div>
            
            <div class="w-full h-[9rem] py-2 grid grid-cols-2 gap-3">
        <!-- Elemen Tambah Alamat Gudang -->
        <button
            type="button"
            class="w-full h-full border border-dashed border-zinc-300 hover:border-zinc-400 rounded-lg bg-zinc-50/60 hover:bg-zinc-100/80 p-3 flex flex-col items-center justify-center gap-1.5 transition-colors duration-150 cursor-pointer group"
        >
            <div class="w-7 h-7 rounded-full bg-zinc-200 group-hover:bg-zinc-900 group-hover:text-white text-zinc-700 flex items-center justify-center transition-colors">
            <svg xmlns="http://www.w3.org/2000/svg" class="w-3.5 h-3.5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
                <path d="M20 10c0 6-8 12-8 12s-8-6-8-12a8 8 0 0 1 16 0Z"/>
                <circle cx="12" cy="10" r="3"/>
            </svg>
            </div>
            <div class="text-center">
            <span class="block text-xs font-bold text-zinc-800 leading-none">Tambah Alamat Gudang</span>
            <span class="block text-[10px] text-zinc-400 mt-1 leading-none">Lokasi pengiriman baru</span>
            </div>
        </button>

        <!-- Elemen Tambah Rekening -->
        <button
            type="button"
            class="w-full h-full border border-dashed border-zinc-300 hover:border-zinc-400 rounded-lg bg-zinc-50/60 hover:bg-zinc-100/80 p-3 flex flex-col items-center justify-center gap-1.5 transition-colors duration-150 cursor-pointer group"
        >
            <div class="w-7 h-7 rounded-full bg-zinc-200 group-hover:bg-zinc-900 group-hover:text-white text-zinc-700 flex items-center justify-center transition-colors">
            <svg xmlns="http://www.w3.org/2000/svg" class="w-3.5 h-3.5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
                <rect width="20" height="14" x="2" y="5" rx="2"/>
                <line x1="2" x2="22" y1="10" y2="10"/>
            </svg>
            </div>
            <div class="text-center">
            <span class="block text-xs font-bold text-zinc-800 leading-none">Tambah Rekening Bank</span>
            <span class="block text-[10px] text-zinc-400 mt-1 leading-none">Rekening pencairan dana</span>
            </div>
        </button>
        </div>
        <div class="flex items-start gap-2.5 p-3 rounded-lg bg-zinc-50 border border-zinc-200 text-zinc-600 my-2">
  <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4 text-zinc-500 shrink-0 mt-0.5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
    <circle cx="12" cy="12" r="10"/>
    <path d="M12 16v-4"/>
    <path d="M12 8h.01"/>
  </svg>
  <p class="text-[11px] leading-relaxed text-zinc-500">
    <span class="font-bold text-zinc-800">Catatan Opsional:</span> Pengisian alamat gudang dan rekening di atas bersifat tidak wajib. Jika diisi, seluruh kategori akan otomatis menggunakan data tersebut. Jika dikosongkan, alamat dan rekening akan mengikuti pengaturan masing-masing kategori.
  </p>
</div>

        
        </div>

    </div>
</section>