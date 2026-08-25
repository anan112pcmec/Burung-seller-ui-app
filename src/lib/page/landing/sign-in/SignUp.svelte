<script lang="ts">
  let mode: 'register' | 'login' = 'register';

  let email = '';
  let password = '';
  let storeName = '';

  function handleSubmit() {
    if (mode === 'register') {
      console.log('Registering seller:', { storeName, email, password });
    } else {
      console.log('Logging in seller:', { email, password });
    }
  }
</script>

<section id="bergabung" class="py-24 w-full bg-white text-zinc-950 border-b border-zinc-200 px-8 md:px-16 flex items-center justify-center">
  <div class="max-w-md w-full space-y-8">

    <!-- Header Section -->
    <div class="text-center space-y-2">
      <span class="text-xs font-bold uppercase tracking-widest text-zinc-400">Portal Mitra</span>
      <h2 class="text-3xl md:text-4xl font-black tracking-tighter text-zinc-950">
        {mode === 'register' ? 'Buka Toko Sekarang' : 'Masuk ke Dashboard'}
      </h2>
      <p class="text-zinc-500 text-sm leading-relaxed">
        {mode === 'register' ? 'Kelola toko, atur promo, dan kendalikan operasional penuh dalam satu tempat.' : 'Akses laporan penjualan, pesanan, dan kelola etalase Anda.'}
      </p>
    </div>

    <!-- Toggle Switch -->
    <div class="grid grid-cols-2 bg-zinc-100 p-1 text-xs font-bold border border-zinc-200">
      <button
        type="button"
        class={`py-2.5 text-center transition-all ${mode === 'register' ? 'bg-zinc-950 text-white shadow-sm' : 'text-zinc-500 hover:text-zinc-950'}`}
        on:click={() => mode = 'register'}
      >
        Daftar Toko
      </button>
      <button
        type="button"
        class={`py-2.5 text-center transition-all ${mode === 'login' ? 'bg-zinc-950 text-white shadow-sm' : 'text-zinc-500 hover:text-zinc-950'}`}
        on:click={() => mode = 'login'}
      >
        Masuk
      </button>
    </div>

    <!-- Dynamic Form -->
    <form on:submit|preventDefault={handleSubmit} class="space-y-4">
      {#if mode === 'register'}
        <div class="space-y-1">
          <label for="store-name" class="text-xs font-bold uppercase tracking-wider text-zinc-600 block">Nama Toko / Aviary</label>
          <input
            id="store-name"
            type="text"
            bind:value={storeName}
            placeholder="Contoh: Sangkar Utama Store"
            required
            class="w-full px-4 py-3 bg-zinc-50 border border-zinc-200 text-sm text-zinc-950 placeholder-zinc-400 focus:outline-none focus:border-zinc-950 transition-colors rounded-none"
          />
        </div>
      {/if}

      <div class="space-y-1">
        <label for="email" class="text-xs font-bold uppercase tracking-wider text-zinc-600 block">Email Bisnis</label>
        <input
          id="email"
          type="email"
          bind:value={email}
          placeholder="nama@tokomu.com"
          required
          class="w-full px-4 py-3 bg-zinc-50 border border-zinc-200 text-sm text-zinc-950 placeholder-zinc-400 focus:outline-none focus:border-zinc-950 transition-colors rounded-none"
        />
      </div>

      <div class="space-y-1">
        <div class="flex justify-between items-center">
          <label for="password" class="text-xs font-bold uppercase tracking-wider text-zinc-600 block">Kata Sandi</label>
          {#if mode === 'login'}
            <a href="#lupa-password" class="text-xs font-medium text-zinc-500 hover:text-zinc-950 underline">Lupa password?</a>
          {/if}
        </div>
        <input
          id="password"
          type="password"
          bind:value={password}
          placeholder="••••••••"
          required
          class="w-full px-4 py-3 bg-zinc-50 border border-zinc-200 text-sm text-zinc-950 placeholder-zinc-400 focus:outline-none focus:border-zinc-950 transition-colors rounded-none"
        />
      </div>

      <button
        type="submit"
        class="w-full py-3.5 bg-zinc-950 text-white font-bold text-sm hover:bg-zinc-800 transition-colors tracking-wide uppercase mt-2"
      >
        {mode === 'register' ? 'Buat Toko Mitra' : 'Masuk ke Dashboard'}
      </button>
    </form>

    <!-- Footer Legal -->
    <p class="text-center text-xs text-zinc-400 leading-relaxed">
      Dengan melanjutkan, Anda menyetujui <a href="#syarat" class="underline text-zinc-600 hover:text-zinc-950">Syarat Ketentuan Mitra</a> & <a href="#privasi" class="underline text-zinc-600 hover:text-zinc-950">Kebijakan Privasi</a> platform.
    </p>

  </div>
</section>