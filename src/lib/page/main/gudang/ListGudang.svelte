<script lang="ts">
  interface GudangItem {
    id_alamat_gudang: number;
    panggilan_alamat_gudang: string;
    nomor_telfon_alamat_gudang: string;
    nama_alamat_gudang: string;
    provinsi_alamat_gudang: string;
    kota_alamat_gudang: string;
    kode_pos_alamat_gudang: string;
    deskripsi_alamat_gudang: string;
    longitude_alamat_gudang: number;
    latitude_alamat_gudang: number;
    created_at: string;
    jumlah_barang: number;
    jumlah_transaksi: number;
  }

  let searchQuery = $state("");
  let selectedProvinsi = $state("all");
  let sortBy = $state("newest");

  const gudangList: GudangItem[] = [
    {
      id_alamat_gudang: 1,
      panggilan_alamat_gudang: "Gudang Pusat",
      nomor_telfon_alamat_gudang: "022-7301122",
      nama_alamat_gudang: "Jl. Industri Raya No. 12",
      provinsi_alamat_gudang: "Jawa Barat",
      kota_alamat_gudang: "Bandung",
      kode_pos_alamat_gudang: "40234",
      deskripsi_alamat_gudang: "Gudang utama, tempat penyimpanan mayoritas stok barang.",
      longitude_alamat_gudang: 107.6191,
      latitude_alamat_gudang: -6.9175,
      created_at: "2025-11-02T08:00:00Z",
      jumlah_barang: 482,
      jumlah_transaksi: 1204
    },
    {
      id_alamat_gudang: 2,
      panggilan_alamat_gudang: "Gudang Cabang Timur",
      nomor_telfon_alamat_gudang: "031-8451190",
      nama_alamat_gudang: "Jl. Rungkut Industri No. 5",
      provinsi_alamat_gudang: "Jawa Timur",
      kota_alamat_gudang: "Surabaya",
      kode_pos_alamat_gudang: "60293",
      deskripsi_alamat_gudang: "Melayani pengiriman area Jawa Timur & Bali.",
      longitude_alamat_gudang: 112.7521,
      latitude_alamat_gudang: -7.3305,
      created_at: "2026-02-14T10:00:00Z",
      jumlah_barang: 316,
      jumlah_transaksi: 587
    },
    {
      id_alamat_gudang: 3,
      panggilan_alamat_gudang: "Gudang Titipan Reseller",
      nomor_telfon_alamat_gudang: "0274-556677",
      nama_alamat_gudang: "Jl. Kaliurang KM 9",
      provinsi_alamat_gudang: "DI Yogyakarta",
      kota_alamat_gudang: "Sleman",
      kode_pos_alamat_gudang: "55581",
      deskripsi_alamat_gudang: "Titip stok ke mitra reseller lokal, belum aktif kirim langsung.",
      longitude_alamat_gudang: 110.4083,
      latitude_alamat_gudang: -7.6979,
      created_at: "2026-05-20T09:30:00Z",
      jumlah_barang: 94,
      jumlah_transaksi: 0
    },
    {
      id_alamat_gudang: 4,
      panggilan_alamat_gudang: "Gudang Musiman Lebaran",
      nomor_telfon_alamat_gudang: "021-88991122",
      nama_alamat_gudang: "Jl. Soekarno Hatta No. 88",
      provinsi_alamat_gudang: "Jawa Barat",
      kota_alamat_gudang: "Bekasi",
      kode_pos_alamat_gudang: "17141",
      deskripsi_alamat_gudang: "Disewa musiman untuk lonjakan stok jelang hari raya.",
      longitude_alamat_gudang: 107.0137,
      latitude_alamat_gudang: -6.2383,
      created_at: "2026-08-01T07:00:00Z",
      jumlah_barang: 58,
      jumlah_transaksi: 41
    },
    {
      id_alamat_gudang: 5,
      panggilan_alamat_gudang: "Gudang Lama",
      nomor_telfon_alamat_gudang: "022-4201090",
      nama_alamat_gudang: "Jl. Pahlawan No. 3",
      provinsi_alamat_gudang: "Jawa Barat",
      kota_alamat_gudang: "Bandung",
      kode_pos_alamat_gudang: "40122",
      deskripsi_alamat_gudang: "Gudang lama, sudah jarang dipakai sejak Gudang Pusat aktif.",
      longitude_alamat_gudang: 107.6098,
      latitude_alamat_gudang: -6.9034,
      created_at: "2025-06-10T11:00:00Z",
      jumlah_barang: 21,
      jumlah_transaksi: 0
    }
  ];

  const daftarProvinsi = [...new Set(gudangList.map((g) => g.provinsi_alamat_gudang))];

  let gudangTertampil = $derived.by(() => {
    let hasil = gudangList.filter((g) => {
      const cocokCari =
        searchQuery.trim() === "" ||
        g.panggilan_alamat_gudang.toLowerCase().includes(searchQuery.toLowerCase()) ||
        g.kota_alamat_gudang.toLowerCase().includes(searchQuery.toLowerCase());
      const cocokProvinsi = selectedProvinsi === "all" || g.provinsi_alamat_gudang === selectedProvinsi;
      return cocokCari && cocokProvinsi;
    });

    if (sortBy === "newest") hasil = hasil.sort((a, b) => +new Date(b.created_at) - +new Date(a.created_at));
    if (sortBy === "barang-terbanyak") hasil = hasil.sort((a, b) => b.jumlah_barang - a.jumlah_barang);
    if (sortBy === "transaksi-terbanyak") hasil = hasil.sort((a, b) => b.jumlah_transaksi - a.jumlah_transaksi);
    if (sortBy === "nama") hasil = hasil.sort((a, b) => a.panggilan_alamat_gudang.localeCompare(b.panggilan_alamat_gudang));

    return hasil;
  });

  function formatTanggal(iso: string): string {
    return new Date(iso).toLocaleDateString("id-ID", { day: "2-digit", month: "short", year: "numeric" });
  }

  function mapEmbedUrl(lat: number, lon: number): string {
    const delta = 0.006;
    const bbox = `${lon - delta},${lat - delta},${lon + delta},${lat + delta}`;
    return `https://www.openstreetmap.org/export/embed.html?bbox=${bbox}&marker=${lat},${lon}&layer=mapnik`;
  }

  function mapsLink(lat: number, lon: number): string {
    return `https://www.google.com/maps?q=${lat},${lon}`;
  }
