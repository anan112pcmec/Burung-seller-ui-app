<script lang="ts">
    // Types untuk Distributor
    type StatusPengajuan = 'Pending' | 'Disetujui' | 'Ditolak' | 'Belum Diajukan';
    type StatusDokumen = 'kosong' | 'mengunggah' | 'terunggah';

    interface DokumenState {
        status: StatusDokumen;
        namaFile: string | null;
    }

    // State Spesifik Distributor
    let form = $state<Record<string, string>>({
        namaPerusahaan: 'PT Sinergi Distribusi Nusantara',
        nib: '1234567890123',
        npwp: '01.234.567.8-901.000'
    });

    let statusPengajuan = $state<StatusPengajuan>('Pending');
    let alasanPenolakan = $state('');

    let dokumen = $state<Record<string, DokumenState>>({
        dokumenIzin: { status: 'terunggah', namaFile: 'izin-distributor-2026.pdf' },
        npwpFoto: { status: 'terunggah', namaFile: 'npwp-scan.jpg' },
        nibFoto: { status: 'kosong', namaFile: null },
        suratKerjasama: { status: 'kosong', namaFile: null }
    });

    const fields = [
        { key: 'namaPerusahaan', label: 'Nama Perusahaan', placeholder: 'PT Sinergi Distribusi Nusantara', required: true },
        { key: 'nib', label: 'Nomor Induk Berusaha (NIB)', placeholder: '1234567890123', required: true },
        { key: 'npwp', label: 'NPWP Perusahaan', placeholder: '01.234.567.8-901.000', required: true }
    ];

    const docs = [
        { key: 'dokumenIzin', label: 'Dokumen Izin Distributor', hint: 'PDF, scan asli, maks 5MB', accept: '.pdf', required: true },
        { key: 'npwpFoto', label: 'Foto NPWP', hint: 'JPG/PNG, jelas & tidak buram', accept: '.jpg,.jpeg,.png', required: true },
        { key: 'nibFoto', label: 'Foto NIB', hint: 'JPG/PNG, jelas & tidak buram', accept: '.jpg,.jpeg,.png', required: true },
        { key: 'suratKerjasama', label: 'Surat Kerjasama Distribusi', hint: 'PDF, ditandatangani kedua pihak', accept: '.pdf', required: false }
    ];

    // Derived States untuk Check & Progress
    let dataFormLengkap = $derived(
        fields.every((f) => !f.required || (form[f.key] && form[f.key].trim() !== ''))
    );

    let dokumenWajibLengkap = $derived(
        docs.filter((d) => d.required).every((d) => dokumen[d.key]?.status === 'terunggah')
    );

    let itemsKelengkapan = $derived([
        { label: 'Informasi Profil Perusahaan', status: dataFormLengkap },
        { label: 'Izin Resmi Distributor', status: dokumen.dokumenIzin?.status === 'terunggah' },
        { label: 'Kelengkapan Foto NIB & NPWP', status: dokumen.nibFoto?.status === 'terunggah' && dokumen.npwpFoto?.status === 'terunggah' },
        { label: 'Surat Perjanjian / Kerjasama', status: dokumen.suratKerjasama?.status === 'terunggah' }
    ]);

    let totalItem = $derived(itemsKelengkapan.length);
    let itemSelesai = $derived(itemsKelengkapan.filter((i) => i.status).length);
    let skorPersen = $derived(Math.round((itemSelesai / totalItem) * 100));

    let isFormReady = $derived(dataFormLengkap && dokumenWajibLengkap);

    // Handlers
    function handleFileChange(e: Event, docKey: string) {
        const target = e.target as HTMLInputElement;
        const file = target.files?.[0];
        if (!file) return;

        dokumen[docKey].status = 'mengunggah';
        dokumen[docKey].namaFile = file.name;

        setTimeout(() => {
            dokumen[docKey].status = 'terunggah';
        }, 900);
    }

    function hapusDokumen(docKey: string) {
        dokumen[docKey].status = 'kosong';
        dokumen[docKey].namaFile = null;
    }

    function statusClass(status: StatusPengajuan): string {
        if (status === 'Disetujui') return 'bg-teal-50 text-teal-700 border-teal-200';
        if (status === 'Ditolak') return 'bg-rose-50 text-rose-600 border-rose-200';
        if (status === 'Pending') return 'bg-zinc-100 text-zinc-600 border-zinc-200';
        return 'bg-zinc-50 text-zinc-400 border-zinc-200';
    }
</script>

