<script lang="ts">
  import { goto } from "$app/navigation";
  import type { Snippet } from "svelte";

  let { children }: { children?: Snippet } = $props();

  let stateSidebar = $state<boolean>(true);
</script>

<section class="flex h-screen w-full bg-white font-sans overflow-hidden select-none ">
  <!-- SIDEBAR -->
  {#if stateSidebar}
    <aside id="sidebar" class="w-14 md:w-16 h-full bg-zinc-200 border-r border-zinc-300/70 flex flex-col justify-between py-3 px-2 transition-all duration-200">
      
      <!-- TOP SECTION: Toggle & Avatar -->
      <div class="flex flex-col items-center gap-4">
        <!-- Toggle Button (Collapse) -->
        <button 
          type="button"
          onclick={() => stateSidebar = !stateSidebar}
          class="p-1.5 text-zinc-600 hover:text-zinc-950 hover:bg-zinc-300/70 rounded transition-colors"
          title="Tutup Sidebar"
        >
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="m15 18-6-6 6-6"/>
          </svg>
        </button>

        <!-- Avatar Placeholder -->
        <div class="w-8 h-8 bg-zinc-950 text-white rounded-full flex items-center justify-center font-bold text-xs shadow-sm ring-2 ring-zinc-300">
          B
        </div>

        <div class="w-full h-px bg-zinc-300/80 my-1"></div>
      </div>

      <!-- MIDDLE SECTION: Core Seller Icons -->
      <nav class="flex flex-col items-center gap-1.5 overflow-y-auto py-2 no-scrollbar">
        <!-- Dashboard -->
        <button type="button" onclick={() => {
            goto("/home")
        }} class="p-2 text-zinc-700 hover:text-zinc-950 hover:bg-zinc-300/80 rounded transition-colors" title="Dashboard">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><rect width="7" height="9" x="3" y="3" rx="1"/><rect width="7" height="5" x="14" y="3" rx="1"/><rect width="7" height="9" x="14" y="12" rx="1"/><rect width="7" height="5" x="3" y="16" rx="1"/></svg>
        </button>

        <!-- Barang / Produk -->
        <button type="button" onclick={() => {
            goto("/barang")
        }} class="p-2 text-zinc-700 hover:text-zinc-950 hover:bg-zinc-300/80 rounded transition-colors" title="Barang">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><path d="M21 8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16Z"/><path d="m3.3 7 8.7 5 8.7-5"/><path d="M12 22V12"/></svg>
        </button>

        <!-- Etalase -->
        <button type="button" onclick={() => {
            goto("/etalase")
        }} class="p-2 text-zinc-700 hover:text-zinc-950 hover:bg-zinc-300/80 rounded transition-colors" title="Etalase">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><path d="m2 7 4.41-4.41A2 2 0 0 1 7.83 2h8.34a2 2 0 0 1 1.42.59L22 7"/><path d="M4 12v8a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-8"/><path d="M15 22v-4a2 2 0 0 0-2-2h-2a2 2 0 0 0-2 2v4"/><path d="M2 7h20"/><path d="M10 12h4"/></svg>
        </button>

        <!-- Transaksi -->
        <button onclick={() => {
            goto("/transaksi")
        }} type="button" class="p-2 text-zinc-700 hover:text-zinc-950 hover:bg-zinc-300/80 rounded transition-colors" title="Transaksi">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><path d="M4 2v20l2-1 2 1 2-1 2 1 2-1 2 1 2-1 2 1V2l-2 1-2-1-2 1-2-1-2 1-2-1-2 1Z"/><path d="M16 8h-6a2 2 0 1 0 0 4h4a2 2 0 1 1 0 4H8"/><path d="M12 6v12"/></svg>
        </button>

        <!-- Diskon -->
        <button onclick={() => {
            goto("/diskon")
        }} type="button" class="p-2 text-zinc-700 hover:text-zinc-950 hover:bg-zinc-300/80 rounded transition-colors" title="Diskon">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><path d="M12.586 2.586A2 2 0 0 0 11.172 2H4a2 2 0 0 0-2 2v7.172a2 2 0 0 0 .586 1.414l8.704 8.704a2.41 2.41 0 0 0 3.408 0l5.88-5.88a2.41 2.41 0 0 0 0-3.408z"/><circle cx="7.5" cy="7.5" r=".5" fill="currentColor"/></svg>
        </button>

        <!-- Gudang / Alamat -->
        <button onclick={() => {
            goto("/gudang")
        }} type="button" class="p-2 text-zinc-700 hover:text-zinc-950 hover:bg-zinc-300/80 rounded transition-colors" title="Gudang">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><path d="M22 8.35V20a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V8.35A2 2 0 0 1 3.26 6.5l8-3.2a2 2 0 0 1 1.48 0l8 3.2A2 2 0 0 1 22 8.35Z"/><path d="M6 18h12v-7H6v7Z"/></svg>
        </button>

        <!-- Berkas / Dokumen -->
        <button onclick={() => {
            goto("/dokumen_jenis")
        }} type="button" class="p-2 text-zinc-700 hover:text-zinc-950 hover:bg-zinc-300/80 rounded transition-colors" title="Dokumen Legalitas">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><path d="M15 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V7Z"/><path d="M14 2v4a1 1 0 0 0 1 1h4"/><path d="M10 9H8"/><path d="M16 13H8"/><path d="M16 17H8"/></svg>
        </button>

        <!-- Social Media -->
        <button onclick={() => {
            goto("/social_media")
        }} type="button" class="p-2 text-zinc-700 hover:text-zinc-950 hover:bg-zinc-300/80 rounded transition-colors" title="Social Media">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><circle cx="18" cy="5" r="3"/><circle cx="6" cy="12" r="3"/><circle cx="18" cy="19" r="3"/><line x1="8.59" x2="15.42" y1="10.68" y2="6.32"/><line x1="8.59" x2="15.42" y1="13.32" y2="17.68"/></svg>
        </button>
      </nav>

      <!-- BOTTOM SECTION: Settings & Credential -->
      <div class="flex flex-col items-center gap-1.5 pt-2 border-t border-zinc-300/80">
        <!-- Settings Credential -->
        <button onclick={() => {
            goto("/credential")
        }}  type="button" class="p-2 text-zinc-700 hover:text-zinc-950 hover:bg-zinc-300/80 rounded transition-colors" title="Kredensial & Keamanan">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><path d="M20 13c0 5-3.5 7.5-7.66 8.95a1 1 0 0 1-.67-.01C7.5 20.5 4 18 4 13V6a1 1 0 0 1 1-1c2 0 4.5-1.2 6.24-2.72a1.17 1.17 0 0 1 1.52 0C14.51 3.81 17 5 19 5a1 1 0 0 1 1 1z"/><path d="m9 12 2 2 4-4"/></svg>
        </button>

        <!-- Settings Sistem -->
        <button type="button" class="p-2 text-zinc-700 hover:text-zinc-950 hover:bg-zinc-300/80 rounded transition-colors" title="Pengaturan Toko">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><path d="M12.22 2h-.44a2 2 0 0 0-2 2v.18a2 2 0 0 1-1 1.73l-.43.25a2 2 0 0 1-2 0l-.15-.08a2 2 0 0 0-2.73.73l-.22.38a2 2 0 0 0 .73 2.73l.15.1a2 2 0 0 1 1 1.72v.51a2 2 0 0 1-1 1.74l-.15.09a2 2 0 0 0-.73 2.73l.22.38a2 2 0 0 0 2.73.73l.15-.08a2 2 0 0 1 2 0l.43.25a2 2 0 0 1 1 1.73V20a2 2 0 0 0 2 2h.44a2 2 0 0 0 2-2v-.18a2 2 0 0 1 1-1.73l.43-.25a2 2 0 0 1 2 0l.15.08a2 2 0 0 0 2.73-.73l.22-.38a2 2 0 0 0-.73-2.73l-.15-.1a2 2 0 0 1-1-1.72v-.51a2 2 0 0 1 1-1.74l.15-.09a2 2 0 0 0 .73-2.73l-.22-.38a2 2 0 0 0-2.73-.73l-.15.08a2 2 0 0 1-2 0l-.43-.25a2 2 0 0 1-1-1.73V4a2 2 0 0 0-2-2z"/><circle cx="12" cy="12" r="3"/></svg>
        </button>
      </div>

    </aside>
  {/if}

  <!-- MAIN CONTENT AREA -->
  <main class="flex-1 h-full overflow-y-auto bg-zinc-50 relative scrollbar-none">
    {#if !stateSidebar}
      <!-- Floating Buka Sidebar Button -->
      <button 
        type="button"
        onclick={() => stateSidebar = !stateSidebar}
        class="absolute top-3 left-3 z-10 p-2 bg-zinc-200 border border-zinc-300 text-zinc-700 hover:text-zinc-950 hover:bg-zinc-300 rounded shadow-sm transition-all"
        title="Buka Sidebar"
      >
        <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <path d="m9 18 6-6-6-6"/>
        </svg>
      </button>
    {/if}

    {@render children?.()}
  </main>
</section>