</script>

{#snippet CardGudang(gudang: GudangItem)}
  {@const tanpaTransaksi = gudang.jumlah_transaksi === 0}
  <div class="w-[27rem] h-[14rem] grid grid-cols-[36%_64%] border border-zinc-200 hover:border-zinc-400 rounded-lg bg-white shadow-2xs transition-all duration-150 overflow-hidden shrink-0 p-4">
    
    <!-- PETA (SISI KIRI) -->
    <div class="relative w-full h-full border-r border-zinc-200 bg-zinc-50 overflow-hidden">
      <iframe
        title="Peta {gudang.panggilan_alamat_gudang}"
        src={mapEmbedUrl(gudang.latitude_alamat_gudang, gudang.longitude_alamat_gudang)}
        class="w-full h-full grayscale-[20%] pointer-events-none"
        loading="lazy"
      ></iframe>
      
      <a
        href={mapsLink(gudang.latitude_alamat_gudang, gudang.longitude_alamat_gudang)}
        target="_blank"
        rel="noopener noreferrer"
        class="absolute bottom-2 right-2 bg-white/95 border border-zinc-200 rounded px-1.5 py-0.5 text-[8px] font-mono uppercase tracking-wider text-zinc-700 hover:bg-zinc-100 shadow-2xs transition-colors"
      >
        Buka Peta
      </a>

      {#if tanpaTransaksi}
        <span class="absolute top-2 left-2 bg-rose-50/95 border border-rose-200 rounded px-1.5 py-0.5 text-[8px] font-mono uppercase tracking-wider text-rose-600 font-semibold shadow-2xs">
          Pasif
        </span>
      {/if}
    </div>

    <!-- DETAIL GUDANG (SISI KANAN) -->
    <div class="p-3.5 flex flex-col justify-between bg-white h-full">
      <div class="flex flex-col gap-1">
        <!-- Header Card -->
        <div class="flex items-start justify-between gap-1">
          <h3 class="text-xs font-bold text-zinc-900 leading-tight truncate">
            {gudang.panggilan_alamat_gudang}
          </h3>
          <span class="text-[9px] text-zinc-400 font-mono shrink-0">
            {formatTanggal(gudang.created_at)}
          </span>
        </div>

        <!-- Alamat Lengkap -->
        <p class="text-[10px] text-zinc-500 leading-tight truncate">
          {gudang.nama_alamat_gudang}, {gudang.kota_alamat_gudang}
        </p>

        <!-- Deskripsi Singkat -->
        <p class="text-[10px] text-zinc-400 line-clamp-2 leading-snug mt-0.5">
          {gudang.deskripsi_alamat_gudang}
        </p>
      </div>

      <!-- Footer Card (Kontak & Stats) -->
      <div class="flex flex-col gap-2 pt-2 border-t border-zinc-100">
        <!-- Telepon -->
        <div class="flex items-center gap-1 text-[10px] text-zinc-500 font-mono">
          <svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-zinc-400 stroke-2" viewBox="0 0 24 24" fill="none" stroke="currentColor">
            <path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z"/>
          </svg>
          <span>{gudang.nomor_telfon_alamat_gudang}</span>
        </div>

        <!-- Metrik Barang & Transaksi -->
        <div class="grid grid-cols-2 gap-2 bg-zinc-50 p-1.5 rounded border border-zinc-100">
          <div class="flex flex-col">
            <span class="text-[8px] font-bold tracking-wider text-zinc-400 uppercase font-mono">Barang</span>
            <span class="text-xs font-bold font-mono tracking-tight text-zinc-900">{gudang.jumlah_barang}</span>
          </div>
          <div class="flex flex-col">
            <span class="text-[8px] font-bold tracking-wider text-zinc-400 uppercase font-mono">Transaksi</span>
            <span class="text-xs font-bold font-mono tracking-tight {tanpaTransaksi ? 'text-zinc-400' : 'text-zinc-900'}">
              {gudang.jumlah_transaksi}
            </span>
          </div>
        </div>
      </div>
    </div>

  </div>
{/snippet}

<section id="list-gudang" class="p-8 h-screen w-full border-t border-zinc-200 grid grid-rows-[auto_auto_1fr] gap-4 bg-white text-zinc-800 font-sans">
  
  <!-- Title Header -->
  <div class="flex items-center justify-between">
    <div>
      <h1 class="text-lg font-bold uppercase tracking-tight text-zinc-900">
        Alamat Gudang
      </h1>
      <p class="text-[10px] text-zinc-400">Kelola lokasi gudang dan pemetaan titik pengiriman</p>
    </div>
  </div>

  <!-- Filter & Controls -->
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
        placeholder="Cari nama gudang / kota..."
        class="w-full bg-white border border-zinc-300 rounded-md pl-9 pr-3 py-1.5 text-xs text-zinc-800 placeholder-zinc-400 focus:outline-none focus:border-zinc-800 transition-colors shadow-2xs"
      />
    </div>

    <!-- Dropdowns -->
    <div class="flex items-center gap-2">
      <!-- Filter Provinsi -->
      <div class="relative">
        <select
          bind:value={selectedProvinsi}
          class="appearance-none bg-white border border-zinc-300 rounded-md pl-3 pr-8 py-1.5 text-xs text-zinc-700 focus:outline-none focus:border-zinc-800 cursor-pointer transition-colors shadow-2xs"
        >
          <option value="all">Semua Provinsi</option>
          {#each daftarProvinsi as prov}
            <option value={prov}>{prov}</option>
          {/each}
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
          <option value="barang-terbanyak">Barang Terbanyak</option>
          <option value="transaksi-terbanyak">Transaksi Terbanyak</option>
          <option value="nama">Nama A-Z</option>
        </select>
        <svg xmlns="http://www.w3.org/2000/svg" class="w-3.5 h-3.5 absolute right-2.5 top-1/2 -translate-y-1/2 text-zinc-400 pointer-events-none" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.75">
          <path d="m6 9 6 6 6-6"/>
        </svg>
      </div>
    </div>
  </div>

  <!-- Gudang Grid Container -->
  <div class="flex flex-wrap gap-4 items-start w-full overflow-y-auto pr-1 pb-4">
    {#if gudangTertampil.length === 0}
      <p class="text-xs text-zinc-400 font-mono py-8 w-full text-center">Tidak ada gudang yang cocok dengan pencarian.</p>
    {:else}
      {#each gudangTertampil as gudang (gudang.id_alamat_gudang)}
        {@render CardGudang(gudang)}
      {/each}
    {/if}
  </div>

</section>