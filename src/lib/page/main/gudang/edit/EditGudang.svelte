<script lang="ts">
  import { preventDefault } from 'svelte/legacy';

  // Tipe data berdasarkan Go Struct AlamatGudang
  type AlamatGudang = {
    id_alamat_gudang: number;
    id_seller_alamat_gudang: number;
    panggilan_alamat_gudang: string;
    nomor_telfon_alamat_gudang: string;
    nama_alamat_gudang: string;
    provinsi_alamat_gudang: string;
    kota_alamat_gudang: string;
    kode_pos_alamat_gudang: string;
    kode_negara_alamat_gudang: string;
    deskripsi_alamat_gudang: string;
    longitude_alamat_gudang: number;
    latitude_alamat_gudang: number;
    foto_gudang_url?: string;
  };

  // State awal (reaktif menggunakan Svelte 5 $state)
  let form = $state<AlamatGudang>({
    id_alamat_gudang: 101,
    id_seller_alamat_gudang: 1,
    panggilan_alamat_gudang: 'Gudang Utama Jakarta',
    nomor_telfon_alamat_gudang: '081234567890',
    nama_alamat_gudang: 'Jl. Industri Raya No. 45, Kebon Jeruk',
    provinsi_alamat_gudang: 'DKI Jakarta',
    kota_alamat_gudang: 'Jakarta Barat',
    kode_pos_alamat_gudang: '11530',
    kode_negara_alamat_gudang: 'IDN',
    deskripsi_alamat_gudang: 'Gudang khusus penyimpanan barang elektronik dan suku cadang.',
    longitude_alamat_gudang: 106.7682,
    latitude_alamat_gudang: -6.1894,
    foto_gudang_url: ''
  });

  let fileFoto = $state<File | null>(null);
  let previewFotoUrl = $state<string>(form.foto_gudang_url || '');

  // Handle upload foto dari microservice lain
  function handleFileChange(e: Event) {
    const target = e.target as HTMLInputElement;
    if (target.files && target.files[0]) {
      fileFoto = target.files[0];
      previewFotoUrl = URL.createObjectURL(fileFoto);
    }
  }

  // Kalkulasi Kelengkapan Profil (0 - 100%)
  let persentaseKelengkapan = $derived(() => {
    const fields = [
      form.panggilan_alamat_gudang,
      form.nomor_telfon_alamat_gudang,
      form.nama_alamat_gudang,
      form.provinsi_alamat_gudang,
      form.kota_alamat_gudang,
      form.kode_pos_alamat_gudang,
      form.deskripsi_alamat_gudang,
      form.latitude_alamat_gudang !== 0,
      form.longitude_alamat_gudang !== 0,
      previewFotoUrl !== ''
    ];
    const filled = fields.filter((f) => Boolean(f)).length;
    return Math.round((filled / fields.length) * 100);
  });

  function handleSubmit() {
    // Logic pengiriman payload form ke Service Alamat & fileFoto ke Service Media Upload
    console.log('Payload Gudang:', form);
    console.log('Payload Foto (Microservice Media):', fileFoto);
  }
</script>

