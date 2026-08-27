<script lang="ts">
  interface Etalase {
    id_etalase: number;
    id_seller_etalase: number;
    nama_etalase: string;
    deskripsi_etalase: string;
    jumlah_barang: number;
    created_at: string;
    updated_at: string;
  }

  let searchQuery = $state("");
  let selectedStatus = $state("all");
  let sortBy = $state("newest");

  // ///////////////////////////////////////////////////////////////////////
  // Hardcode dulu — ganti dengan hasil fetch list etalase milik seller
  // ///////////////////////////////////////////////////////////////////////
  const etalaseList: Etalase[] = [
    {
      id_etalase: 1,
      id_seller_etalase: 501,
      nama_etalase: "Etalase Utama",
      deskripsi_etalase: "Kumpulan produk andalan dan best seller toko.",
      jumlah_barang: 42,
      created_at: "2026-02-14T08:00:00Z",
      updated_at: "2026-08-20T10:32:00Z"
    },
    {
      id_etalase: 2,
      id_seller_etalase: 501,
      nama_etalase: "Koleksi Baju Pria",
      deskripsi_etalase: "Kemeja, kaos, dan celana khusus kategori pria dewasa.",
      jumlah_barang: 27,
      created_at: "2026-03-02T09:15:00Z",
      updated_at: "2026-08-18T14:02:00Z"
    },
    {
      id_etalase: 3,
      id_seller_etalase: 501,
      nama_etalase: "Aksesoris & Tas",
      deskripsi_etalase: "Tas, dompet, dan aksesoris pelengkap gaya harian.",
      jumlah_barang: 15,
      created_at: "2026-04-10T11:40:00Z",
      updated_at: "2026-08-25T09:11:00Z"
    },
    {
      id_etalase: 4,
      id_seller_etalase: 501,
      nama_etalase: "Promo Spesial",
      deskripsi_etalase: "Barang diskon dan bundling harga terbatas.",
      jumlah_barang: 8,
      created_at: "2026-06-01T07:20:00Z",
      updated_at: "2026-08-26T16:45:00Z"
    },
    {
      id_etalase: 5,
      id_seller_etalase: 501,
      nama_etalase: "Sepatu & Sandal",
      deskripsi_etalase: "Alas kaki pria dan wanita berbagai ukuran.",
      jumlah_barang: 19,
      created_at: "2026-05-18T13:00:00Z",
      updated_at: "2026-08-11T08:30:00Z"
    },
    {
      id_etalase: 6,
      id_seller_etalase: 501,
      nama_etalase: "Barang Preloved",
      deskripsi_etalase: "Etalase baru, belum ada barang ditambahkan.",
      jumlah_barang: 0,
      created_at: "2026-08-22T10:00:00Z",
      updated_at: "2026-08-22T10:00:00Z"
    }
  ];

  const maxJumlahBarang = Math.max(...etalaseList.map((e) => e.jumlah_barang), 1);

  function formatTanggal(iso: string): string {
    return new Date(iso).toLocaleDateString("id-ID", { day: "2-digit", month: "short", year: "numeric" });
  }
</script>

