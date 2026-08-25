<script lang="ts">
  const testimonials = [
    {
      highlight: "“OMZET NAIK 3X LIPAT DALAM 2 BULAN.”",
      quote: "Fitur etalase kustom dan sistem promo mandiri bikin pembeli lebih yakin. Dulu cuma jualan via grup WA, sekarang punya storefront resmi sendiri.",
      author: "Hendra Wijaya",
      role: "Owner Kicau Master",
      city: "Bandung"
    },
    {
      highlight: "“AKHIRNYA PUNYA KENDALI PENUH TOKO.”",
      quote: "Sistem auto-approve order dan multiple account pencairan dana sangat membantu operasional tim. Platform lain terlalu mengekang.",
      author: "Budi Santoso",
      role: "Distributor Sangkar Mas",
      city: "Surakarta"
    },
    {
      highlight: "“PENCAIRAN TRANSPARAN & CEPAT.”",
      quote: "Nggak ada potongan tersembunyi. Rating dua arah ke kurir juga bikin pengiriman hewan jauh lebih aman dan bertanggung jawab.",
      author: "Aris Munandar",
      role: "Founder Aviary Jaya",
      city: "Surabaya"
    }
  ];
</script>

<section id="testimoni" class="py-24 w-full bg-white text-zinc-950 border-b border-zinc-200 px-8 md:px-16">
  <div class="max-w-7xl mx-auto space-y-16">
    
    <!-- Section Header -->
    <div class="flex flex-col md:flex-row md:items-end justify-between gap-6 border-b border-zinc-200 pb-8">
      <div class="space-y-2 max-w-2xl">
        <span class="text-xs font-bold uppercase tracking-widest text-zinc-400">Pengalaman Mitra</span>
        <h2 class="text-4xl md:text-6xl font-black tracking-tighter text-zinc-950 leading-none uppercase">
          Cerita Dari Sarang
        </h2>
      </div>
      <p class="text-zinc-500 text-sm max-w-xs leading-relaxed font-medium">
        Dengarkan langsung dari para penjual yang mentransformasi bisnisnya di platform ini.
      </p>
    </div>

    <!-- Typographic Cards Grid -->
    <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
      {#each testimonials as item}
        <div class="border border-zinc-200 p-8 md:p-10 flex flex-col justify-between space-y-10 hover:border-zinc-950 transition-colors duration-200 bg-zinc-50/50">
          
          <div class="space-y-6">
            <!-- Typographic Highlight -->
            <h3 class="text-2xl md:text-3xl font-black tracking-tighter text-zinc-950 leading-tight">
              {item.highlight}
            </h3>
            
            <!-- Quote Body -->
            <p class="text-zinc-600 text-sm leading-relaxed font-normal">
              {item.quote}
            </p>
          </div>

          <!-- Author Profile -->
          <div class="pt-6 border-t border-zinc-200 space-y-1">
            <h4 class="text-xs font-extrabold uppercase tracking-wider text-zinc-950">{item.author}</h4>
            <p class="text-[11px] font-medium text-zinc-400 uppercase tracking-widest">{item.role} • {item.city}</p>
          </div>

        </div>
      {/each}
    </div>

  </div>
</section>