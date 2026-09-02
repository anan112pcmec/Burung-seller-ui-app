<script lang="ts">
    let {kategori, harga, nama, deskripsi, viewed, likes, total_komen, action, action_el, photo}: {kategori: string, harga: number, nama: string, deskripsi: string, viewed: number, likes: number, total_komen:number, action: () => void, action_el: HTMLElement | null, photo: string} = $props()
    function formatRupiah(angka: number): string{
        return `Rp${angka.toLocaleString('id-ID')}`
    }

    function formatCompact(n: number): string {
		if (n >= 1_000_000) return `${(n / 1_000_000).toFixed(1)}jt`;
		if (n >= 1_000) return `${(n / 1_000).toFixed(1)}rb`;
		return `${n}`;
	}

    if (action_el !== null ) {action_el.addEventListener("click", action);}
</script>

<div class="w-full sm:w-[23rem] border border-zinc-200 hover:border-zinc-400 rounded-lg bg-white shadow-sm overflow-hidden flex flex-col transition-colors duration-150">
    <div class="grid grid-cols-[30%_70%]">
        <div>

        </div>
        <div class="p-3 flex flex-col gap-2">
						<div class="flex items-start justify-between gap-2">
							<div class="min-w-0">
								<span class="px-1.5 py-0.5 bg-zinc-100 text-zinc-600 text-[9px] font-medium rounded uppercase tracking-wider">
									{kategori}
								</span>
								<h3 class="text-xs font-semibold text-zinc-800 leading-tight truncate mt-1">{nama}</h3>
							</div>
							<span class="text-[11px] font-bold text-zinc-900 font-mono flex-shrink-0">{formatRupiah(harga)}</span>
						</div>
						<p class="text-[10px] text-zinc-400 line-clamp-2 leading-tight">{deskripsi}</p>

						<div class="grid grid-cols-3 gap-2 pt-2 mt-1 border-t border-zinc-100 text-[10px] text-zinc-600 font-mono">
							<div class="flex items-center gap-1">
								<svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-zinc-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
									<path d="M2.062 12.348a1 1 0 0 1 0-.696 10.75 10.75 0 0 1 19.876 0 1 1 0 0 1 0 .696 10.75 10.75 0 0 1-19.876 0"/><circle cx="12" cy="12" r="3"/>
								</svg>
								<span>{viewed}</span>
							</div>
							<div class="flex items-center gap-1">
								<svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-zinc-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
									<path d="M19 14c1.49-1.46 3-3.21 3-5.5A5.5 5.5 0 0 0 16.5 3c-1.76 0-3 .5-4.5 2-1.5-1.5-2.74-2-4.5-2A5.5 5.5 0 0 0 2 8.5c0 2.3 1.5 4.05 3 5.5l7 7Z"/>
								</svg>
								<span>{likes}</span>
							</div>
							<div class="flex items-center gap-1">
								<svg xmlns="http://www.w3.org/2000/svg" class="w-3 h-3 text-zinc-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
									<path d="M7.9 20A9 9 0 1 0 4 16.1L2 22Z"/>
								</svg>
								<span>{total_komen}</span>
							</div>
						</div>

						{action_el ? action_el : ""}
					</div>
    </div>					
</div>