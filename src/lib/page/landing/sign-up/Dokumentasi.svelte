<script lang="ts">
  interface subBabContent {
    subBab: string;
    penjelasan: string;
  }
  interface DokumentasiContent {
    bab: string;
    subBab: subBabContent[];
  }

  // Data Kebijakan & Ekosistem Seller "Burung"
  let docs: DokumentasiContent[] = $state([
    {
      bab: "1. Core & Hierarchy",
      subBab: [
        {
          subBab: "Multi-Tier Seller",
          penjelasan: "Jalur pertumbuhan berjenjang dari Seller Biasa, Distributor (verifikasi NIB/NPWP), hingga Brand Resmi (sertifikat merek). Memberikan posisi tawar dan kredibilitas lebih tinggi di platform."
        },
        {
          subBab: "Personal Branding Toko",
          penjelasan: "Pengelolaan identitas toko secara fleksibel melalui Punchline, Jam Operasional, Deskripsi Khusus, dan Badge Dedikasi untuk membangun reputasi independen."
        },
        {
          subBab: "Etalase Curated",
          penjelasan: "Pengelompokan koleksi produk khusus di luar kategori bawaan platform. Seller bebas membuat tema etalase untuk menaikkan nilai estetika toko."
        },
        {
          subBab: "Verifikasi Legalitas",
          penjelasan: "Prosedur otomatis penyerahan dan pemeriksaan dokumen resmi negara guna pembukaan fitur distributor dan akses fitur grosir skala besar."
        }
      ]
    },
    {
      bab: "2. Produk & Stok",
      subBab: [
        {
          subBab: "Sinkronisasi Stok Otomatis",
          penjelasan: "Sistem pembaruan inventori langsung saat transaksi terjadi, mencegah overbooking barang dan meminimalkan pembatalan pesanan."
        },
        {
          subBab: "Relasi Induk & Variasi",
          penjelasan: "Struktur manajemen barang berbasis induk (parent) dan anak variasi (SKU). Memudahkan ulasan dan rating terkumpul di satu halaman barang."
        },
        {
          subBab: "Social Engagement Layer",
          penjelasan: "Fitur interaksi dua arah pada halaman produk yang mendukung komentar, balasan seller, hingga mention antar pengguna untuk meningkatkan trust."
        },
        {
          subBab: "Manajemen Kategori",
          penjelasan: "Pengelompokan barang secara hirarkis yang presisi untuk mempermudah pencarian buyer serta optimalisasi SEO di luar ekosistem Burung."
        }
      ]
    },
    {
      bab: "3. Logistik & Kirim",
      subBab: [
        {
          subBab: "Flexible Order Control",
          penjelasan: "Kebebasan menentukan skema pemrosesan pesanan secara manual atau otomatis (IsAuto & AutoPengiriman). Operasional toko berjalan sesuai kapasitas seller."
        },
        {
          subBab: "Rating Kurir Dua Arah",
          penjelasan: "Sistem evaluasi transparan di mana seller memiliki hak menilai kinerja kurir pengirim (Merpati Express) demi menjaga kualitas pengiriman barang."
        },
        {
          subBab: "Multi-Gudang & Alamat",
          penjelasan: "Dukungan penentuan titik penjemputan barang dari beberapa alamat gudang terpisah untuk efisiensi ongkos kirim dan waktu kirim."
        },
        {
          subBab: "Tracking Real-time",
          penjelasan: "Integrasi sistem pelacakan otomatis yang memperbarui status armada ekspedisi dari penjemputan awal hingga serah terima di buyer."
        }
      ]
    },
    {
      bab: "4. Diskon & Promo",
      subBab: [
        {
          subBab: "Granular Diskon Engine",
          penjelasan: "Fitur promosi mandiri yang dapat diterapkan langsung pada barang spesifik atau seluruh etalase kategori tanpa butuh approval platform."
        },
        {
          subBab: "Timer & Penjadwalan",
          penjelasan: "Pengaturan tanggal dan jam mulai/selesai promo secara otomatis, memungkinkan pelaksanaan flash sale independen di luar campaign resmi."
        },
        {
          subBab: "Kupon Toko Mandiri",
          penjelasan: "Pembuatan kode voucher potongan harga atau cashback khusus toko yang bisa dibagikan langsung lewat media sosial atau deskripsi toko."
        },
        {
          subBab: "Analisis Dampak Promo",
          penjelasan: "Metrik pantauan konversi penjualan sebelum dan sesudah promosi dijalankan untuk mengukur efektivitas margin keuntungan."
        }
      ]
    },
    {
      bab: "5. Finansial & Cair",
      subBab: [
        {
          subBab: "Multi Payout Account",
          penjelasan: "Dukungan banyak rekening pencairan dana sekaligus dengan sistem Default Flag untuk kemudahan pengelolaan kas toko."
        },
        {
          subBab: "Pencairan Ekspres",
          penjelasan: "Mekanisme penarikan saldo penjualan yang langsung diproses ke rekening seller tanpa ada penahanan dana berlama-lama oleh sistem."
        },
        {
          subBab: "Transparansi Biaya Admin",
          penjelasan: "Skema potongan platform yang jujur, rendah, dan tanpa biaya tersembunyi, dihitung secara otomatis saat tiap order selesai."
        },
        {
          subBab: "Laporan Akuntansi",
          penjelasan: "Rekapitulasi riwayat transaksi, pemasukan bersih, hingga rincian potongan promo yang dapat diunduh dalam format standar untuk pembukuan."
        }
      ]
    }
  ]);

  let activeBabIndex = $state(0);
  let activeSubBabIndex = $state(0);

  // Svelte 5 Rune: Reaktif otomatis mengikuti perubahan activeBabIndex & activeSubBabIndex
  let currentPenjelasan = $derived(docs[activeBabIndex]?.subBab[activeSubBabIndex]);
