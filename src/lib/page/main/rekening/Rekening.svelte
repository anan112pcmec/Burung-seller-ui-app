<script lang="ts">
  import { goto } from '$app/navigation';

	// ///////////////////////////////////////////////////////////////////////
	// Types
	// ///////////////////////////////////////////////////////////////////////

	interface RekeningItem {
		idRekeningSeller: number;
		namaBank: string;
		nomorRekening: string;
		pemilikRekening: string;
		isDefault: boolean;
		createdAt: string;
		jumlahBarangTerhubung: number; // count KategoriBarang WHERE id_rekening = ini
	}

	// ///////////////////////////////////////////////////////////////////////
	// Mock data — ganti dengan fetch list RekeningSeller milik seller ini
	// ///////////////////////////////////////////////////////////////////////

	let rekeningList = $state<RekeningItem[]>([
		{
			idRekeningSeller: 1,
			namaBank: 'BCA',
			nomorRekening: '4520019283',
			pemilikRekening: 'Ahmad Fauzan',
			isDefault: true,
			createdAt: '2026-02-10T08:00:00Z',
			jumlahBarangTerhubung: 34
		},
		{
			idRekeningSeller: 2,
			namaBank: 'Mandiri',
			nomorRekening: '1330008812345',
			pemilikRekening: 'Ahmad Fauzan',
			isDefault: false,
			createdAt: '2026-04-22T10:15:00Z',
			jumlahBarangTerhubung: 8
		},
		{
			idRekeningSeller: 3,
			namaBank: 'Bank Syariah Indonesia (BSI)',
			nomorRekening: '7710092345',
			pemilikRekening: 'Ahmad Fauzan',
			isDefault: false,
			createdAt: '2026-07-05T13:40:00Z',
			jumlahBarangTerhubung: 0
		}
	]);

	// ///////////////////////////////////////////////////////////////////////
	// Helpers
	// ///////////////////////////////////////////////////////////////////////

	function maskNorek(nomor: string): string {
		if (nomor.length <= 4) return nomor;
		return `${'•'.repeat(Math.max(nomor.length - 4, 0))}${nomor.slice(-4)}`;
	}

	function formatTanggal(iso: string): string {
		return new Date(iso).toLocaleDateString('id-ID', { day: '2-digit', month: 'short', year: 'numeric' });
	}

	function inisialBank(nama: string): string {
		const kata = nama.replace(/\(.*?\)/g, '').trim().split(/\s+/);
		if (kata.length === 1) return kata[0].slice(0, 3).toUpperCase();
		return kata.map((k) => k[0]).join('').slice(0, 3).toUpperCase();
	}

	// warna monogram bank di-hash dari nama biar konsisten tiap render, tanpa
	// harus nyimpen mapping warna manual per bank
	const paletMonogram = ['bg-teal-600', 'bg-slate-700', 'bg-zinc-700', 'bg-cyan-700', 'bg-indigo-600'];

	function warnaMonogram(nama: string): string {
		let hash = 0;
		for (let i = 0; i < nama.length; i++) hash = (hash + nama.charCodeAt(i)) % paletMonogram.length;
		return paletMonogram[hash];
	}

	function jadikanDefault(id: number) {
		// TODO: panggil SetDefaultRekeningSeller — transactional di backend
		// (unset default lama, set default baru).
		for (const rek of rekeningList) rek.isDefault = rek.idRekeningSeller === id;
	}
</script>

