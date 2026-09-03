<script lang="ts">
	import { goto } from "$app/navigation";

    const {id, path, fun}: {id:number, path: string, fun?: () => void} = $props()
</script>

 <div 
 role="button"
 onclick={() => {
    if (fun){
        fun()
    }

    goto(path)
 }}
 class="etalase-overview-{id} relative w-full aspect-square bg-zinc-50 border border-zinc-950/10 flex flex-col items-center justify-center gap-3 group hover:border-slate-950 hover:bg-white transition-colors cursor-pointer">

    
    <!-- Ikon Etalase Struktural -->
    <svg class="w-7 h-7 text-zinc-300 group-hover:text-slate-900 transition-colors" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5">
      <path stroke-linecap="square" stroke-linejoin="miter" d="M3.75 9.776c.112-.017.227-.026.344-.026h15.812c.117 0 .232.009.344.026m-16.5 0a2.25 2.25 0 00-1.883 2.542l.857 6a2.25 2.25 0 002.227 1.932H19.05a2.25 2.25 0 002.227-1.932l.857-6a2.25 2.25 0 00-1.883-2.542m-16.5 0V6A2.25 2.25 0 016 3.75h3.879a1.5 1.5 0 011.06.44l2.122 2.12a1.5 1.5 0 001.06.44H18A2.25 2.25 0 0120.25 9v.776" />
    </svg>

    <!-- Tipografi Label -->
    <div class="flex flex-col items-center gap-0.5">
        <span class="font-mono text-[10px] text-zinc-500 uppercase tracking-widest font-bold group-hover:text-slate-950 transition-colors">
          Etalase
        </span>
        <span class="font-mono text-[9px] text-zinc-400 group-hover:text-zinc-500 transition-colors">
          #{id + 1}
        </span>
    </div>

    <!-- Aksen Garis Aktif (Muncul saat di-hover) -->
    <div class="absolute bottom-0 left-0 w-full h-[2px] bg-slate-950 scale-x-0 group-hover:scale-x-100 transition-transform origin-left duration-300"></div>
  </div>