</script>

<section id="dokumentasi" class="h-[26rem] w-full grid grid-cols-[17%_83%] border border-zinc-200 bg-white font-sans text-xs select-none p-5">
  <!-- KOLOM KIRI: Navigation (17%) -->
  <div class="border-r border-zinc-200 overflow-y-auto bg-zinc-50/50 p-2 space-y-3">
    {#each docs as item, babIdx}
      <div>
        <div class="px-2 py-1 text-[9px] font-bold uppercase tracking-widest text-zinc-400 truncate">
          {item.bab}
        </div>
        <div class="mt-1 space-y-0.5">
          {#each item.subBab as sub, subIdx}
            <button
              type="button"
              onclick={() => { activeBabIndex = babIdx; activeSubBabIndex = subIdx; }}
              class="w-full text-left px-2 py-1.5 rounded-sm transition-all duration-150 truncate block text-[11px]
                     {activeBabIndex === babIdx && activeSubBabIndex === subIdx 
                       ? 'bg-zinc-900 text-white font-semibold' 
                       : 'text-zinc-600 hover:bg-zinc-200/60 hover:text-zinc-900'}"
            >
              {sub.subBab}
            </button>
          {/each}
        </div>
      </div>
    {/each}
  </div>

  <!-- KOLOM KANAN: Detail Penjelasan (83%) -->
  <div class="overflow-y-auto p-6 bg-white flex flex-col justify-between">
    {#if currentPenjelasan}
      <div class="space-y-3">
        <!-- Breadcrumb Header -->
        <div class="flex items-center gap-1.5 pb-2 border-b border-zinc-100 text-[10px]">
          <span class="font-sans tracking-wider text-zinc-400 uppercase">{docs[activeBabIndex].bab}</span>
          <span class="text-zinc-300">/</span>
          <span class="font-sans font-bold tracking-wider text-zinc-900 uppercase">{currentPenjelasan.subBab}</span>
        </div>
        
        <!-- Judul Sub-Bab -->
        <h3 class="text-sm font-bold text-zinc-900 tracking-tight">
          {currentPenjelasan.subBab}
        </h3>

        <!-- Teks Penjelasan -->
        <p class="text-zinc-600 leading-relaxed font-normal text-xs whitespace-pre-line">
          {currentPenjelasan.penjelasan}
        </p>
      </div>

      <!-- Bottom Metadata Footer -->
      <div class="pt-4 mt-6 border-t border-zinc-100 flex items-center justify-between text-[10px] text-zinc-400 font-sans">
        <span>Kebijakan // Seller Burung</span>
        <span>Hal :: {activeBabIndex + 1}.{activeSubBabIndex + 1}</span>
      </div>
    {:else}
      <div class="h-full flex items-center justify-center text-zinc-400 font-sans text-[11px]">
        NO_DATA_SELECTED
      </div>
    {/if}
  </div>
</section>