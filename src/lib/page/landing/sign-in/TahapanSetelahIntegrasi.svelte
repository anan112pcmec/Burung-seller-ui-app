<script lang="ts">
  const steps = [
    {
      day: "01",
      tag: "HARI 1",
      title: "Registrasi & Berkas",
      desc: "Unggah identitas toko, NIB (opsional untuk distributor), dan tentukan rekening pencairan dana."
    },
    {
      day: "02-03",
      tag: "HARI 2-3",
      title: "Verifikasi Kilat",
      desc: "Sistem melakukan validasi otomatis. Akun terverifikasi tanpa perlu menunggu konfirmasi berbulan-bulan."
    },
    {
      day: "04-05",
      tag: "HARI 4-5",
      title: "Setup Etalase & Stok",
      desc: "Import katalog produk, atur kategori kustom, serta sesuaikan jadwal promo mandiri tokomu."
    },
    {
      day: "06",
      tag: "HARI 6",
      title: "Uji Coba Kirim",
      desc: "Simulasi penerimaan pesanan dan pengujian integrasi pelacak pengiriman hewan secara real-time."
    },
    {
      day: "07",
      tag: "HARI 7",
      title: "Resmi Beroperasi",
      desc: "Toko aktif sepenuhnya di ekosistem Burung. Mulai jualan dan terima pembayaran langsung ke rekening utama."
    }
  ];
</script>

<section id="tahapan-setelah-integrasi" class="py-24 w-full bg-white text-zinc-950 border-b border-zinc-200 px-8 md:px-16">
  <div class="max-w-7xl mx-auto space-y-16">
    
    <!-- Section Header -->
    <div class="flex flex-col md:flex-row md:items-end justify-between gap-6">
      <div class="space-y-3 max-w-xl">
        <span class="text-xs font-bold uppercase tracking-widest text-zinc-400">Onboarding 7 Hari</span>
        <h2 class="text-3xl md:text-5xl font-black tracking-tighter text-zinc-950 leading-tight">
          Validasi Cepat. Siap Jualan Dalam 1 Minggu.
        </h2>
      </div>
      <p class="text-zinc-500 text-sm max-w-xs leading-relaxed font-medium">
        Tidak ada proses rumit yang memakan waktu berbulan-bulan. Kami memangkas birokrasi integrasi agar bisnis Anda langsung berjalan.
      </p>
    </div>

    <!-- Timeline Grid with Minimalist Date Badges -->
    <div class="relative grid grid-cols-1 md:grid-cols-5 gap-6 pt-4">
      
      <!-- Horizontal Line Connector (Desktop Only) -->
      <div class="hidden md:block absolute top-[3.25rem] left-8 right-8 h-[1px] bg-zinc-200 -z-0"></div>

      {#each steps as step, i}
        <div class="relative z-10 bg-white border border-zinc-200 p-6 flex flex-col justify-between space-y-6 hover:border-zinc-950 transition-colors duration-200">
          
          <div class="space-y-4">
            <!-- Minimalist Date/Calendar Badge -->
            <div class="w-16 border border-zinc-950 bg-zinc-50 overflow-hidden text-center">
              <div class="bg-zinc-950 text-white text-[9px] font-black uppercase tracking-widest py-0.5">
                {step.tag}
              </div>
              <div class="text-xl font-black text-zinc-950 py-1.5 tracking-tighter">
                {step.day}
              </div>
            </div>

            <!-- Content -->
            <div class="space-y-2">
              <h3 class="text-lg font-bold tracking-tight text-zinc-950 leading-snug">
                {step.title}
              </h3>
              <p class="text-zinc-500 text-xs leading-relaxed font-normal">
                {step.desc}
              </p>
            </div>
          </div>

          <!-- Step Indicator Footnote -->
          <div class="pt-4 border-t border-zinc-100 text-[10px] font-bold text-zinc-400 uppercase tracking-widest">
            Langkah 0{i + 1}
          </div>

        </div>
      {/each}

    </div>

    <!-- Banner Catatan Tambahan -->
    <div class="bg-zinc-50 border border-zinc-200 p-6 flex flex-col md:flex-row items-start md:items-center justify-between gap-4">
      <div class="space-y-1">
        <h4 class="text-sm font-bold text-zinc-950">Butuh bantuan migrasi dari marketplace lain?</h4>
        <p class="text-xs text-zinc-500">Tim technical support siap memandu proses import data barang & etalase toko Anda secara manual maupun API.</p>
      </div>
      <a href="#support" class="text-xs font-bold uppercase tracking-wider bg-zinc-950 text-white px-5 py-3 hover:bg-zinc-800 transition-colors shrink-0">
        Konsultasi Migrasi
      </a>
    </div>

  </div>
</section>