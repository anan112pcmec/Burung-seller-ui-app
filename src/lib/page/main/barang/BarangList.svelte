<script lang="ts">
  let searchQuery = $state("");
  let selectedCategory = $state("all");
  let sortBy = $state("newest");

  const ulang = 6;
</script>

{#snippet CardBarang(index: number)}
  <div class="h-[14rem] w-[23.rem] border border-zinc-200 hover:border-zinc-400 rounded-lg bg-white overflow-hidden shadow-sm transition-all duration-150 flex flex-col group">
    <div class="grid grid-cols-[34%_66%] h-full">
      
      <!-- SISI KIRI (34%) -->
      <div class="grid grid-rows-2 border-r border-zinc-100 bg-zinc-50/50">
        <!-- Foto Barang Placeholder -->
        <div class="relative flex items-center justify-center border-b border-zinc-100 overflow-hidden bg-zinc-100/60">
          <svg xmlns="http://www.w3.org/2000/svg" class="w-7 h-7 text-zinc-400 stroke-1" viewBox="0 0 24 24" fill="none" stroke="currentColor">
            <rect width="18" height="18" x="3" y="3" rx="2" ry="2"/>
            <circle cx="9" cy="9" r="2"/>
            <path d="m21 15-3.086-3.086a2 2 0 0 0-2.828 0L6 21"/>
          </svg>
          <span class="absolute top-1.5 left-1.5 px-1.5 py-0.5 bg-zinc-900/80 backdrop-blur-xs text-white text-[9px] font-mono rounded">
            #{index + 1}
          </span>
        </div>

        <!-- Visual Mini Donut Chart Data Kategori -->
        <div class="p-2 flex flex-col items-center justify-center gap-1 bg-white">
          <div class="relative w-10 h-10">
            <svg class="w-full h-full -rotate-90" viewBox="0 0 36 36">
              <circle cx="18" cy="18" r="15.915" fill="none" class="stroke-zinc-100" stroke-width="3.5" />
              <circle cx="18" cy="18" r="15.915" fill="none" class="stroke-zinc-900" stroke-width="3.5" stroke-dasharray="70 30" stroke-dashoffset="0" />
              <circle cx="18" cy="18" r="15.915" fill="none" class="stroke-slate-400" stroke-width="3.5" stroke-dasharray="20 80" stroke-dashoffset="-70" />
            </svg>
            <span class="absolute inset-0 flex items-center justify-center text-[9px] font-bold font-mono text-zinc-700">70%</span>
          </div>
          <span class="text-[9px] text-zinc-400 font-medium tracking-tight truncate w-full text-center">Top Sold Category</span>
        </div>
      </div>

      <!-- SISI KANAN (66%) -->
      <div class="grid grid-rows-2 p-3 bg-white">
        <!-- Info Detail Barang -->
        <div class="flex flex-col justify-between border-b border-zinc-100 pb-2">
          <div>
            <div class="flex items-center justify-between gap-1 mb-1">
              <span class="px-1.5 py-0.5 bg-zinc-100 text-zinc-700 text-[9px] font-medium rounded uppercase tracking-wider">
                Elektronik
              </span>
              <span class="text-[11px] font-bold text-zinc-900 font-mono">
                Rp 250.000
              </span>
            </div>
            <h3 class="text-xs font-semibold text-zinc-800 leading-tight truncate">
              Kabel Data Type-C Fast Charge
            </h3>
            <p class="text-[10px] text-zinc-400 line-clamp-2 mt-0.5 leading-tight">
              Kabel premium braided nylon panjang 1 meter output 65W.
            </p>
          </div>
        </div>

        <!-- Engagement Info -->
        <div class="pt-2 flex flex-col justify-between">
          <span class="text-[9px] font-semibold uppercase tracking-wider text-zinc-400">Metrik Aktivitas</span>
          <div class="grid grid-cols-2 gap-y-1.5 gap-x-2 text-[10px] text-zinc-600 font-mono">
            <!-- Transaksi -->
            <div class="flex items-center gap-1">
              <svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-zinc-500" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M6 2 3 6v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2V6l-3-4Z"/><path d="M3 6h18"/><path d="M16 10a4 4 0 0 1-8 0"/></svg>
              <span>1.2k trx</span>
            </div>
            <!-- Likes -->
            <div class="flex items-center gap-1">
              <svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-zinc-500" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M19 14c1.49-1.46 3-3.21 3-5.5A5.5 5.5 0 0 0 16.5 3c-1.76 0-3 .5-4.5 2-1.5-1.5-2.74-2-4.5-2A5.5 5.5 0 0 0 2 8.5c0 2.3 1.5 4.05 3 5.5l7 7Z"/></svg>
              <span>342</span>
            </div>
            <!-- Comments -->
            <div class="flex items-center gap-1">
              <svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-zinc-500" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M7.9 20A9 9 0 1 0 4 16.1L2 22Z"/></svg>
              <span>89</span>
            </div>
            <!-- Saves/Archive -->
            <div class="flex items-center gap-1">
              <svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-zinc-500" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="m19 21-7-4-7 4V5a2 2 0 0 1 2-2h10a2 2 0 0 1 2 2v16z"/></svg>
              <span>154</span>
            </div>
          </div>
        </div>
      </div>

    </div>
  </div>
{/snippet}

<section id="barang-list" class="p-6 h-screen w-full border-t border-zinc-200 grid grid-rows-[auto_1fr] gap-6 bg-white text-zinc-800 font-sans">
    <div>		
        <h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
            Detail Informasi &amp; Barang
        </h1>
    </div>

  <!-- Header: Search, Filter, & Sorting -->
  <div class="flex flex-col sm:flex-row items-stretch sm:items-center justify-between gap-3 w-full border-b border-zinc-200 pb-4">
    
    <!-- Searchbar -->
    <div class="relative flex-1 max-w-xs">
      <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4 absolute left-3 top-1/2 -translate-y-1/2 text-zinc-400" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.75">
        <circle cx="11" cy="11" r="8"/>
        <path d="m21 21-4.3-4.3"/>
      </svg>
      <input 
        type="text" 
        bind:value={searchQuery}
        placeholder="Cari nama barang..." 
        class="w-full bg-white border border-zinc-300 rounded-md pl-9 pr-3 py-1.5 text-xs text-zinc-800 placeholder-zinc-400 focus:outline-none focus:border-zinc-800 transition-colors shadow-2xs"
      />
    </div>

    <!-- Controls Group -->
    <div class="flex items-center gap-2">
      <!-- Filter Kategori -->
      <div class="relative">
        <select 
          bind:value={selectedCategory}
          class="appearance-none bg-white border border-zinc-300 rounded-md pl-3 pr-8 py-1.5 text-xs text-zinc-700 focus:outline-none focus:border-zinc-800 cursor-pointer transition-colors shadow-2xs"
        >
          <option value="all">Semua Kategori</option>
          <option value="elektronik">Elektronik</option>
          <option value="pakaian">Pakaian</option>
          <option value="makanan">Makanan & Minuman</option>
        </select>
        <svg xmlns="http://www.w3.org/2000/svg" class="w-3.5 h-3.5 absolute right-2.5 top-1/2 -translate-y-1/2 text-zinc-400 pointer-events-none" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.75">
          <path d="m6 9 6 6 6-6"/>
        </svg>
      </div>

      <!-- Sorting -->
      <div class="relative">
        <select 
          bind:value={sortBy}
          class="appearance-none bg-white border border-zinc-300 rounded-md pl-3 pr-8 py-1.5 text-xs text-zinc-700 focus:outline-none focus:border-zinc-800 cursor-pointer transition-colors shadow-2xs"
        >
          <option value="newest">Urutkan: Terbaru</option>
          <option value="price-low">Harga: Low - High</option>
          <option value="price-high">Harga: High - Low</option>
          <option value="stock">Stok Terbanyak</option>
        </select>
        <svg xmlns="http://www.w3.org/2000/svg" class="w-3.5 h-3.5 absolute right-2.5 top-1/2 -translate-y-1/2 text-zinc-400 pointer-events-none" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.75">
          <path d="m6 9 6 6 6-6"/>
        </svg>
      </div>
    </div>

  </div>

  <!-- Product Grid / Container -->
  <div class="flex flex-wrap gap-4 items-start w-full overflow-y-auto scrollbar-none pr-1">
    {#each Array(ulang) as _, i (i)}
      {@render CardBarang(i)}
    {/each}
  </div>

</section>