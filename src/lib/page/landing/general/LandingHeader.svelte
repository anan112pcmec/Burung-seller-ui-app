<script lang="ts">
  import { goto } from "$app/navigation";

  let isMobileMenuOpen = false;

  const navLinks = [
    { name: 'Keunggulan', href: '#kenapa-harus-bergabung' },
    { name: 'Integrasi', href: '#tahapan-setelah-integrasi' },
    { name: 'Testimoni', href: '#testimoni' },
    { name: 'Mitra', href: '#mereka-yang-sudah-bergabung' }
  ];
</script>

<header id="header" class="w-full bg-slate-950 text-white  border-zinc-800/80 sticky  z-50 px-8 md:px-16">
  <div class="max-w-7xl mx-auto h-20 flex items-center justify-between">
    
    <!-- Brand Logo -->
    <a href="#" class="flex items-center gap-3 group">
      <span class="text-xl md:text-2xl font-black tracking-tighter uppercase text-white">
        BURUNG
      </span>
      <span class="text-zinc-600 font-light text-lg">|</span>
      <span class="text-xl md:text-2xl font-bold uppercase tracking-widest text-zinc-400 group-hover:text-white transition-colors">
        Seller
      </span>
    </a>

    <!-- Desktop Navigation -->
    <nav class="hidden md:flex items-center gap-8">
      {#each navLinks as link}
        <a 
          href={link.href} 
          class="text-xs font-bold uppercase tracking-widest text-zinc-400 hover:text-white transition-colors"
        >
          {link.name}
        </a>
      {/each}
    </nav>

    <!-- Action Buttons -->
    <div class="hidden md:flex items-center gap-4">
      <button
        onclick={() =>{goto("/sign-in")}} 
        class="text-xs font-bold uppercase tracking-wider text-zinc-300 hover:text-white px-4 py-2 transition-colors"
      >
        Masuk
    </button>
      <a 
        href="#bergabung" 
        class="text-xs font-black uppercase tracking-wider bg-white text-slate-950 px-5 py-2.5 hover:bg-zinc-200 transition-colors"
      >
        Buka Toko
      </a>
    </div>

    <!-- Mobile Hamburger Toggle -->
    <button 
      type="button"
      class="md:hidden text-zinc-400 hover:text-white focus:outline-none"
      onclick={() => isMobileMenuOpen = !isMobileMenuOpen}
      aria-label="Toggle Navigation"
    >
      <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        {#if isMobileMenuOpen}
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
        {:else}
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
        {/if}
      </svg>
    </button>

  </div>

  <!-- Mobile Menu Dropdown -->
  {#if isMobileMenuOpen}
    <div class="md:hidden bg-slate-950 border-b border-zinc-800 px-4 pt-2 pb-6 space-y-4">
      <nav class="flex flex-col space-y-3">
        {#each navLinks as link}
          <a 
            href={link.href} 
            class="text-xs font-bold uppercase tracking-widest text-zinc-400 hover:text-white py-1"
            onclick={() => isMobileMenuOpen = false}
          >
            {link.name}
          </a>
        {/each}
      </nav>
      <div class="pt-4 border-t border-zinc-800 flex flex-col gap-2">
        <button 
          class="text-center text-xs font-bold uppercase tracking-wider text-zinc-300 py-2 border border-zinc-700"
          onclick={() =>{
            goto("/sign-in")
          }}
        >
          Masuk
        </button>
        <a 
          href="#bergabung" 
          class="text-center text-xs font-black uppercase tracking-wider bg-white text-slate-950 py-2.5"
          onclick={() => isMobileMenuOpen = false}
        >
          Buka Toko
        </a>
      </div>
    </div>
  {/if}
</header>