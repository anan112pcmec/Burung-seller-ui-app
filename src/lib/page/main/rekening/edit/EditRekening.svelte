<script lang="ts">
    // ///////////////////////////////////////////////////////////////////////
    // Types & Constants
    // ///////////////////////////////////////////////////////////////////////

    const DAFTAR_BANK: string[] = [
        'BCA',
        'Mandiri',
        'BNI',
        'BRI',
        'CIMB Niaga',
        'Permata',
        'Danamon',
        'BTN',
        'Bank Syariah Indonesia (BSI)',
        'OCBC NISP'
    ];

    // ///////////////////////////////////////////////////////////////////////
    // State — Rekening Seller
    // ///////////////////////////////////////////////////////////////////////

    let namaBank = $state('BCA');
    let pemilikRekening = $state('Ahmad Fauzan');
    let isDefault = $state(true);

    let nomorRekeningBaru = $state('');
    let konfirmasiNomorRekening = $state('');

	
    // Sanitasi input agar hanya menerima angka
    function handleInputNomor(e: Event) {
        const target = e.target as HTMLInputElement;
        target.value = target.value.replace(/\D/g, '');
        nomorRekeningBaru = target.value;
    }

    function handleInputKonfirmasi(e: Event) {
        const target = e.target as HTMLInputElement;
        target.value = target.value.replace(/\D/g, '');
        konfirmasiNomorRekening = target.value;
    }

    // ///////////////////////////////////////////////////////////////////////
    // Validasi
    // ///////////////////////////////////////////////////////////////////////

    let nomorRekeningDiisi = $derived(nomorRekeningBaru.trim().length >= 8);
    let nomorRekeningCocok = $derived(nomorRekeningDiisi && nomorRekeningBaru === konfirmasiNomorRekening);
    let konfirmasiTidakKosongTapiBeda = $derived(konfirmasiNomorRekening.length > 0 && !nomorRekeningCocok);

    let formValid = $derived(
        namaBank.length > 0 && pemilikRekening.trim().length > 0 && nomorRekeningCocok
    );

    // ///////////////////////////////////////////////////////////////////////
    // Progress / Kelengkapan Form
    // ///////////////////////////////////////////////////////////////////////

    let itemKelengkapan = $derived([
        { label: 'Pilih Bank', selesai: namaBank.length > 0 },
        { label: 'Nama Pemilik Rekening', selesai: pemilikRekening.trim().length > 0 },
        { label: 'Nomor Rekening Baru (min. 8 digit)', selesai: nomorRekeningDiisi },
        { label: 'Konfirmasi Nomor Rekening Cocok', selesai: nomorRekeningCocok },
        { label: 'Pengaturan Rekening Utama (Default)', selesai: isDefault }
    ]);

    let skorPersen = $derived(
        Math.round((itemKelengkapan.filter((i) => i.selesai).length / itemKelengkapan.length) * 100)
    );

    // ///////////////////////////////////////////////////////////////////////
    // Submit Action
    // ///////////////////////////////////////////////////////////////////////

    let statusSimpan = $state<'idle' | 'menyimpan' | 'tersimpan'>('idle');

    function simpanPerubahan() {
        if (!formValid) return;
        statusSimpan = 'menyimpan';
        setTimeout(() => {
            statusSimpan = 'tersimpan';
            nomorRekeningBaru = '';
            konfirmasiNomorRekening = '';
            setTimeout(() => (statusSimpan = 'idle'), 2500);
        }, 700);
    }

	$effect(() =>{
		if (nomorRekeningBaru == ''){
			konfirmasiNomorRekening = ''
		}
	})
</script>