<div class="grid grid-cols-1 lg:grid-cols-12 gap-8 items-start p-5">
    <!-- KOLOM KIRI: FORM & UPLOAD (Span 7) -->
    <div class="lg:col-span-7 space-y-6">
        
        <!-- Status Card -->
        <div class="border rounded-sm p-4 sm:p-5 flex flex-col sm:flex-row sm:items-center sm:justify-between gap-3 {statusClass(statusPengajuan)}">
            <div class="flex items-center gap-3">
                <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4 flex-shrink-0" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                    {#if statusPengajuan === 'Disetujui'}
                        <path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><path d="m9 11 3 3L22 4"/>
                    {:else if statusPengajuan === 'Ditolak'}
                        <circle cx="12" cy="12" r="10"/><path d="m15 9-6 6M9 9l6 6"/>
                    {:else}
                        <circle cx="12" cy="12" r="10"/><path d="M12 6v6l4 2"/>
                    {/if}
                </svg>
                <div>
                    <p class="text-xs font-bold uppercase tracking-wider">Status Verifikasi Distributor: {statusPengajuan}</p>
                    {#if statusPengajuan === 'Ditolak' && alasanPenolakan}
                        <p class="text-[11px] mt-0.5 leading-relaxed opacity-90">{alasanPenolakan}</p>
                    {:else if statusPengajuan === 'Pending'}
                        <p class="text-[11px] mt-0.5 leading-relaxed opacity-80">Pengajuan distributor sedang ditinjau oleh tim verifikasi.</p>
                    {:else if statusPengajuan === 'Disetujui'}
                        <p class="text-[11px] mt-0.5 leading-relaxed opacity-80">Badge Resmi Distributor telah aktif.</p>
                    {/if}
                </div>
            </div>
        </div>

        <!-- Section 1: Data Legalitas -->
        <div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5">
            <span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
                01. Data Legalitas Perusahaan Distributor
            </span>

            <div class="mt-4 grid grid-cols-1 sm:grid-cols-2 gap-4">
                {#each fields as field}
                    <label class="flex flex-col gap-1.5">
                        <span class="text-[10px] font-medium text-slate-600 flex items-center justify-between">
                            {field.label}
                            {#if field.required}<span class="text-rose-500">*</span>{/if}
                        </span>
                        <input
                            type="text"
                            bind:value={form[field.key]}
                            placeholder={field.placeholder}
                            class="border border-zinc-300 rounded-md px-3 py-2 text-xs text-slate-900 placeholder-zinc-400 focus:outline-none focus:border-slate-950 transition-colors"
                        />
                    </label>
                {/each}
            </div>
        </div>

        <!-- Section 2: Upload Dokumen -->
        <div class="border border-zinc-800/20 rounded-sm p-4 sm:p-5">
            <span class="text-[9px] sm:text-[10px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">
                02. Dokumen Pendukung Distributor
            </span>

            <div class="mt-4 grid grid-cols-1 sm:grid-cols-2 gap-3">
                {#each docs as doc}
                    {@const state = dokumen[doc.key]}
                    <div class="border border-dashed border-zinc-300 rounded-md p-3 flex items-start gap-3">
                        <div class="w-9 h-9 rounded-md flex items-center justify-center flex-shrink-0 {state.status === 'terunggah' ? 'bg-teal-50' : 'bg-zinc-100'}">
                            {#if state.status === 'terunggah'}
                                <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4 text-teal-600" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                                    <path d="M14.5 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V7.5L14.5 2Z"/><path d="M14 2v6h6"/><path d="m9 15 2 2 4-4"/>
                                </svg>
                            {:else if state.status === 'mengunggah'}
                                <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4 text-zinc-400 animate-spin" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                                    <path d="M21 12a9 9 0 1 1-6.219-8.56"/>
                                </svg>
                            {:else}
                                <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4 text-zinc-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                                    <path d="M12 13v8"/><path d="m8 17 4-4 4 4"/><path d="M20.39 18.39A5 5 0 0 0 18 9h-1.26A8 8 0 1 0 3 16.3"/>
                                </svg>
                            {/if}
                        </div>

                        <div class="flex-1 min-w-0">
                            <p class="text-[11px] font-semibold text-zinc-800 truncate">
                                {doc.label}
                                {#if doc.required}<span class="text-rose-500">*</span>{/if}
                            </p>

                            {#if state.status === 'kosong'}
                                <p class="text-[9px] text-zinc-400 mt-0.5">{doc.hint}</p>
                                <label class="inline-flex items-center gap-1 mt-2 text-[10px] font-medium text-slate-950 border border-zinc-300 rounded px-2 py-1 cursor-pointer hover:bg-zinc-50 transition-colors">
                                    Pilih File
                                    <input type="file" accept={doc.accept} class="hidden" onchange={(e) => handleFileChange(e, doc.key)} />
                                </label>
                            {:else if state.status === 'mengunggah'}
                                <p class="text-[9px] text-zinc-400 mt-0.5 truncate">Mengunggah {state.namaFile}...</p>
                            {:else}
                                <p class="text-[9px] text-teal-700 mt-0.5 truncate font-medium">{state.namaFile}</p>
                                <div class="flex items-center gap-2 mt-1.5">
                                    <label class="text-[9px] font-medium text-slate-600 underline cursor-pointer">
                                        Ganti
                                        <input type="file" accept={doc.accept} class="hidden" onchange={(e) => handleFileChange(e, doc.key)} />
                                    </label>
                                    <button type="button" onclick={() => hapusDokumen(doc.key)} class="text-[9px] font-medium text-rose-500 underline">
                                        Hapus
                                    </button>
                                </div>
                            {/if}
                        </div>
                    </div>
                {/each}
            </div>
        </div>

        <!-- Submit Button -->
        <div class="flex justify-end pt-2">
            <button
                type="button"
                disabled={!isFormReady}
                class="px-6 py-2.5 bg-slate-950 text-white text-[11px] font-bold uppercase tracking-wider rounded-sm hover:bg-slate-800 disabled:opacity-40 disabled:cursor-not-allowed transition-all"
            >
                {statusPengajuan === 'Belum Diajukan' ? 'Ajukan Verifikasi Distributor' : 'Simpan & Ajukan Ulang'}
            </button>
        </div>
    </div>

    <!-- KOLOM KANAN: LIVE PREVIEW & PROGRESS (Span 5 - Sticky) -->
    <div class="lg:col-span-5 lg:sticky lg:top-6 space-y-4">
        
        <!-- Live Card Preview (Official Distributor Badge) -->
        <div class="border border-zinc-800/20 rounded-sm p-4 bg-zinc-900 text-white relative overflow-hidden shadow-sm">
            <div class="absolute top-0 right-0 w-32 h-32 bg-teal-500/10 rounded-full blur-2xl pointer-events-none"></div>

            <div class="flex items-center justify-between border-b border-zinc-800 pb-3 mb-3">
                <div class="flex items-center gap-2">
                    <span class="w-2 h-2 rounded-full {isFormReady ? 'bg-teal-400' : 'bg-amber-400'}"></span>
                    <span class="text-[10px] font-mono tracking-widest text-zinc-400 uppercase">Distributor Verification</span>
                </div>
                <span class="text-[9px] font-mono bg-teal-950/80 text-teal-300 border border-teal-800 px-2 py-0.5 rounded">
                    OFFICIAL
                </span>
            </div>

            <div class="space-y-3">
                <div>
                    <p class="text-[10px] font-mono text-zinc-500 uppercase">Perusahaan Distributor</p>
                    <p class="text-sm font-semibold text-zinc-100 truncate">{form.namaPerusahaan || '—'}</p>
                </div>

                <div class="grid grid-cols-2 gap-2 text-xs font-mono pt-1">
                    <div>
                        <p class="text-[9px] text-zinc-500 uppercase">NIB</p>
                        <p class="text-[11px] text-zinc-200 font-medium truncate">{form.nib || '—'}</p>
                    </div>
                    <div>
                        <p class="text-[9px] text-zinc-500 uppercase">NPWP</p>
                        <p class="text-[11px] text-zinc-200 font-medium truncate">{form.npwp || '—'}</p>
                    </div>
                </div>

                <div class="pt-2 border-t border-zinc-800 flex items-center justify-between text-[10px] font-mono">
                    <span class="text-zinc-500">Izin Distribusi:</span>
                    <span class={dokumen.dokumenIzin?.status === 'terunggah' ? 'text-teal-400 font-bold' : 'text-zinc-500'}>
                        {dokumen.dokumenIzin?.status === 'terunggah' ? 'TERVERIFIKASI' : 'BELUM ADA'}
                    </span>
                </div>
            </div>
        </div>

        <!-- Checklist Kelengkapan & Progress Ring -->
        <div class="border border-zinc-800/20 rounded-sm p-4 bg-white">
            <div class="flex items-center justify-between pb-3 border-b border-zinc-100 mb-3">
                <div>
                    <p class="text-[9px] font-bold tracking-[0.15em] text-slate-950/40 uppercase font-mono">Kelengkapan Berkas</p>
                    <p class="text-xs font-semibold text-slate-800 mt-0.5">{itemSelesai} dari {totalItem} Persyaratan Terpenuhi</p>
                </div>

                <!-- Circular Progress SVG -->
                <div class="relative w-10 h-10 flex items-center justify-center">
                    <svg class="w-full h-full transform -rotate-90" viewBox="0 0 36 36">
                        <path class="text-zinc-200" stroke-width="3" stroke="currentColor" fill="none" d="M18 2.0845 a 15.9155 15.9155 0 0 1 0 31.831 a 15.9155 15.9155 0 0 1 0 -31.831" />
                        <path class={skorPersen === 100 ? 'text-teal-600' : 'text-slate-950'} stroke-dasharray="{skorPersen}, 100" stroke-width="3" stroke-linecap="round" stroke="currentColor" fill="none" d="M18 2.0845 a 15.9155 15.9155 0 0 1 0 31.831 a 15.9155 15.9155 0 0 1 0 -31.831" />
                    </svg>
                    <span class="absolute text-[10px] font-mono font-bold">{skorPersen}%</span>
                </div>
            </div>

            <!-- List Checklist -->
            <ul class="space-y-2">
                {#each itemsKelengkapan as item}
                    <li class="flex items-center justify-between text-[11px]">
                        <span class={item.status ? 'text-slate-800 font-medium' : 'text-zinc-400'}>{item.label}</span>
                        {#if item.status}
                            <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4 text-teal-600" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                                <polyline points="20 6 9 17 4 12"/>
                            </svg>
                        {:else}
                            <span class="w-2 h-2 rounded-full bg-zinc-300"></span>
                        {/if}
                    </li>
                {/each}
            </ul>
        </div>

    </div>
</div>