<div class="max-w-7xl mx-auto p-6 grid grid-cols-1 lg:grid-cols-12 gap-8 font-sans bg-zinc-50 min-h-screen">
  
  <!-- SISI KIRI: FORM EDIT DATA (LGI: 7 Columns) -->
  <div class="lg:col-span-7 bg-white p-6 rounded-xl border border-zinc-200 shadow-2xs flex flex-col gap-6">
    <div>
      <h1 class="text-xl font-bold text-zinc-900">Edit Data Gudang</h1>
      <p class="text-xs text-zinc-500 mt-0.5">Perbarui informasi lokasi, kontak, dan foto media gudang Anda.</p>
    </div>

    <form onsubmit={preventDefault(handleSubmit)} class="flex flex-col gap-5">
      
      <!-- Upload Foto Gudang (Handled oleh Microservice Foto/Media) -->
      <div class="flex flex-col gap-2 p-4 bg-zinc-50 border border-zinc-200 rounded-lg">
        <label class="text-xs font-semibold text-zinc-700">Foto Area Gudang</label>
        <div class="flex items-center gap-4">
          <div class="w-20 h-20 rounded-md bg-zinc-200 border border-zinc-300 overflow-hidden flex items-center justify-center shrink-0">
            {#if previewFotoUrl}
              <img src={previewFotoUrl} alt="Preview Upload" class="w-full h-full object-cover" />
            {:else}
              <span class="text-[10px] text-zinc-400 font-mono text-center px-1">Tanpa Foto</span>
            {/if}
          </div>
          <div class="flex flex-col gap-1.5">
            <input 
              type="file" 
              accept="image/*" 
              onchange={handleFileChange}
              id="upload-foto"
              class="hidden" 
            />
            <label 
              for="upload-foto" 
              class="px-3 py-1.5 text-xs font-medium bg-white border border-zinc-300 rounded-md shadow-2xs hover:bg-zinc-100 cursor-pointer text-center text-zinc-700"
            >
              Pilih Foto Baru
            </label>
            <span class="text-[10px] text-zinc-400">Format JPG, PNG max 2MB (Tersimpan di Service Media)</span>
          </div>
        </div>
      </div>

      <!-- Informasional Utama -->
      <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
        <div class="flex flex-col gap-1">
          <label for="panggilan" class="text-xs font-semibold text-zinc-700">Nama Panggilan Gudang *</label>
          <input 
            id="panggilan"
            type="text" 
            bind:value={form.panggilan_alamat_gudang} 
            placeholder="Contoh: Gudang Utama"
            class="px-3 py-2 text-xs border border-zinc-300 rounded-md focus:outline-none focus:border-zinc-500 font-medium"
            required
          />
        </div>

        <div class="flex flex-col gap-1">
          <label for="telepon" class="text-xs font-semibold text-zinc-700">Nomor Telepon *</label>
          <input 
            id="telepon"
            type="text" 
            bind:value={form.nomor_telfon_alamat_gudang} 
            placeholder="08xxxxxxxxxx"
            class="px-3 py-2 text-xs border border-zinc-300 rounded-md focus:outline-none focus:border-zinc-500 font-mono"
            required
          />
        </div>
      </div>

      <!-- Alamat Lengkap -->
      <div class="flex flex-col gap-1">
        <label for="alamat" class="text-xs font-semibold text-zinc-700">Alamat Lengkap *</label>
        <textarea 
          id="alamat"
          bind:value={form.nama_alamat_gudang} 
          rows="2"
          placeholder="Nama jalan, nomor bangunan, RT/RW"
          class="px-3 py-2 text-xs border border-zinc-300 rounded-md focus:outline-none focus:border-zinc-500"
          required
        ></textarea>
      </div>

      <!-- Detail Wilayah -->
      <div class="grid grid-cols-1 sm:grid-cols-3 gap-4">
        <div class="flex flex-col gap-1">
          <label for="provinsi" class="text-xs font-semibold text-zinc-700">Provinsi *</label>
          <input 
            id="provinsi"
            type="text" 
            bind:value={form.provinsi_alamat_gudang} 
            class="px-3 py-2 text-xs border border-zinc-300 rounded-md focus:outline-none focus:border-zinc-500"
            required
          />
        </div>

        <div class="flex flex-col gap-1">
          <label for="kota" class="text-xs font-semibold text-zinc-700">Kota/Kabupaten *</label>
          <input 
            id="kota"
            type="text" 
            bind:value={form.kota_alamat_gudang} 
            class="px-3 py-2 text-xs border border-zinc-300 rounded-md focus:outline-none focus:border-zinc-500"
            required
          />
        </div>

        <div class="flex flex-col gap-1">
          <label for="kodepos" class="text-xs font-semibold text-zinc-700">Kode Pos *</label>
          <input 
            id="kodepos"
            type="text" 
            bind:value={form.kode_pos_alamat_gudang} 
            class="px-3 py-2 text-xs border border-zinc-300 rounded-md focus:outline-none focus:border-zinc-500 font-mono"
            required
          />
        </div>
      </div>

      <!-- Koordinat Geografis -->
      <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
        <div class="flex flex-col gap-1">
          <label for="latitude" class="text-xs font-semibold text-zinc-700">Latitude</label>
          <input 
            id="latitude"
            type="number" 
            step="any"
            bind:value={form.latitude_alamat_gudang} 
            class="px-3 py-2 text-xs border border-zinc-300 rounded-md focus:outline-none focus:border-zinc-500 font-mono"
          />
        </div>

        <div class="flex flex-col gap-1">
          <label for="longitude" class="text-xs font-semibold text-zinc-700">Longitude</label>
          <input 
            id="longitude"
            type="number" 
            step="any"
            bind:value={form.longitude_alamat_gudang} 
            class="px-3 py-2 text-xs border border-zinc-300 rounded-md focus:outline-none focus:border-zinc-500 font-mono"
          />
        </div>
      </div>

      <!-- Deskripsi -->
      <div class="flex flex-col gap-1">
        <label for="deskripsi" class="text-xs font-semibold text-zinc-700">Deskripsi Catatan Gudang</label>
        <textarea 
          id="deskripsi"
          bind:value={form.deskripsi_alamat_gudang} 
          rows="3"
          placeholder="Catatan akses bongkar muat, spesifikasi area, dll."
          class="px-3 py-2 text-xs border border-zinc-300 rounded-md focus:outline-none focus:border-zinc-500"
        ></textarea>
      </div>

      <!-- Actions -->
      <div class="flex justify-end gap-3 pt-4 border-t border-zinc-100">
        <button 
          type="button" 
          class="px-4 py-2 text-xs font-medium text-zinc-600 bg-zinc-100 hover:bg-zinc-200 rounded-md transition-colors"
        >
          Batal
        </button>
        <button 
          type="submit" 
          class="px-4 py-2 text-xs font-semibold text-white bg-zinc-900 hover:bg-zinc-800 rounded-md shadow-2xs transition-colors"
        >
          Simpan Perubahan
        </button>
      </div>
    </form>
  </div>

  <!-- SISI KANAN: PREVIEW LIVE & INDIKATOR KELENGKAPAN (LGI: 5 Columns) -->
  <div class="lg:col-span-5 flex flex-col gap-6">
    
    <!-- Widget Indicator Kelengkapan Data -->
    <div class="bg-white p-5 rounded-xl border border-zinc-200 shadow-2xs flex flex-col gap-3">
      <div class="flex items-center justify-between">
        <span class="text-xs font-bold text-zinc-900">Kelengkapan Profil Gudang</span>
        <span class="text-xs font-mono font-bold text-zinc-900">{persentaseKelengkapan()}%</span>
      </div>
      
      <!-- Progress Bar -->
      <div class="w-full h-2 bg-zinc-100 rounded-full overflow-hidden">
        <div 
          class="h-full bg-zinc-900 transition-all duration-300 ease-out" 
          style="width: {persentaseKelengkapan()}%"
        ></div>
      </div>

      <!-- Checklist items -->
      <div class="grid grid-cols-2 gap-2 pt-2 text-[11px] text-zinc-500">
        <div class="flex items-center gap-1.5">
          <span class={form.panggilan_alamat_gudang ? "text-emerald-600" : "text-zinc-300"}>✓</span>
          <span>Nama Gudang</span>
        </div>
        <div class="flex items-center gap-1.5">
          <span class={form.nomor_telfon_alamat_gudang ? "text-emerald-600" : "text-zinc-300"}>✓</span>
          <span>No. Telepon</span>
        </div>
        <div class="flex items-center gap-1.5">
          <span class={form.nama_alamat_gudang ? "text-emerald-600" : "text-zinc-300"}>✓</span>
          <span>Alamat Jalan</span>
        </div>
        <div class="flex items-center gap-1.5">
          <span class={form.kode_pos_alamat_gudang ? "text-emerald-600" : "text-zinc-300"}>✓</span>
          <span>Kode Pos</span>
        </div>
        <div class="flex items-center gap-1.5">
          <span class={form.latitude_alamat_gudang && form.longitude_alamat_gudang ? "text-emerald-600" : "text-zinc-300"}>✓</span>
          <span>Koordinat GPS</span>
        </div>
        <div class="flex items-center gap-1.5">
          <span class={previewFotoUrl ? "text-emerald-600" : "text-zinc-300"}>✓</span>
          <span>Foto Gudang</span>
        </div>
      </div>
    </div>

    <!-- Preview Card Live -->
    <div class="flex flex-col gap-2">
      <span class="text-xs font-semibold text-zinc-500 tracking-wide uppercase font-mono">Tampilan Card (Preview)</span>
      
      <!-- Sesuai Desain Card Sebelumnya -->
      <div class="w-full h-[14rem] grid grid-cols-[36%_64%] border border-zinc-200 rounded-lg bg-white shadow-2xs overflow-hidden">
        
        <!-- PETA / FOTO (SISI KIRI) -->
        <div class="relative w-full h-full border-r border-zinc-200 bg-zinc-100 overflow-hidden">
          {#if previewFotoUrl}
            <img src={previewFotoUrl} alt="Foto Gudang" class="w-full h-full object-cover" />
          {:else}
            <div class="w-full h-full flex flex-col items-center justify-center p-2 text-center text-zinc-400">
              <span class="text-[10px] font-mono">Tidak ada foto</span>
            </div>
          {/if}
          <span class="absolute bottom-2 right-2 bg-white/95 border border-zinc-200 rounded px-1.5 py-0.5 text-[8px] font-mono uppercase tracking-wider text-zinc-700 shadow-2xs">
            {form.kode_negara_alamat_gudang || 'IDN'}
          </span>
        </div>

        <!-- DETAIL GUDANG (SISI KANAN) -->
        <div class="p-3.5 flex flex-col justify-between bg-white h-full">
          <div class="flex flex-col gap-1">
            <div class="flex items-start justify-between gap-1">
              <h3 class="text-xs font-bold text-zinc-900 leading-tight truncate">
                {form.panggilan_alamat_gudang || 'Nama Gudang'}
              </h3>
              <span class="text-[9px] text-zinc-400 font-mono shrink-0">Hari Ini</span>
            </div>

            <p class="text-[10px] text-zinc-500 leading-tight truncate">
              {form.nama_alamat_gudang || 'Alamat Belum Diisi'}, {form.kota_alamat_gudang}
            </p>

            <p class="text-[10px] text-zinc-400 line-clamp-2 leading-snug mt-0.5">
              {form.deskripsi_alamat_gudang || 'Belum ada deskripsi.'}
            </p>
          </div>

          <div class="flex flex-col gap-2 pt-2 border-t border-zinc-100">
            <div class="flex items-center gap-1 text-[10px] text-zinc-500 font-mono">
              <svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-zinc-400 stroke-2" viewBox="0 0 24 24" fill="none" stroke="currentColor">
                <path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z"/>
              </svg>
              <span>{form.nomor_telfon_alamat_gudang || '-'}</span>
            </div>

            <div class="grid grid-cols-2 gap-2 bg-zinc-50 p-1.5 rounded border border-zinc-100">
              <div class="flex flex-col">
                <span class="text-[8px] font-bold tracking-wider text-zinc-400 uppercase font-mono">Lat / Long</span>
                <span class="text-[10px] font-mono text-zinc-700 truncate">
                  {form.latitude_alamat_gudang || 0}, {form.longitude_alamat_gudang || 0}
                </span>
              </div>
            </div>
          </div>
        </div>

      </div>
    </div>

  </div>
</div>