{#snippet CardEtalase(etalase: Etalase, index: number)}
  {@const persenIsi = Math.round((etalase.jumlah_barang / maxJumlahBarang) * 100)}
  {@const aktif = etalase.jumlah_barang > 0}
  {@const banner = [
    'from-teal-700 via-teal-800 to-zinc-900',
    'from-zinc-700 via-zinc-800 to-slate-900',
    'from-slate-700 via-slate-800 to-teal-900',
    'from-zinc-800 via-teal-900 to-zinc-950'
  ][index % 4]}

  <div class="h-[15.5rem] w-[23rem] border border-zinc-200 hover:border-zinc-400 rounded-lg bg-white overflow-hidden shadow-sm transition-all duration-150 flex flex-col group">

    <!-- BANNER — identitas etalase, kayak header profil/channel -->
    <div class="relative h-[4.75rem] w-full shrink-0 overflow-hidden bg-gradient-to-br {banner}">
      <!-- pattern watermark tipis, motif ikon toko diulang -->
      <svg class="absolute inset-0 w-full h-full opacity-[0.08]" preserveAspectRatio="xMidYMid slice">
        <pattern id="etalase-pattern-{etalase.id_etalase}" width="34" height="34" patternUnits="userSpaceOnUse">
          <path d="M4 12h26v4a3 3 0 0 1-3 3 3 3 0 0 1-3-3 3 3 0 0 1-3 3 3 3 0 0 1-3-3 3 3 0 0 1-3 3h-2a3 3 0 0 1-3-3 3 3 0 0 1-3 3H8a3 3 0 0 1-3-3v-4Z"
            fill="none" stroke="white" stroke-width="1"/>
        </pattern>
        <rect width="100%" height="100%" fill="url(#etalase-pattern-{etalase.id_etalase})" />
      </svg>

      <!-- rank -->
      <span class="absolute top-2 left-2 px-1.5 py-0.5 bg-black/30 backdrop-blur-xs text-white text-[9px] font-mono rounded">
        #{index + 1}
      </span>

      <!-- status -->
      <span class="absolute top-2 right-2 px-1.5 py-0.5 {aktif ? 'bg-teal-500/20 text-teal-50' : 'bg-white/15 text-white/70'} backdrop-blur-xs text-[9px] font-medium rounded uppercase tracking-wider">
        {aktif ? 'Aktif' : 'Kosong'}
      </span>
    </div>

    <!-- KONTEN — avatar overlap + info -->
    <div class="relative px-4 pb-3.5 flex-1 flex flex-col min-h-0">
      <div class="-mt-6 mb-2 flex items-end justify-between">
        <div class="w-12 h-12 rounded-full bg-white border-[3px] border-white ring-1 ring-zinc-200 shadow-sm flex items-center justify-center shrink-0">
          <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5 text-zinc-400 stroke-1" viewBox="0 0 24 24" fill="none" stroke="currentColor">
            <path d="M2 7h20v3a2 2 0 0 1-2 2h-1a2 2 0 0 1-2-2 2 2 0 0 1-2 2h-1a2 2 0 0 1-2-2 2 2 0 0 1-2 2H9a2 2 0 0 1-2-2 2 2 0 0 1-2 2H4a2 2 0 0 1-2-2Z"/>
            <path d="M4 12v6a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-6"/>
            <path d="M9 21v-5a1 1 0 0 1 1-1h4a1 1 0 0 1 1 1v5"/>
          </svg>
        </div>
        <span class="text-[9px] text-zinc-400 font-mono mb-0.5">dibuat {formatTanggal(etalase.created_at)}</span>
      </div>

      <h3 class="text-xs font-semibold text-zinc-800 leading-tight truncate">
        {etalase.nama_etalase}
      </h3>
      <p class="text-[10px] text-zinc-400 line-clamp-2 mt-0.5 leading-tight">
        {etalase.deskripsi_etalase}
      </p>

      <!-- Isi Etalase — progress bar, bukan donut, biar gak kembar sama kartu barang -->
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
            {etalase.jumlah_barang} barang
          </span>
          <span class="flex items-center gap-1">
            <svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-zinc-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M12 8v4l3 3"/><circle cx="12" cy="12" r="9"/>
            </svg>
            diperbarui {formatTanggal(etalase.updated_at)}
          </span>
        </div>
      </div>
    </div>
  </div>
{/snippet}

<section id="etalase-list" class="p-6 h-screen w-full border-t border-zinc-200 grid grid-rows-[auto_1fr] gap-6 bg-white text-zinc-800 font-sans">
  <div>
    <h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
      Etalase Kamu /  12
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
        placeholder="Cari nama etalase..."
        class="w-full bg-white border border-zinc-300 rounded-md pl-9 pr-3 py-1.5 text-xs text-zinc-800 placeholder-zinc-400 focus:outline-none focus:border-zinc-800 transition-colors shadow-2xs"
      />
    </div>

   
    <!-- Controls Group -->
    <div class="flex items-center gap-2">
      <!-- Filter Status -->
      <div class="relative">
        <select
          bind:value={selectedStatus}
          class="appearance-none bg-white border border-zinc-300 rounded-md pl-3 pr-8 py-1.5 text-xs text-zinc-700 focus:outline-none focus:border-zinc-800 cursor-pointer transition-colors shadow-2xs"
        >
          <option value="all">Semua Status</option>
          <option value="aktif">Aktif (ada barang)</option>
          <option value="kosong">Kosong</option>
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
          <option value="oldest">Urutkan: Terlama</option>
          <option value="barang-most">Jumlah Barang: Terbanyak</option>
          <option value="barang-least">Jumlah Barang: Tersedikit</option>
          <option value="name">Nama A-Z</option>
        </select>
        <svg xmlns="http://www.w3.org/2000/svg" class="w-3.5 h-3.5 absolute right-2.5 top-1/2 -translate-y-1/2 text-zinc-400 pointer-events-none" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.75">
          <path d="m6 9 6 6 6-6"/>
        </svg>
      </div>
    </div>

  </div>

  <!-- Etalase Grid / Container -->
  <div class="flex flex-wrap gap-4 items-start w-full overflow-y-auto scrollbar-none pr-1">
    {#each etalaseList as etalase, i (etalase.id_etalase)}
      {@render CardEtalase(etalase, i)}
    {/each}
  </div>

</section>