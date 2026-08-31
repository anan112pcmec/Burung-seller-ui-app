<script lang="ts">
    let isAgreed = $state(false);
</script>

<div class="w-full space-y-3 font-sans">
    <!-- Baris Centang Persetujuan (Tengah) -->
    <div class="flex items-center justify-center py-1">
        <label class="flex items-center gap-2.5 text-xs font-medium text-zinc-600 cursor-pointer select-none hover:text-zinc-900 transition-colors">
            <input 
                type="checkbox" 
                bind:checked={isAgreed}
                class="w-4 h-4 rounded border-zinc-300 text-zinc-900 focus:ring-zinc-900 cursor-pointer accent-zinc-900"
            />
            <span>Saya telah memeriksa dan menyetujui seluruh <span class="font-bold underline text-zinc-800">Syarat & Ketentuan</span> rilis barang ini.</span>
        </label>
    </div>

    <!-- Section Button Utama -->
    <section id="tambah-upload" class="w-full grid grid-cols-3 gap-3 h-[6rem] p-4 bg-white border-t border-zinc-200 mb-10">
        <!-- Ketentuan -->
        <button
            type="button"
            class="border border-zinc-200 hover:border-zinc-300 rounded-lg bg-zinc-50/60 hover:bg-zinc-100/80 p-3 flex flex-col items-center justify-center gap-1.5 transition-colors duration-150 cursor-pointer group"
        >
            <div class="w-7 h-7 rounded-md bg-zinc-200/70 group-hover:bg-zinc-300/70 text-zinc-700 flex items-center justify-center transition-colors">
                <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                    <path d="M14.5 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V7.5L14.5 2z"/>
                    <polyline points="14 2 14 8 20 8"/>
                    <line x1="16" x2="8" y1="13" y2="13"/>
                    <line x1="16" x2="8" y1="17" y2="17"/>
                </svg>
            </div>
            <div class="text-center leading-none">
                <span class="block text-xs font-bold text-zinc-800">Ketentuan</span>
                <span class="block text-[10px] text-zinc-400 mt-1">Syarat & panduan rilis</span>
            </div>
        </button>

        <!-- FAQ -->
        <button
            type="button"
            class="h-full border border-zinc-200 hover:border-zinc-300 rounded-lg bg-zinc-50/60 hover:bg-zinc-100/80 p-3 flex flex-col items-center justify-center gap-1.5 transition-colors duration-150 cursor-pointer group"
        >
            <div class="w-7 h-7 rounded-md bg-zinc-200/70 group-hover:bg-zinc-300/70 text-zinc-700 flex items-center justify-center transition-colors">
                <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                    <circle cx="12" cy="12" r="10"/>
                    <path d="M9.09 9a3 3 0 0 1 5.83 1c0 2-3 3-3 3"/>
                    <line x1="12" x2="12.01" y1="17" y2="17"/>
                </svg>
            </div>
            <div class="text-center leading-none">
                <span class="block text-xs font-bold text-zinc-800">FAQ</span>
                <span class="block text-[10px] text-zinc-400 mt-1">Bantuan & tanya jawab</span>
            </div>
        </button>

        <!-- Button Rilis -->
        <button
            type="button"
            disabled={!isAgreed}
            class="h-full w-full bg-zinc-900 hover:bg-zinc-800 disabled:bg-zinc-100 disabled:text-zinc-400 disabled:border disabled:border-zinc-200 disabled:cursor-not-allowed text-white font-bold rounded-lg p-3 flex flex-col items-center justify-center gap-1 transition-all duration-150 cursor-pointer shadow-sm active:scale-[0.99]"
        >
            <div class="flex items-center justify-center gap-2">
                <span class="text-xs uppercase tracking-wider">Rilis Barang Induk</span>
                <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
                    <path d="M5 12h14"/>
                    <path d="m12 5 7 7-7 7"/>
                </svg>
            </div>
            <span class="text-[9px] font-normal text-zinc-400 group-hover:text-zinc-300">
                {isAgreed ? 'Siap dipublikasikan' : 'Centang persetujuan dahulu'}
            </span>
        </button>
    </section>
</div>