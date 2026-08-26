<script lang="ts">
  let isOtpStep = false;

  // Field register sesuai parameter PreSellerRegistration
  let nama = '';
  let username = '';
  let email = '';
  let password = '';
  let jenis = 'Perorangan'; // Options: Perorangan, Distributor, Brand Resmi
  let sellerDedication = 'Peternak & Hobi'; // Options: Peternak & Hobi, UMKM Lokal, Skala Industri

  // Field OTP
  let otpCode = '';

  function handleSubmit() {
    if (!isOtpStep) {
      // Payload PreSellerRegistration
      console.log('PreSellerRegistration Payload:', {
        nama,
        username,
        email,
        jenis,
        SellerDedication: sellerDedication,
        password
      });
      // Simulasi respon HTTP 200 OK -> Lanjut ke step OTP
      isOtpStep = true;
    } else {
      // Submit OTP ke handler verifikasi
      console.log('Verifying OTP:', otpCode);
    }
  }
</script>

<section id="bergabung" class="w-full h-[22rem] bg-white text-zinc-950 border-b border-zinc-200 px-6 md:px-12 flex items-center justify-center overflow-hidden">
  <div class="max-w-6xl w-full h-full flex flex-col md:flex-row items-center justify-between gap-8 py-6">

    <!-- Kolom Kiri: Header & Informasi -->
    <div class="md:w-1/3 space-y-3 flex flex-col justify-center shrink-0">
      <span class="text-xs font-bold uppercase tracking-widest text-zinc-400">Portal Mitra</span>
      <h2 class="text-2xl md:text-3xl font-black tracking-tighter text-zinc-950 leading-tight">
        {#if isOtpStep}
          Verifikasi OTP
        {:else}
          Buka Toko Sekarang
        {/if}
      </h2>
      <p class="text-zinc-500 text-xs md:text-sm leading-relaxed">
        {#if isOtpStep}
          Kode OTP telah dikirimkan ke <span class="font-bold text-zinc-950">{email}</span>. Berlaku selama 3 menit.
        {:else}
          Isi data legalitas & profil awal untuk pendaftaran akun seller secara instan.
        {/if}
      </p>
      {#if !isOtpStep}
        <p class="text-xs text-zinc-400 pt-1">
          Sudah memiliki akun? <a href="/login" class="underline font-bold text-zinc-950 hover:text-zinc-700">Masuk di sini</a>
        </p>
      {/if}
    </div>

    <!-- Kolom Kanan: Form Horizontal Grid -->
    <div class="md:w-2/3 w-full flex flex-col justify-center">
      <form on:submit|preventDefault={handleSubmit} class="space-y-3">
        
        {#if isOtpStep}
          <!-- Step OTP (Tampilan Terfokus) -->
          <div class="max-w-sm mx-auto space-y-3 w-full">
            <div class="space-y-1">
              <label for="otp" class="text-[11px] font-bold uppercase tracking-wider text-zinc-600 block text-center">Masukkan 6 Digit OTP</label>
              <input
                id="otp"
                type="text"
                maxlength="6"
                bind:value={otpCode}
                placeholder="000000"
                required
                class="w-full px-4 py-2 bg-zinc-50 border border-zinc-200 text-center font-mono text-xl tracking-widest text-zinc-950 focus:outline-none focus:border-zinc-950 rounded-none"
              />
            </div>
            <button
              type="submit"
              class="w-full py-2.5 bg-zinc-950 text-white font-bold text-xs hover:bg-zinc-800 transition-colors tracking-wide uppercase"
            >
              Verifikasi & Selesaikan
            </button>
            <button
              type="button"
              on:click={() => isOtpStep = false}
              class="w-full text-center text-xs font-medium text-zinc-500 hover:text-zinc-950 underline block"
            >
              Kembali ke Form Pendaftaran
            </button>
          </div>

        {:else}
          <!-- Form Registration Grid (3 Kolom x 2 Baris) -->
          <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-3">
            
            <!-- Nama Pemilik -->
            <div class="space-y-1">
              <label for="nama" class="text-[11px] font-bold uppercase tracking-wider text-zinc-600 block">Nama Lengkap</label>
              <input
                id="nama"
                type="text"
                bind:value={nama}
                placeholder="Hendra Wijaya"
                required
                class="w-full px-3 py-2 bg-zinc-50 border border-zinc-200 text-xs text-zinc-950 placeholder-zinc-400 focus:outline-none focus:border-zinc-950 rounded-none"
              />
            </div>

            <!-- Username -->
            <div class="space-y-1">
              <label for="username" class="text-[11px] font-bold uppercase tracking-wider text-zinc-600 block">Username Toko</label>
              <input
                id="username"
                type="text"
                bind:value={username}
                placeholder="hendra_aviary"
                required
                class="w-full px-3 py-2 bg-zinc-50 border border-zinc-200 text-xs text-zinc-950 placeholder-zinc-400 focus:outline-none focus:border-zinc-950 rounded-none"
              />
            </div>

            <!-- Email Bisnis -->
            <div class="space-y-1">
              <label for="email" class="text-[11px] font-bold uppercase tracking-wider text-zinc-600 block">Email Bisnis</label>
              <input
                id="email"
                type="email"
                bind:value={email}
                placeholder="nama@tokomu.com"
                required
                class="w-full px-3 py-2 bg-zinc-50 border border-zinc-200 text-xs text-zinc-950 placeholder-zinc-400 focus:outline-none focus:border-zinc-950 rounded-none"
              />
            </div>

            <!-- Jenis Seller -->
            <div class="space-y-1">
              <label for="jenis" class="text-[11px] font-bold uppercase tracking-wider text-zinc-600 block">Jenis Kemitraan</label>
              <select
                id="jenis"
                bind:value={jenis}
                class="w-full px-3 py-2 bg-zinc-50 border border-zinc-200 text-xs text-zinc-950 focus:outline-none focus:border-zinc-950 rounded-none"
              >
                <option value="Perorangan">Perorangan</option>
                <option value="Distributor">Distributor Resmi</option>
                <option value="Brand">Brand Resmi</option>
              </select>
            </div>

            <!-- Dedikasi Seller -->
            <div class="space-y-1">
              <label for="dedication" class="text-[11px] font-bold uppercase tracking-wider text-zinc-600 block">Dedikasi Operasional</label>
              <select
                id="dedication"
                bind:value={sellerDedication}
                class="w-full px-3 py-2 bg-zinc-50 border border-zinc-200 text-xs text-zinc-950 focus:outline-none focus:border-zinc-950 rounded-none"
              >
                <option value="Peternak & Hobi">Peternak & Hobi</option>
                <option value="UMKM Lokal">UMKM Lokal</option>
                <option value="Skala Industri">Skala Industri</option>
              </select>
            </div>

            <!-- Password -->
            <div class="space-y-1">
              <label for="password" class="text-[11px] font-bold uppercase tracking-wider text-zinc-600 block">Kata Sandi</label>
              <input
                id="password"
                type="password"
                bind:value={password}
                placeholder="••••••••"
                required
                class="w-full px-3 py-2 bg-zinc-50 border border-zinc-200 text-xs text-zinc-950 placeholder-zinc-400 focus:outline-none focus:border-zinc-950 rounded-none"
              />
            </div>

          </div>

          <!-- Bottom Actions / Legal -->
          <div class="pt-2 flex flex-col sm:flex-row items-center justify-between gap-3">
            <p class="text-[10px] text-zinc-400 leading-tight">
              Dengan mendaftar, Anda menyetujui <a href="#syarat" class="underline text-zinc-600">Syarat Ketentuan</a> & <a href="#privasi" class="underline text-zinc-600">Kebijakan Privasi</a>.
            </p>
            <button
              type="submit"
              class="w-full sm:w-auto px-8 py-2.5 bg-zinc-950 text-white font-bold text-xs hover:bg-zinc-800 transition-colors tracking-wide uppercase shrink-0"
            >
              Kirim Kode OTP
            </button>
          </div>
        {/if}

      </form>
    </div>

  </div>
</section>