<section id="edit-rekening" class="w-full bg-white p-4 sm:p-6 lg:p-8 text-slate-950 pb-20">
    <!-- HEADER -->
    <div class="flex items-center justify-between flex-wrap gap-3 pb-6 mb-6 border-b border-zinc-800/10">
        <div>
            <span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
                REKENING TOKO / EDIT
            </span>
            <h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
                Edit Rekening
            </h1>
        </div>
        
        <!-- INDICATOR RING SKOR -->
        <div class="flex items-center gap-2">
            <div class="relative w-8 h-8">
                <svg class="w-full h-full -rotate-90" viewBox="0 0 36 36">
                    <circle cx="18" cy="18" r="15.915" fill="none" class="stroke-zinc-100" stroke-width="3.5" />
                    <circle
                        cx="18" cy="18" r="15.915" fill="none"
                        class={skorPersen === 100 ? 'stroke-teal-600' : 'stroke-slate-950'}
                        stroke-width="3.5"
                        stroke-dasharray="{skorPersen} {100 - skorPersen}"
                        stroke-linecap="round"
                    />
                </svg>
                <span class="absolute inset-0 flex items-center justify-center text-[8px] font-bold font-mono">{skorPersen}%</span>
            </div>
            <div class="leading-tight">
                <p class="text-[10px] font-bold text-zinc-800">Form {skorPersen === 100 ? 'Siap Disimpan' : 'Belum Lengkap'}</p>
                <p class="text-[9px] text-zinc-400">Lengkapi data rekening untuk pencairan dana</p>
            </div>
        </div>
    </div>

    <div class="grid grid-cols-1 lg:grid-cols-12 gap-6">
        <!-- KOLOM KIRI — FORM EDIT REKENING -->
        <div class="lg:col-span-7 flex flex-col gap-5">
            <!-- BANNER PENJELASAN -->
            <div class="bg-zinc-50 border border-zinc-200 rounded-md p-3.5 text-[10px] text-zinc-500 leading-relaxed">
                Demi keamanan, nomor rekening yang sebelumnya terdaftar <strong class="text-zinc-700">tidak ditampilkan</strong>
                di halaman ini, sekalipun disamarkan. Jika Anda tidak berniat mengganti nomor rekening, tetap masukkan
                nomor yang sama persis seperti sebelumnya.
            </div>

            <!-- FORM CARD -->
            <div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5 flex flex-col gap-4">
                <label class="flex flex-col gap-1.5">
                    <span class="text-[10px] font-medium text-slate-600">Nama Bank</span>
                    <select
                        bind:value={namaBank}
                        class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 focus:outline-none focus:border-slate-950 transition-colors"
                    >
                        {#each DAFTAR_BANK as bank}
                            <option value={bank}>{bank}</option>
                        {/each}
                    </select>
                </label>

                <label class="flex flex-col gap-1.5">
                    <span class="text-[10px] font-medium text-slate-600">Nama Pemilik Rekening</span>
                    <input
                        type="text"
                        bind:value={pemilikRekening}
                        placeholder="Contoh: Ahmad Fauzan"
                        class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 focus:outline-none focus:border-slate-950 transition-colors"
                    />
                </label>

                <div class="pt-2 border-t border-dashed border-zinc-200">
                    <label class="flex flex-col gap-1.5">
                        <span class="text-[10px] font-medium text-slate-600">Nomor Rekening</span>
                        <input
                            type="text"
                            inputmode="numeric"
                            value={nomorRekeningBaru}
                            oninput={handleInputNomor}
                            placeholder="Masukkan nomor rekening"
                            class="border border-zinc-300 rounded-md px-3 py-2 text-xs font-mono text-slate-900 placeholder-zinc-400 focus:outline-none focus:border-slate-950 transition-colors"
                        />
                    </label>

                    <label class="flex flex-col gap-1.5 mt-3">
                       
                        {#if nomorRekeningBaru !== ''}
							 <span class="text-[10px] font-medium text-slate-600">Konfirmasi Nomor Rekening Baru</span>
							<input
                            type="text"
                            inputmode="numeric"
                            value={konfirmasiNomorRekening}
                            oninput={handleInputKonfirmasi}
                            placeholder="Ketik ulang nomor rekening di atas"
                            class="border rounded-md px-3 py-2 text-xs font-mono text-slate-900 placeholder-zinc-400 focus:outline-none transition-colors {konfirmasiTidakKosongTapiBeda
                                ? 'border-rose-400 focus:border-rose-500'
                                : 'border-zinc-300 focus:border-slate-950'}"
                        />
						{/if}
                        {#if konfirmasiTidakKosongTapiBeda}
                            <span class="text-[9px] text-rose-500">Belum sama dengan nomor rekening di atas.</span>
                        {:else if nomorRekeningCocok}
                            <span class="text-[9px] text-teal-600">Nomor rekening cocok.</span>
                        {/if}
                    </label>
                </div>

                <label class="flex items-center gap-2 text-[10px] text-zinc-600 pt-2 border-t border-dashed border-zinc-200 cursor-pointer">
                    <input type="checkbox" bind:checked={isDefault} class="accent-slate-950 rounded" />
                    Jadikan rekening default untuk pencairan dana
                </label>
            </div>

            <!-- AKSI BUTTONS -->
            <div class="flex items-center gap-3">
                <button
                    type="button"
                    disabled={!formValid || statusSimpan === 'menyimpan'}
                    onclick={simpanPerubahan}
                    class="px-5 py-2.5 bg-slate-950 text-white text-[11px] font-bold uppercase tracking-wider rounded-sm hover:bg-slate-800 transition-colors disabled:opacity-30 disabled:cursor-not-allowed"
                >
                    {statusSimpan === 'menyimpan' ? 'Menyimpan...' : 'Simpan Perubahan'}
                </button>
                <button type="button" class="text-[10px] font-medium text-slate-500 underline hover:text-slate-800">
                    Batal
                </button>
                {#if statusSimpan === 'tersimpan'}
                    <span class="text-[10px] font-medium text-teal-700">Tersimpan ✓</span>
                {/if}
            </div>
        </div>

        <!-- KOLOM KANAN — RIGHT PREVIEW & PROGRESS -->
        <div class="lg:col-span-5">
            <div class="lg:sticky lg:top-6 flex flex-col gap-4">
                <!-- PREVIEW KARTU BANK LIVE (DISESUAIKAN DENGAN KARTU SISTEM ANDA) -->
				<div class="relative w-full aspect-[1.58/1] rounded-xl bg-gradient-to-br from-zinc-900 via-zinc-800 to-zinc-950 p-4 sm:p-5 text-white shadow-xl border border-zinc-700/60 overflow-hidden grid grid-rows-[25%_40%_35%]">
            
    <!-- Texture Pattern Overlay -->
    <div class="absolute inset-0 opacity-20 pointer-events-none z-0">
        <svg width="100%" height="100%" xmlns="http://www.w3.org/2000/svg">
            <defs>
                <pattern id="card-pattern-preview" width="12" height="12" patternUnits="userSpaceOnUse">
                    <circle cx="6" cy="6" r="4" fill="none" stroke="currentColor" stroke-width="0.5"/>
                    <path d="M0 6h12M6 0v12" stroke="currentColor" stroke-width="0.3"/>
                </pattern>
            </defs>
            <rect width="100%" height="100%" fill="url(#card-pattern-preview)"/>
        </svg>
    </div>

    <!-- ROW 1 (TOP): Brand Bank & Contactless Icon -->
    <div class="relative z-10 flex items-center justify-between">
        <span class="text-sm sm:text-base font-black tracking-widest uppercase text-zinc-100 font-mono leading-none">
            {namaBank ? namaBank.slice(0, 10) : 'CARD'}
        </span>
        <svg class="w-5 h-5 sm:w-6 sm:h-6 text-zinc-400 opacity-80" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
            <path d="M8.5 14.5A4 4 0 0 1 8.5 9.5" />
            <path d="M12 17a8 8 0 0 0 0-10" />
            <path d="M15.5 19.5a12 12 0 0 0 0-15" />
        </svg>
    </div>

    <!-- ROW 2 (MIDDLE): Gold Chip EMV & Badge Status -->
    <div class="relative z-10 flex items-center justify-between">
        <!-- Microchip Gold (Standard Card Scale Ratio) -->
        <div class="w-10 h-7 sm:w-11 sm:h-8 bg-gradient-to-br from-amber-300 via-amber-400 to-amber-600 rounded-md border border-amber-500/80 flex items-center justify-center p-[2px] shadow-sm">
            <div class="w-full h-full border-[0.5px] border-amber-800/50 rounded-[3px] grid grid-cols-2 gap-[1px]">
                <div class="border-r border-b border-amber-800/40"></div>
                <div class="border-b border-amber-800/40"></div>
                <div class="border-r border-amber-800/40"></div>
                <div></div>
            </div>
        </div>

        {#if isDefault}
            <span class="text-[10px] sm:text-xs font-mono font-bold uppercase tracking-wider text-teal-300 bg-teal-400/10 px-2 py-0.5 rounded border border-teal-400/30">
                DEFAULT
            </span>
        {:else}
            <span class="text-[10px] sm:text-xs font-mono font-bold uppercase tracking-wider text-amber-400/90 bg-amber-400/10 px-2 py-0.5 rounded border border-amber-400/20">
                PLATINUM
            </span>
        {/if}
    </div>

    <!-- ROW 3 (BOTTOM): Pemilik, Digit Rekening & Logo Network -->
    <div class="relative z-10 flex items-end justify-between">
        <div class="flex flex-col justify-end space-y-0.5">
            <span class="text-[10px] sm:text-xs font-mono text-zinc-400 uppercase tracking-wider truncate max-w-[140px] sm:max-w-[180px]">
                {pemilikRekening || 'NAMA PEMILIK'}
            </span>
            <span class="text-xs sm:text-sm font-mono text-zinc-100 font-semibold tracking-widest">
                •••• {nomorRekeningBaru ? nomorRekeningBaru.slice(-4) : '8888'}
            </span>
        </div>
        
        <!-- Mini Network Logo (Mastercard Style circles) -->
        <div class="flex -space-x-2 items-center pb-0.5">
            <div class="w-5 h-5 sm:w-6 sm:h-6 rounded-full bg-rose-500/90"></div>
            <div class="w-5 h-5 sm:w-6 sm:h-6 rounded-full bg-amber-500/90 mix-blend-screen"></div>
        </div>
    </div>

</div>

                <!-- PROGRESS CHECKLIST -->
                <div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5">
                    <div class="flex items-center justify-between mb-2">
                        <span class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
                            Status Pengisian
                        </span>
                        <span class="text-[10px] font-bold font-mono {skorPersen === 100 ? 'text-teal-600' : 'text-zinc-700'}">
                            {skorPersen}%
                        </span>
                    </div>

                    <div class="w-full h-1.5 bg-zinc-100 rounded-full overflow-hidden mb-4">
                        <div
                            class="h-full rounded-full transition-all duration-300 {skorPersen === 100 ? 'bg-teal-600' : 'bg-slate-950'}"
                            style="width: {skorPersen}%;"
                        ></div>
                    </div>

                    <ul class="flex flex-col gap-2">
                        {#each itemKelengkapan as item}
                            <li class="flex items-center justify-between text-[10px]">
                                <span class={item.selesai ? 'text-zinc-700 line-through opacity-70' : 'text-zinc-500'}>
                                    {item.label}
                                </span>
                                {#if item.selesai}
                                    <span class="text-teal-600 font-bold">✓</span>
                                {:else}
                                    <span class="text-zinc-300">○</span>
                                {/if}
                            </li>
                        {/each}
                    </ul>
                </div>
            </div>
        </div>
    </div>
</section>