{#snippet CardRekening(rekening: RekeningItem, index: number)}
    <div onclick={() =>{goto("/rekening/details")}} class="relative w-full max-w-[360px] h-[225px] flex flex-col justify-between rounded-xl border border-zinc-300 bg-gradient-to-b from-zinc-50 via-white to-zinc-100/90 p-4 shadow-sm transition-all duration-200 hover:border-zinc-400 hover:shadow-md group overflow-hidden">
        
        <!-- Passbook Top Accent (Garis Hiasan Buku Rekening) -->
        <div class="absolute top-0 left-0 right-0 h-1.5 bg-zinc-800"></div>

        <!-- HEADER: Index, Nama Bank & Status Default -->
        <div class="flex items-center justify-between pt-1 border-b border-zinc-200/80 pb-2.5">
            <div class="flex items-center gap-2">
                <span class="px-2 py-0.5 rounded bg-zinc-200/80 font-mono text-[11px] font-bold text-zinc-700">
                    #{String(index + 1).padStart(2, '0')}
                </span>
                <span class="font-mono text-xs font-bold uppercase tracking-wider text-zinc-800">
                    {rekening.namaBank || 'BANK'}
                </span>
            </div>

            <!-- Badge Status Default -->
            {#if rekening.isDefault}
                <span class="inline-flex items-center gap-1.5 rounded-full bg-emerald-50 px-2.5 py-0.5 text-[10px] font-semibold text-emerald-700 border border-emerald-200">
                    <span class="h-1.5 w-1.5 rounded-full bg-emerald-500"></span>
                    Default
                </span>
            {/if}
        </div>

        <!-- BODY: Info Rekening & Ilustrasi Kartu Fisik Mini -->
        <div class="grid grid-cols-12  items-center my-auto">
            <!-- Sisi Kiri: Teks Detail (Col 7) -->
            <div class="col-span-8 space-y-3">
                <div>
                    <p class="text-[12px] font-semibold uppercase tracking-widest text-zinc-400">Nomor Rekening</p>
                    <p class="font-mono text-lg sm:text-lg font-bold tracking-wider text-zinc-900 leading-tight">
                        {maskNorek(rekening.nomorRekening)}
                    </p>
                </div>
                <div>
                    <p class="text-[12px] font-semibold uppercase tracking-widest text-zinc-400">Pemilik Rekening</p>
                    <p class="text-sm font-bold uppercase tracking-wide text-zinc-800 truncate leading-tight">
                        {rekening.pemilikRekening}
                    </p>
                </div>
            </div>

            <!-- Sisi Kanan: Grafik Kartu ATM/Debit Platinum (Col 5) -->
            <div class="col-span-4 flex justify-end">
                <div class="relative w-full aspect-[1.58/1] rounded-lg bg-gradient-to-br from-zinc-900 via-zinc-800 to-zinc-950 p-2 text-white shadow-md border border-zinc-700/60 flex flex-col justify-between overflow-hidden group-hover:scale-105 transition-transform duration-300">
                    
                    <!-- Texture Pattern Overlay (Efek Motif Batik / Geometric Vector) -->
                    <div class="absolute inset-0 opacity-20 pointer-events-none">
                        <svg width="100%" height="100%" xmlns="http://www.w3.org/2000/svg">
                            <defs>
                                <pattern id="card-pattern" width="12" height="12" patternUnits="userSpaceOnUse">
                                    <circle cx="6" cy="6" r="4" fill="none" stroke="currentColor" stroke-width="0.5"/>
                                    <path d="M0 6h12M6 0v12" stroke="currentColor" stroke-width="0.3"/>
                                </pattern>
                            </defs>
                            <rect width="100%" height="100%" fill="url(#card-pattern)"/>
                        </svg>
                    </div>

                    <!-- Card Top: Brand Bank & Symbol Contactless -->
                    <div class="relative z-10 flex items-center justify-between">
                        <span class="text-[8px] font-extrabold tracking-widest uppercase text-zinc-200 font-mono">
                            {rekening.namaBank ? rekening.namaBank.slice(0, 6) : 'CARD'}
                        </span>
                        <!-- Contactless Icon -->
                        <svg class="w-2.5 h-2.5 text-zinc-400 opacity-80" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
                            <path d="M8.5 14.5A4 4 0 0 1 8.5 9.5" />
                            <path d="M12 17a8 8 0 0 0 0-10" />
                            <path d="M15.5 19.5a12 12 0 0 0 0-15" />
                        </svg>
                    </div>

                    <!-- Card Middle: Gold Chip EMV & Badge Platinum -->
                    <div class="relative z-10 flex items-center justify-between my-0.5">
                        <!-- Microchip Gold -->
                        <div class="w-4 h-3 bg-gradient-to-br from-amber-300 via-amber-400 to-amber-600 rounded-[2px] border border-amber-500/80 flex items-center justify-center p-[1px] shadow-2xs">
                            <div class="w-full h-full border-[0.5px] border-amber-800/40 rounded-[1px] grid grid-cols-2 gap-[1px]">
                                <div class="border-r border-b border-amber-800/30"></div>
                                <div class="border-b border-amber-800/30"></div>
                                <div class="border-r border-amber-800/30"></div>
                                <div></div>
                            </div>
                        </div>
                        <span class="text-[6px] font-mono font-bold uppercase tracking-widest text-amber-400/90 bg-amber-400/10 px-1 py-[1px] rounded border border-amber-400/20">
                            PLATINUM
                        </span>
                    </div>

                    <!-- Card Bottom: Digit Akhir & Logo Dual Circle (Mastercard/GPN Style) -->
                    <div class="relative z-10 flex items-end justify-between">
                        <span class="text-[7px] font-mono text-zinc-300 tracking-tight">
                            •••• {rekening.nomorRekening ? rekening.nomorRekening.slice(-4) : '8888'}
                        </span>
                        <!-- Mini Network Logo -->
                        <div class="flex -space-x-1 items-center">
                            <div class="w-2.5 h-2.5 rounded-full bg-rose-500/90"></div>
                            <div class="w-2.5 h-2.5 rounded-full bg-amber-500/90"></div>
                        </div>
                    </div>

                </div>
            </div>
        </div>

        <!-- FOOTER: Meta Info & Action Button -->
        <div class="pt-2 border-t border-dashed border-zinc-200 flex items-center justify-between">
            <div class="flex gap-4">
                <div>
                    <p class="text-[9px] font-medium uppercase tracking-wider text-zinc-400">Terhubung</p>
                    <p class="font-mono text-xs font-bold text-zinc-700">
                        {rekening.jumlahBarangTerhubung} varian
                    </p>
                </div>
                <div>
                    <p class="text-[9px] font-medium uppercase tracking-wider text-zinc-400">Ditambahkan</p>
                    <p class="font-mono text-[11px] text-zinc-600">
                        {formatTanggal(rekening.createdAt)}
                    </p>
                </div>
            </div>

            {#if !rekening.isDefault}
                <button
                    type="button"
                    onclick={() => jadikanDefault(rekening.idRekeningSeller)}
                    class="px-2.5 py-1 text-[10px] font-semibold text-zinc-700 hover:text-zinc-900 bg-zinc-100 hover:bg-zinc-200 rounded border border-zinc-300 transition-colors cursor-pointer"
                >
                    Set Default
                </button>
            {/if}
              <button
                    type="button"
                    onclick={() =>{goto("/rekening/edit")}}
                    class="px-2.5 py-1 text-[10px] font-semibold text-zinc-700 hover:text-zinc-900 bg-zinc-100 hover:bg-zinc-200 rounded border border-zinc-300 transition-colors cursor-pointer"
                >
                    Edit
                </button>
        </div>
    </div>
{/snippet}

<section id="rekening" class="w-full bg-white p-4 sm:p-6 lg:p-8 text-slate-950">
	<div class="flex items-center justify-between flex-wrap gap-3 pb-5 mb-6 border-b border-zinc-800/10">
		<div>
			<span class="text-[9px] sm:text-[10px] font-bold tracking-[0.18em] text-slate-950/40 uppercase font-mono">
				PENCAIRAN DANA
			</span>
			<h1 class="mt-1 text-xl sm:text-2xl font-bold uppercase tracking-tight leading-none">
				Rekening Toko
			</h1>
		</div>
		<button
			type="button"
			class="text-[10px] font-bold uppercase tracking-wider text-slate-950 border border-zinc-300 rounded px-3 py-1.5 hover:bg-zinc-50 transition-colors"
		>
			+ Tambah Rekening
		</button>
	</div>

	<div class="flex flex-wrap gap-4">
		{#each rekeningList as rekening, i (rekening.idRekeningSeller)}
			{@render CardRekening(rekening, i)}
		{/each}

		{#if rekeningList.length === 0}
			<p class="text-[11px] text-zinc-400 w-full text-center py-10">
				Belum ada rekening terdaftar. Tambahkan minimal satu rekening agar dana penjualan bisa dicairkan.
			</p>
		{/if}
	</div>
</section>