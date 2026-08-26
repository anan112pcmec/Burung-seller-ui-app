<script lang="ts">
  const valueProps = [
    {
      code: '01',
      title: 'Kendali Operasional Penuh',
      desc: 'Atur sistem approve pesanan secara otomatis atau manual, atur jam operasional toko, hingga kelola banyak rekening pencairan dana secara mandiri.'
    },
    {
      code: '02',
      title: 'Jalur Upgrade Eksklusif',
      desc: 'Bukan sekadar seller. Tumbuh bertahap menuju status Distributor resmi hingga Brand Terverifikasi dengan sertifikasi NIB dan Hak Merek.'
    },
    {
      code: '03',
      title: 'Identitas Toko Otentik',
      desc: 'Tampilkan personal branding tokomu. Kelola etalase kustom, sematkan punchline unik, dan raih badge komitmen untuk meningkatkan kredibilitas.'
    },
    {
      code: '04',
      title: 'Promosi Mandiri Tanpa Antre',
      desc: 'Bikin kampanye diskon spesifik per produk atau kategori dengan jadwal otomatis. Eksekusi strategi marketing kapan saja tanpa nunggu approval.'
    },
    {
      code: '05',
      title: 'Transparansi Dua Arah',
      desc: 'Sistem evaluasi yang adil. Selain dinilai pembeli, kamu juga memiliki hak memberikan rating dan ulasan terhadap kualitas layanan kurir.'
    },
    {
      code: '06',
      title: 'Fleksibilitas Pencairan',
      desc: 'Kelola multiple payout account dengan indikator rekening utama. Transaksi lebih aman, transparan, dan pencairan dana langsung dalam kendalimu.'
    }
  ];
</script>

<!-- Section 1: Metrik Singkat -->
<section id="mereka-yang-sudah-bergabung" class="h-[22rem] w-full bg-white border-b border-zinc-200 text-zinc-950 flex items-center px-8 md:px-16">
  <div class="max-w-7xl mx-auto w-full grid grid-cols-1 md:grid-cols-12 gap-8 items-center">
    
    <div class="md:col-span-5 space-y-3">
      <span class="text-xs font-bold uppercase tracking-widest text-zinc-400">Ekosistem Seller</span>
      <h2 class="text-3xl md:text-4xl font-black tracking-tighter text-zinc-950 leading-tight">
        Mereka Yang Sudah Bergabung
      </h2>
      <p class="text-zinc-500 text-sm leading-relaxed max-w-sm">
        Jaringan penjual terverifikasi yang terus tumbuh di seluruh Indonesia dalam satu platform terpadu.
      </p>
    </div>

    <div class="md:col-span-7 grid grid-cols-3 gap-6 border-t md:border-t-0 md:border-l border-zinc-200 pt-6 md:pt-0 md:pl-12">
      
      <div class="space-y-1">
        <h3 class="text-4xl md:text-5xl font-black tracking-tighter text-zinc-950">1,200+</h3>
        <p class="text-xs font-semibold text-zinc-400 uppercase tracking-wider">Mitra Aktif</p>
      </div>

      <div class="space-y-1">
        <h3 class="text-4xl md:text-5xl font-black tracking-tighter text-zinc-950">48</h3>
        <p class="text-xs font-semibold text-zinc-400 uppercase tracking-wider">Kota</p>
      </div>

      <div class="space-y-1">
        <h3 class="text-4xl md:text-5xl font-black tracking-tighter text-zinc-950">99.6%</h3>
        <p class="text-xs font-semibold text-zinc-400 uppercase tracking-wider">Kepuasan</p>
      </div>

    </div>

  </div>
</section>

<!-- Section 2: Carousel Keunggulan Platform -->
<section id="kenapa-harus-bergabung" class="py-20 w-full bg-zinc-50 border-b border-zinc-200 text-zinc-950">
  <div class="max-w-7xl mx-auto px-8 md:px-16 space-y-12">
    
    <!-- Carousel Header -->
    <div class="flex flex-col md:flex-row md:items-end justify-between gap-6">
      <div class="space-y-2 max-w-xl">
        <span class="text-xs font-bold uppercase tracking-widest text-zinc-400">Value Proposition</span>
        <h2 class="text-3xl md:text-4xl font-black tracking-tighter text-zinc-950">
          Mengapa Mengembangkan Toko Di Sini?
        </h2>
      </div>
      <p class="text-zinc-500 text-sm max-w-xs leading-relaxed">
        Platform dirancang untuk memberi fleksibilitas dan kendali penuh atas bisnis Anda.
      </p>
    </div>

    <!-- Horizontal Card Carousel -->
    <div class="flex gap-6 overflow-x-auto snap-x snap-mandatory pb-8 pt-2 scrollbar-thin scrollbar-thumb-zinc-300">
      {#each valueProps as item}
        <div class="min-w-[300px] md:min-w-[360px] max-w-[360px] snap-start border border-zinc-200 bg-white p-8 rounded-none flex flex-col justify-between hover:border-zinc-950 transition-colors duration-200">
          <div class="space-y-6">
            <span class="text-2xl font-black text-zinc-300 tracking-tighter block">{item.code}</span>
            <h3 class="text-xl font-bold tracking-tight text-zinc-950 leading-snug">{item.title}</h3>
            <p class="text-zinc-600 text-sm leading-relaxed font-normal">
              {item.desc}
            </p>
          </div>
          <div class="mt-8 pt-4 border-t border-zinc-100 flex items-center justify-end">
            <span class="text-xs font-bold uppercase tracking-widest text-zinc-400">Burung Ecosystem</span>
          </div>
        </div>
      {/each}
    </div>

  </div>
</section>