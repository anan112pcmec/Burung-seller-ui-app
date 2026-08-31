<script lang="ts">
    const ulang = 10;
</script>

{#snippet Kategori()}
    <div class="w-64 shrink-0 border border-zinc-200 hover:border-zinc-400 rounded-lg bg-white overflow-hidden shadow-sm transition-colors duration-150 flex flex-col justify-between p-3.5 gap-3">
        <!-- Frame Foto Produk (Potret Atas) -->
        <div class="w-full h-40 rounded-md bg-zinc-100 border border-zinc-200 overflow-hidden shrink-0 flex items-center justify-center relative">
            <svg xmlns="http://www.w3.org/2000/svg" class="w-8 h-8 text-zinc-400" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5">
                <rect width="18" height="18" x="3" y="3" rx="2" ry="2"/>
                <circle cx="9" cy="9" r="2"/>
                <path d="m21 15-3.086-3.086a2 2 0 0 0-2.828 0L6 21"/>
            </svg>
            
            <!-- Badge Overlay -->
            <span class="absolute top-2 left-2 bg-zinc-900/80 text-white text-[8px] font-mono px-1.5 py-0.5 rounded uppercase tracking-wider">
                Original
            </span>
            <span class="absolute top-2 right-2 bg-teal-50 text-teal-700 text-[9px] font-mono font-semibold px-1.5 py-0.5 rounded border border-teal-200">
                Stok: 45
            </span>
        </div>

        <!-- Info Main -->
        <div class="space-y-1">
            <span class="text-[9px] font-mono text-zinc-400 uppercase tracking-wider block">SKU: KTG-8821</span>
            <h4 class="text-xs font-bold text-zinc-800 truncate leading-tight">
                Variasi Hitam XL
            </h4>
            <p class="text-[10px] text-zinc-400 line-clamp-2 leading-tight">
                Kategori variasi ukuran besar dengan material linen murni.
            </p>
        </div>

        <!-- Harga -->
        <div class="text-sm font-bold font-mono text-zinc-900">
            Rp 149.000 <span class="text-[10px] text-zinc-400 font-normal line-through ml-1">Rp 175.000</span>
        </div>

        <!-- Spesifikasi (Grid 3 Kolom) -->
        <div class="grid grid-cols-3 gap-1 bg-zinc-50 p-2 rounded border border-zinc-100 text-[9px] font-mono text-zinc-600">
            <div>
                <span class="block text-zinc-400 text-[8px] uppercase">Warna</span>
                <span class="font-semibold text-zinc-700 truncate block">Hitam</span>
            </div>
            <div>
                <span class="block text-zinc-400 text-[8px] uppercase">Berat</span>
                <span class="font-semibold text-zinc-700 truncate block">250g</span>
            </div>
            <div>
                <span class="block text-zinc-400 text-[8px] uppercase">Dimensi</span>
                <span class="font-semibold text-zinc-700 truncate block">30x20</span>
            </div>
        </div>

        <!-- Relasi Gudang & Rekening -->
        <div class="flex flex-col gap-1 text-[9px] font-mono text-zinc-400 border-t border-zinc-100 pt-2">
            <span class="flex items-center gap-1.5 truncate" title="Gudang Utama Jakarta">
                <svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-zinc-400 shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                    <path d="M20 10c0 6-8 12-8 12s-8-6-8-12a8 8 0 0 1 16 0Z"/>
                    <circle cx="12" cy="10" r="3"/>
                </svg>
                Gudang Utama
            </span>
            <span class="flex items-center gap-1.5 truncate" title="BCA - 8829101">
                <svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-zinc-400 shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                    <rect width="20" height="14" x="2" y="5" rx="2"/>
                    <line x1="2" x2="22" y1="10" y2="10"/>
                </svg>
                BCA • 8829
            </span>
        </div>
    </div>
{/snippet}

<section id="kategori-barang" class="p-5">
<div class="border-b border-zinc-200 pb-3 mb-6 flex items-center justify-between w-full">
    <h2 class="text-2xl font-bold uppercase tracking-tight text-zinc-900 leading-none">
        Kategori Barang
    </h2> 
    
    <div class="flex items-center gap-2 text-zinc-800 font-mono font-bold">
        <span class="text-2xl leading-none">4</span>
        <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6 stroke-zinc-700" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <rect width="7" height="7" x="3" y="3" rx="1"/>
            <rect width="7" height="7" x="14" y="3" rx="1"/>
            <rect width="7" height="7" x="14" y="14" rx="1"/>
            <rect width="7" height="7" x="3" y="14" rx="1"/>
        </svg>
    </div>
</div>
<div class="flex flex-nowrap gap-4 items-stretch overflow-x-auto bg-white font-sans scrollbar-none">
    {#each Array(ulang) as _, i}
        {#if i == 0}
            <!-- Card Tambah Kategori -->
            <button
                type="button"
                class="w-64 border-2 border-dashed border-zinc-300 hover:border-zinc-400 rounded-lg bg-zinc-50/60 hover:bg-zinc-100/80 p-5 flex flex-col items-center justify-center gap-2 transition-colors duration-150 cursor-pointer group shrink-0 min-h-[24rem]"
            >
                <div class="w-10 h-10 rounded-full bg-zinc-200 group-hover:bg-zinc-900 group-hover:text-white text-zinc-700 flex items-center justify-center transition-colors">
                    <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
                        <path d="M5 12h14"/><path d="M12 5v14"/>
                    </svg>
                </div>
                <div class="text-center">
                    <span class="block text-xs font-bold text-zinc-800 leading-tight">Tambah Kategori</span>
                    <span class="block text-[10px] text-zinc-400 mt-1">Buat variasi, stok, SKU & harga baru</span>
                </div>
            </button>
        {/if}

        {@render Kategori()}
    {/each}
</div>
</section>