<script lang="ts">
  import { goto } from "$app/navigation";

  // Svelte 5 / TS Header Component
</script>

<header class="w-full h-[3rem]  text-slate-950 border-b border-zinc-800/80 grid grid-cols-[10%_70%_20%] items-center px-4 font-sans text-xs select-none">
  <!-- Brand Logo -->
  <div class="flex items-center gap-1.5 truncate">
    <span class="font-black tracking-tight uppercase text-slate-900 text-[11px] md:text-xs">
      BURUNG
    </span>
    <span class="text-zinc-700 font-light text-[10px]">/</span>
    <span class="font-mono font-bold uppercase tracking-wider text-zinc-600 text-[10px] md:text-[11px]">
      Seller
    </span>
  </div>

  <!-- Tengah (Spacer / Container) -->
  <div></div>

  <!-- Right Actions & Profile -->
  <div class="flex items-center gap-2 justify-items-end">
    <!-- Notification Icon -->
    <button 
      type="button" 
      onclick={(e:Event) => {
        e.preventDefault()
        goto("/activity")
      }} 
      class="p-1.5 text-zinc-600 hover:text-white hover:bg-zinc-900 rounded-sm transition-colors duration-150 focus:outline-none"
      aria-label="Notifikasi"
    >
      <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round">
        <path d="M10.268 21a2 2 0 0 0 3.464 0"/>
        <path d="M3.262 15.326A1 1 0 0 0 4 17h16a1 1 0 0 0 .74-1.673C19.41 13.956 18 12.499 18 8A6 6 0 0 0 6 8c0 4.499-1.411 5.956-2.738 7.326"/>
      </svg>
    </button>

    <!-- Mail Icon -->
    <button 
      type="button"
      onclick={(e:Event) => {
        e.preventDefault()
        goto("/mailbox")
      }} 
      class="p-1.5 text-zinc-600 hover:text-white hover:bg-zinc-900 rounded-sm transition-colors duration-150 focus:outline-none"
      aria-label="Pesan"
    >
      <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round">
        <path d="m22 7-8.991 5.727a2 2 0 0 1-2.009 0L2 7"/>
        <rect x="2" y="4" width="20" height="16" rx="2"/>
      </svg>
    </button>

    <!-- Profile Button dengan Divider -->
    <button 
      type="button" 
      class="flex items-center gap-2 text-zinc-700 hover:text-slate-900 transition-colors duration-150 focus:outline-none pl-3 border-l border-zinc-800/80 h-6 truncate w-full justify-start"
    >
        <div class=" h-5 w-5 bg-slate-800 rounded-full">
            <img src="" alt="">
        </div>
      <span class="text-[11px] font-semibold tracking-tight truncate">
        Faiz Hannan Hakim
      </span>
    </button>
  </div>
</header>