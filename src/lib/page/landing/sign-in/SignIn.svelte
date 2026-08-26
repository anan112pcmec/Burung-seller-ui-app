<script lang="ts">
  import { goto } from "$app/navigation";

  let email = $state('');
  let password = $state('');
  let showPassword = $state(false);

  function handleEmailLogin(e: SubmitEvent) {
    e.preventDefault();
    console.log('Logging in with:', { email, password });
    // Integrasi dengan endpoint Auth Go Fiber
  }

  function handleGoogleSSO() {
    console.log('Triggering Google SSO login...');
    // Redirect ke URL Google OAuth
  }
</script>

<div class="min-h-screen w-full bg-white text-zinc-950 flex flex-col justify-center items-center p-6 border-b border-zinc-200 select-none">
  <div class="w-full max-w-md border border-zinc-200 bg-white p-8 md:p-10 space-y-8 shadow-sm hover:border-zinc-950 transition-colors duration-200">
    
    <!-- Header -->
    <div class="space-y-3 border-b border-zinc-200 pb-6">
      <span class="text-[10px] font-black uppercase tracking-widest text-zinc-400">Portal Mitra Seller</span>
      <h1 class="text-3xl font-black tracking-tighter uppercase text-zinc-950 leading-none">
        Masuk Ke Akun
      </h1>
      <p class="text-zinc-500 text-xs font-medium leading-relaxed">
        Kelola etalase, transaksi, dan operasional tokomu secara terpusat.
      </p>
    </div>

    <div class="space-y-6">
      <!-- SSO Google Button -->
      <button
        type="button"
        onclick={handleGoogleSSO}
        class="w-full flex items-center justify-center gap-3 bg-zinc-50 hover:bg-zinc-100 border border-zinc-200 text-zinc-950 font-bold text-xs uppercase tracking-wider py-3 px-4 transition-colors"
      >
        <svg class="w-4 h-4" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M23.745 12.27c0-.7-.06-1.4-.19-2.07H12v4.51h6.6c-.29 1.52-1.14 2.82-2.4 3.68v3.05h3.88c2.27-2.09 3.665-5.17 3.665-9.17z" fill="#4285F4"/>
          <path d="M12 24c3.24 0 5.95-1.08 7.93-2.91l-3.88-3.05c-1.08.72-2.45 1.16-4.05 1.16-3.11 0-5.74-2.1-6.68-4.93H1.37v3.15C3.34 21.32 7.39 24 12 24z" fill="#34A853"/>
          <path d="M5.32 14.27a7.22 7.22 0 0 1 0-4.54V6.58H1.37a11.98 11.98 0 0 0 0 10.84l3.95-3.15z" fill="#FBBC05"/>
          <path d="M12 4.75c1.77 0 3.35.61 4.6 1.8l3.42-3.42C17.95 1.19 15.24 0 12 0 7.39 0 3.34 2.68 1.37 6.58l3.95 3.15c.94-2.83 3.57-4.98 6.68-4.98z" fill="#EA4335"/>
        </svg>
        Lanjutkan dengan Google
      </button>

      <!-- Divider Separator -->
      <div class="relative flex items-center justify-center">
        <div class="border-t border-zinc-200 w-full"></div>
        <span class="bg-white px-3 text-[10px] font-black uppercase tracking-widest text-zinc-400 absolute">Atau</span>
      </div>

      <!-- Form Sign In -->
      <form onsubmit={handleEmailLogin} class="space-y-5">
        
        <!-- Field Email -->
        <div class="space-y-2">
          <label for="email" class="text-[10px] font-black uppercase tracking-widest text-zinc-950 block">Email Bisnis</label>
          <input
            id="email"
            type="email"
            bind:value={email}
            placeholder="nama@tokomu.com"
            required
            class="w-full px-4 py-3 bg-white border border-zinc-200 text-xs text-zinc-950 placeholder-zinc-400 font-medium focus:outline-none focus:border-zinc-950 transition-colors"
          />
        </div>

        <!-- Field Password -->
        <div class="space-y-2">
          <div class="flex items-center justify-between">
            <label for="password" class="text-[10px] font-black uppercase tracking-widest text-zinc-950 block">Kata Sandi</label>
            <a href="#lupa-password" class="text-[10px] font-bold uppercase tracking-wider text-zinc-400 hover:text-zinc-950 transition-colors">Lupa sandi?</a>
          </div>
          <div class="relative">
            <input
              id="password"
              type={showPassword ? 'text' : 'password'}
              bind:value={password}
              placeholder="••••••••"
              required
              class="w-full px-4 py-3 bg-white border border-zinc-200 text-xs text-zinc-950 placeholder-zinc-400 font-medium focus:outline-none focus:border-zinc-950 transition-colors pr-16"
            />
            <button
              type="button"
              onclick={() => (showPassword = !showPassword)}
              class="absolute right-3 top-1/2 -translate-y-1/2 text-[10px] font-black uppercase tracking-widest text-zinc-400 hover:text-zinc-950"
            >
              {showPassword ? 'Sembunyi' : 'Lihat'}
            </button>
          </div>
        </div>

        <!-- Submit Button -->
        <button
          type="submit"
          class="w-full py-3.5 bg-zinc-950 hover:bg-zinc-800 text-white font-black text-xs uppercase tracking-widest transition-colors"
        >
          Masuk Ke Dashboard
        </button>
      </form>
    </div>

    <!-- Footer Register -->
    <div class="pt-6 border-t border-zinc-200 flex items-center justify-between text-xs">
      <span class="text-zinc-500 font-medium">Belum terdaftar?</span>
      <button 
        onclick={() => goto("/sign-up")} 
        class="font-black uppercase tracking-wider text-zinc-950 hover:underline text-xs"
      >
        Daftar Mitra Sekarang
      </button>
    </div>

  </div>
</div>