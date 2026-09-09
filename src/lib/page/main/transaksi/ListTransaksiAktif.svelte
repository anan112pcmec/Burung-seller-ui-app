<script lang="ts">
    // ///////////////////////////////////////////////////////////////////////
    // Types
    // ///////////////////////////////////////////////////////////////////////

    type TabAktif = 'dipesan' | 'diproses' | 'pengiriman';
    type StatusFotoPacking = 'kosong' | 'mengunggah' | 'terunggah';
    type UrutanSort = 'default' | 'pendapatan-desc' | 'pendapatan-asc';

    interface TransaksiAktif {
        idTransaksi: number;
        kodeOrderSistem: string;
        namaBarang: string;
        namaVarian: string;
        kuantitas: number;
        total: number;
        waktuPesan: string;
        catatanPembeli: string;
    }

    interface FotoPacking {
        id: number;
        namaFile: string;
        status: StatusFotoPacking;
    }

    interface JejakPengiriman {
        idJejakPengiriman: number;
        idPengiriman: number;
        lokasi: string;
        keterangan: string;
        latitude: number;
        longtitude: number;
        createdAt: string;
        fotoUrls?: string[];
    }

    interface Pengiriman {
        idPengiriman: number;
        idTransaksi: number;
        idSeller: number;
        idAlamatGudang: number;
        idAlamatPengguna: number;
        idKurir?: number;
        namaKurir?: string;
        beratBarang: number;
        kendaraanRequired: string;
        jenisPengiriman: string;
        jarakTempuh: string;
        kurirPaid: number;
        status: string;
        createdAt: string;
        updatedAt: string;
    }

    interface TransaksiPengiriman extends TransaksiAktif {
        pengiriman: Pengiriman;
        jejak: JejakPengiriman[];
    }

    // ///////////////////////////////////////////////////////////////////////
    // State — Tab, Filter, Cari & Sort
    // ///////////////////////////////////////////////////////////////////////

    let tabAktif = $state<TabAktif>('dipesan');
    let idTerpilih = $state<number | null>(null);

    // Filter & Sort State
    let kataKunciCari = $state('');
    let opsiSort = $state<UrutanSort>('default');

    // ///////////////////////////////////////////////////////////////////////
    // Mock Data
    // ///////////////////////////////////////////////////////////////////////

    let transaksiDipesan = $state<TransaksiAktif[]>([
        {
            idTransaksi: 5001,
            kodeOrderSistem: 'ORD-20260828-0011',
            namaBarang: 'Kemeja Flanel Kotak-kotak',
            namaVarian: 'Merah, size L',
            kuantitas: 2,
            total: 318_000,
            waktuPesan: '2026-08-28T08:12:00Z',
            catatanPembeli: 'Tolong dibungkus rapi ya kak.'
        },
        {
            idTransaksi: 5002,
            kodeOrderSistem: 'ORD-20260828-0014',
            namaBarang: 'Kaos Polos Combed 30s',
            namaVarian: 'Hitam, size M',
            kuantitas: 5,
            total: 325_000,
            waktuPesan: '2026-08-28T09:40:00Z',
            catatanPembeli: ''
        },
        {
            idTransaksi: 5003,
            kodeOrderSistem: 'ORD-20260828-0022',
            namaBarang: 'Celana Chino Slimfit',
            namaVarian: 'Khaki, size 32',
            kuantitas: 1,
            total: 189_000,
            waktuPesan: '2026-08-28T10:05:00Z',
            catatanPembeli: 'Kalau bisa dikirim hari ini kak.'
        }
    ]);

    let transaksiDiproses = $state<TransaksiAktif[]>([
        {
            idTransaksi: 4990,
            kodeOrderSistem: 'ORD-20260827-0142',
            namaBarang: 'Jaket Bomber Varsity',
            namaVarian: 'Navy, size L',
            kuantitas: 2,
            total: 490_000,
            waktuPesan: '2026-08-27T14:32:00Z',
            catatanPembeli: ''
        },
        {
            idTransaksi: 4988,
            kodeOrderSistem: 'ORD-20260827-0139',
            namaBarang: 'Tumbler Stainless 500ml',
            namaVarian: 'Sage Green',
            kuantitas: 3,
            total: 195_000,
            waktuPesan: '2026-08-27T11:20:00Z',
            catatanPembeli: ''
        }
    ]);

    let transaksiPengiriman = $state<TransaksiPengiriman[]>([
        {
            idTransaksi: 4975,
            kodeOrderSistem: 'ORD-20260826-0089',
            namaBarang: 'Sepatu Sneakers Canvas',
            namaVarian: 'Putih, size 42',
            kuantitas: 1,
            total: 320_000,
            waktuPesan: '2026-08-26T09:15:00Z',
            catatanPembeli: 'Pastikan dus tidak penyok.',
            pengiriman: {
                idPengiriman: 9001,
                idTransaksi: 4975,
                idSeller: 10,
                idAlamatGudang: 1,
                idAlamatPengguna: 88,
                idKurir: 502,
                namaKurir: 'Budi Santoso (Express Direct)',
                beratBarang: 1200,
                kendaraanRequired: 'Motor',
                jenisPengiriman: 'Instant',
                jarakTempuh: '8.4 km',
                kurirPaid: 24000,
                status: 'In Transit',
                createdAt: '2026-08-26T10:30:00Z',
                updatedAt: '2026-08-26T11:15:00Z'
            },
            jejak: [
                {
                    idJejakPengiriman: 1,
                    idPengiriman: 9001,
                    lokasi: 'Hub Penampungan Barat, Jakarta',
                    keterangan: 'Paket dalam perjalanan ke alamat tujuan oleh kurir.',
                    latitude: -6.175392,
                    longtitude: 106.827153,
                    createdAt: '2026-08-26T11:15:00Z',
                    fotoUrls: ['https://placehold.co/300x200/0f172a/ffffff?text=Transit+Hub']
                }
            ]
        }
    ]);

    // ///////////////////////////////////////////////////////////////////////
    // Computations ($derived) — Filter, Search, Sort & Metrics
    // ///////////////////////////////////////////////////////////////////////

    // 1. Ambil data mentah sesuai tab yang aktif
    let daftarBaku = $derived(
        tabAktif === 'dipesan'
            ? transaksiDipesan
            : tabAktif === 'diproses'
                ? transaksiDiproses
                : transaksiPengiriman
    );

    // 2. Filter berdasarkan Pencarian Nama Barang / Kode Order
    let daftarTersaring = $derived(
        daftarBaku.filter((item) => {
            const query = kataKunciCari.trim().toLowerCase();
            if (!query) return true;
            return (
                item.namaBarang.toLowerCase().includes(query) ||
                item.kodeOrderSistem.toLowerCase().includes(query) ||
                item.namaVarian.toLowerCase().includes(query)
            );
        })
    );

    // 3. Sorting berdasarkan Pendapatan
    let daftarAktif = $derived.by(() => {
        const copy = [...daftarTersaring];
        if (opsiSort === 'pendapatan-desc') {
            return copy.sort((a, b) => b.total - a.total);
        } else if (opsiSort === 'pendapatan-asc') {
            return copy.sort((a, b) => a.total - b.total);
        }
        return copy;
    });

    // 4. METRIK REAL-TIME
    let totalTransaksi = $derived(daftarAktif.length);
    let totalKuantitasBarang = $derived(daftarAktif.reduce((acc, curr) => acc + curr.kuantitas, 0));
    let totalPendapatan = $derived(daftarAktif.reduce((acc, curr) => acc + curr.total, 0));

    // Item Terpilih
    let transaksiTerpilih = $derived(
        daftarAktif.find((t) => t.idTransaksi === idTerpilih) ?? null
    );

    let pengirimanTerpilih = $derived(
        tabAktif === 'pengiriman' && transaksiTerpilih
            ? (transaksiTerpilih as TransaksiPengiriman)
            : null
    );

    // ///////////////////////////////////////////////////////////////////////
    // Helper Functions
    // ///////////////////////////////////////////////////////////////////////

    function pilihTab(tab: TabAktif) {
        tabAktif = tab;
        idTerpilih = null;
        resetForm();
    }

    function pilihTransaksi(id: number) {
        idTerpilih = id;
        resetForm();
    }

    function resetForm() {
        catatanTolak = '';
        modeTolak = false;
        jadwalkanOtomatis = false;
        waktuAutoKirim = '';
    }

    function formatRupiah(n: number): string {
        return `Rp${n.toLocaleString('id-ID')}`;
    }

    function formatWaktu(iso: string): string {
        return new Date(iso).toLocaleString('id-ID', {
            day: '2-digit',
            month: 'short',
            hour: '2-digit',
            minute: '2-digit'
        });
    }

    // ///////////////////////////////////////////////////////////////////////
    // Form States & Actions (Tab Dipesan & Diproses)
    // ///////////////////////////////////////////////////////////////////////

    let catatanTolak = $state('');
    let modeTolak = $state(false);
    let jadwalkanOtomatis = $state(false);
    let waktuAutoKirim = $state('');
    let fotoPackingPerTransaksi = $state<Record<number, FotoPacking[]>>({});

    function terimaPesanan() {
        if (!transaksiTerpilih) return;
        transaksiDipesan = transaksiDipesan.filter((t) => t.idTransaksi !== transaksiTerpilih!.idTransaksi);
        idTerpilih = null;
        resetForm();
    }

    function bukaTolak() {
        catatanTolak = '';
        modeTolak = true;
    }

    function konfirmasiTolak() {
        if (!transaksiTerpilih || catatanTolak.trim().length === 0) return;
        transaksiDipesan = transaksiDipesan.filter((t) => t.idTransaksi !== transaksiTerpilih!.idTransaksi);
        idTerpilih = null;
        modeTolak = false;
        catatanTolak = '';
    }

    function fotoUntuk(id: number): FotoPacking[] {
        return fotoPackingPerTransaksi[id] ?? [];
    }

    function tambahFotoPacking(e: Event, idTransaksi: number) {
        const target = e.target as HTMLInputElement;
        const file = target.files?.[0];
        if (!file) return;

        if (!fotoPackingPerTransaksi[idTransaksi]) {
            fotoPackingPerTransaksi[idTransaksi] = [];
        }

        const idFoto = Date.now();
        const baru: FotoPacking = { id: idFoto, namaFile: file.name, status: 'mengunggah' };
        
        fotoPackingPerTransaksi[idTransaksi] = [...fotoPackingPerTransaksi[idTransaksi], baru];

        setTimeout(() => {
            fotoPackingPerTransaksi[idTransaksi] = fotoPackingPerTransaksi[idTransaksi].map((f) =>
                f.id === idFoto ? { ...f, status: 'terunggah' } : f
            );
        }, 800);

        target.value = '';
    }

    function hapusFotoPacking(idTransaksi: number, idFoto: number) {
        if (!fotoPackingPerTransaksi[idTransaksi]) return;
        fotoPackingPerTransaksi[idTransaksi] = fotoPackingPerTransaksi[idTransaksi].filter((f) => f.id !== idFoto);
    }

    let fotoSiapUntukKirim = $derived(
        transaksiTerpilih ? fotoUntuk(transaksiTerpilih.idTransaksi).some((f) => f.status === 'terunggah') : false
    );

    function kirimBarang() {
        if (!transaksiTerpilih || !fotoSiapUntukKirim) return;
        transaksiDiproses = transaksiDiproses.filter((t) => t.idTransaksi !== transaksiTerpilih!.idTransaksi);
        idTerpilih = null;
    }
</script>

<section id="list-transaksi-aktif" class="w-full  text-slate-950">
    <!-- HEADER MAIN -->
    <div class="pb-4 mb-4 border-b border-zinc-200">
        <span class="text-[10px] font-bold tracking-[0.18em] text-slate-500 uppercase font-mono">
            ORDER
        </span>
        <h1 class="mt-1 text-2xl font-bold uppercase tracking-tight leading-none text-slate-900">
            Pusat Pengelolaan Pesanan
        </h1>

        <!-- NAVIGATION TAB -->
        <div class="inline-flex border border-zinc-300 rounded overflow-hidden mt-4 bg-white flex-wrap shadow-xs">
            <button
                type="button"
                onclick={() => pilihTab('dipesan')}
                class="px-4 py-2.5 text-xs font-semibold uppercase tracking-wider transition duration-150 {tabAktif === 'dipesan'
                    ? 'bg-slate-900 text-white'
                    : 'text-slate-600 hover:bg-slate-100'}"
            >
                Pesanan Masuk ({transaksiDipesan.length})
            </button>
            <button
                type="button"
                onclick={() => pilihTab('diproses')}
                class="px-4 py-2.5 text-xs font-semibold uppercase tracking-wider transition duration-150 {tabAktif === 'diproses'
                    ? 'bg-slate-900 text-white'
                    : 'text-slate-600 hover:bg-slate-100'}"
            >
                Sedang Diproses ({transaksiDiproses.length})
            </button>
            <button
                type="button"
                onclick={() => pilihTab('pengiriman')}
                class="px-4 py-2.5 text-xs font-semibold uppercase tracking-wider transition duration-150 {tabAktif === 'pengiriman'
                    ? 'bg-slate-900 text-white'
                    : 'text-slate-600 hover:bg-slate-100'}"
            >
                Dalam Pengiriman ({transaksiPengiriman.length})
            </button>
        </div>
    </div>

    <!-- METRIK & SUMMARY CARD -->
    <div class="grid grid-cols-1 sm:grid-cols-3 gap-3 mb-5">
        <div class="bg-white border border-zinc-200 rounded p-3 shadow-xs">
            <span class="text-[9px] font-bold text-zinc-400 uppercase font-mono tracking-wider">Total Pesanan</span>
            <p class="text-xl font-extrabold text-slate-900 mt-0.5 font-mono">{totalTransaksi} <span class="text-xs font-normal text-zinc-500">Transaksi</span></p>
        </div>
        <div class="bg-white border border-zinc-200 rounded p-3 shadow-xs">
            <span class="text-[9px] font-bold text-zinc-400 uppercase font-mono tracking-wider">Total Item / Barang</span>
            <p class="text-xl font-extrabold text-slate-900 mt-0.5 font-mono">{totalKuantitasBarang} <span class="text-xs font-normal text-zinc-500">Pcs / Unit</span></p>
        </div>
        <div class="bg-white border border-zinc-200 rounded p-3 shadow-xs">
            <span class="text-[9px] font-bold text-zinc-400 uppercase font-mono tracking-wider">Akumulasi Pendapatan</span>
            <p class="text-xl font-extrabold text-slate-600 mt-0.5 font-mono">{formatRupiah(totalPendapatan)}</p>
        </div>
    </div>

    <!-- TOOLBAR: CARI BARANG & SORT PENDAPATAN -->
    <div class="flex flex-col sm:flex-row items-stretch sm:items-center justify-between gap-3 mb-4 bg-white p-3 border border-zinc-200 rounded shadow-xs">
        <!-- Input Cari -->
        <div class="relative flex-1">
            <input
                type="text"
                bind:value={kataKunciCari}
                placeholder="Cari nama barang atau ID transaksi..."
                class="w-full border border-zinc-300 rounded px-3 py-1.5 text-xs text-slate-900 focus:outline-none focus:border-slate-900 transition-colors"
            />
            {#if kataKunciCari}
                <button
                    type="button"
                    onclick={() => (kataKunciCari = '')}
                    class="absolute right-2 top-1/2 -translate-y-1/2 text-xs text-zinc-400 hover:text-zinc-600 font-bold"
                >
                    ✕
                </button>
            {/if}
        </div>

        <!-- Dropdown Sort -->
        <div class="flex items-center gap-2">
            <span class="text-[10px] font-bold text-zinc-500 uppercase font-mono whitespace-nowrap">Urutkan:</span>
            <select
                bind:value={opsiSort}
                class="border border-zinc-300 rounded px-2.5 py-1.5 text-xs text-slate-900 bg-white focus:outline-none focus:border-slate-900"
            >
                <option value="default">Terbaru (Default)</option>
                <option value="pendapatan-desc">Pendapatan Terbesar (Highest)</option>
                <option value="pendapatan-asc">Pendapatan Terkecil (Lowest)</option>
            </select>
        </div>
    </div>

    <!-- GRID UTAMA (LIST & DETAIL PANEL) -->
    <div class="grid grid-cols-1 lg:grid-cols-[60%_40%] xl:grid-cols-[65%_35%] gap-4">
        <!-- LIST TRANSAKSI -->
        <div class="flex flex-col gap-2 h-[14rem] overflow-y-auto scrollbar-none">
            {#each daftarAktif as trx (trx.idTransaksi)}
                <button
                    type="button"
                    onclick={() => pilihTransaksi(trx.idTransaksi)}
                    class="text-left border rounded p-3 transition-all shadow-xs {idTerpilih === trx.idTransaksi
                        ? 'border-slate-900 bg-white ring-2 ring-slate-900/10'
                        : 'border-zinc-200 hover:border-zinc-400 bg-white'}"
                >
                    <div class="flex items-start justify-between gap-3 flex-wrap">
                        <div class="min-w-0 flex-1">
                            <div class="flex items-center gap-2 flex-wrap">
                                <span class="text-[9px] text-zinc-500 font-mono font-bold uppercase">{trx.kodeOrderSistem}</span>
                                <span class="text-[9px] font-semibold px-1.5 py-0.5 rounded uppercase font-mono bg-zinc-100 text-zinc-700">
                                    {trx.kuantitas} Unit
                                </span>
                                {#if tabAktif === 'pengiriman'}
                                    <span class="text-[8px] font-semibold px-1.5 py-0.5 rounded uppercase font-mono bg-slate-900 text-white">
                                        {(trx as TransaksiPengiriman).pengiriman.status}
                                    </span>
                                {/if}
                            </div>
                            <p class="text-xs font-bold text-slate-900 truncate mt-1">{trx.namaBarang}</p>
                            <p class="text-[10px] text-zinc-500 mt-0.5">{trx.namaVarian}</p>
                            {#if trx.catatanPembeli}
                                <p class="text-[10px] text-zinc-500 italic mt-1 truncate bg-amber-50 border-l-2 border-amber-400 px-1.5 py-0.5">
                                    "{trx.catatanPembeli}"
                                </p>
                            {/if}
                        </div>
                        <div class="text-right flex-shrink-0">
                            <p class="text-xs font-extrabold font-mono text-slate-600">{formatRupiah(trx.total)}</p>
                            <p class="text-[9px] text-zinc-400 font-mono mt-0.5">{formatWaktu(trx.waktuPesan)}</p>
                        </div>
                    </div>
                </button>
            {/each}

            {#if daftarAktif.length === 0}
                <div class="bg-white border border-zinc-200 rounded p-8 text-center">
                    <p class="text-xs text-zinc-500">
                        {#if kataKunciCari}
                            Tidak ditemukan pesanan dengan pencarian "<strong>{kataKunciCari}</strong>".
                        {:else}
                            Tidak ada data pesanan pada tab ini.
                        {/if}
                    </p>
                </div>
            {/if}
        </div>

        <!-- DETAIL & PANEL AKSI -->
        <div class="border border-zinc-300 rounded p-4 sm:p-5 h-fit lg:sticky lg:top-4 bg-white shadow-xs">
            {#if !transaksiTerpilih}
                <p class="text-xs text-zinc-400 text-center py-10">
                    Pilih salah satu pesanan di sebelah kiri untuk mengelola.
                </p>
            {:else if tabAktif === 'dipesan'}
                <!-- AKSI: TERIMA / TOLAK -->
                <span class="text-[9px] font-bold tracking-[0.15em] text-slate-400 uppercase font-mono">
                    {transaksiTerpilih.kodeOrderSistem}
                </span>
                <h3 class="text-sm font-bold text-slate-900 mt-1">{transaksiTerpilih.namaBarang}</h3>
                <p class="text-xs text-zinc-500 mt-0.5">{transaksiTerpilih.namaVarian} · <strong class="text-slate-800">{transaksiTerpilih.kuantitas} Unit</strong></p>
                <p class="text-sm font-extrabold font-mono text-slate-600 mt-2">{formatRupiah(transaksiTerpilih.total)}</p>

                {#if !modeTolak}
                    <div class="mt-4 border-t border-zinc-200 pt-4">
                        <label class="flex items-center gap-2 text-xs text-slate-700 mb-3 cursor-pointer">
                            <input type="checkbox" bind:checked={jadwalkanOtomatis} class="accent-slate-900" />
                            Jadwalkan Pengiriman Otomatis
                        </label>
                        {#if jadwalkanOtomatis}
                            <input
                                type="datetime-local"
                                bind:value={waktuAutoKirim}
                                class="w-full border border-zinc-300 rounded px-3 py-1.5 text-xs text-slate-900 focus:outline-none focus:border-slate-900 mb-3"
                            />
                        {/if}

                        <button
                            type="button"
                            onclick={terimaPesanan}
                            class="w-full px-4 py-2.5 bg-slate-900 text-white text-xs font-bold uppercase tracking-wider rounded hover:bg-slate-800 transition-colors"
                        >
                            Terima Pesanan
                        </button>
                        <button
                            type="button"
                            onclick={bukaTolak}
                            class="w-full mt-2 px-4 py-2.5 border border-rose-300 text-rose-600 text-xs font-bold uppercase tracking-wider rounded hover:bg-rose-50 transition-colors"
                        >
                            Tolak Pesanan
                        </button>
                    </div>
                {:else}
                    <div class="mt-4 border-t border-zinc-200 pt-4">
                        <label class="flex flex-col gap-1">
                            <span class="text-xs font-semibold text-slate-700">Alasan Penolakan</span>
                            <textarea
                                bind:value={catatanTolak}
                                rows={3}
                                placeholder="Jelaskan alasan penolakan..."
                                class="border border-zinc-300 rounded p-2 text-xs focus:outline-none focus:border-rose-500 resize-none"
                            ></textarea>
                        </label>
                        <div class="flex items-center gap-3 mt-3">
                            <button
                                type="button"
                                disabled={catatanTolak.trim().length === 0}
                                onclick={konfirmasiTolak}
                                class="px-4 py-2 bg-rose-600 text-white text-xs font-bold uppercase rounded hover:bg-rose-700 disabled:opacity-40"
                            >
                                Konfirmasi Tolak
                            </button>
                            <button type="button" onclick={() => (modeTolak = false)} class="text-xs text-slate-500 underline">
                                Batal
                            </button>
                        </div>
                    </div>
                {/if}
            {:else if tabAktif === 'diproses'}
                <!-- AKSI: FOTO PACKING -->
                <span class="text-[9px] font-bold tracking-[0.15em] text-slate-400 uppercase font-mono">
                    {transaksiTerpilih.kodeOrderSistem}
                </span>
                <h3 class="text-sm font-bold text-slate-900 mt-1">{transaksiTerpilih.namaBarang}</h3>
                <p class="text-xs text-zinc-500 mt-0.5">{transaksiTerpilih.namaVarian} · {transaksiTerpilih.kuantitas} Unit</p>

                <div class="mt-4 border-t border-zinc-200 pt-4">
                    <p class="text-xs text-zinc-600 mb-2 font-medium">Foto Packing Bukti Pengiriman:</p>

                    <div class="flex flex-col gap-2 mb-3">
                        {#each fotoUntuk(transaksiTerpilih.idTransaksi) as foto (foto.id)}
                            <div class="flex items-center justify-between border border-zinc-200 rounded p-2 text-xs">
                                <span class="truncate flex-1 text-zinc-700">{foto.namaFile}</span>
                                <span class="text-[10px] font-bold uppercase px-1.5 py-0.5 rounded {foto.status === 'terunggah' ? 'bg-slate-100 text-slate-800' : 'bg-amber-100 text-amber-800'}">
                                    {foto.status}
                                </span>
                                <button type="button" onclick={() => hapusFotoPacking(transaksiTerpilih!.idTransaksi, foto.id)} class="ml-2 text-rose-500 hover:underline text-[10px]">
                                    Hapus
                                </button>
                            </div>
                        {/each}
                    </div>

                    <label class="inline-flex items-center justify-center text-xs font-bold text-slate-900 border border-zinc-300 rounded px-3 py-2 cursor-pointer hover:bg-zinc-50 w-full mb-3">
                        + Tambah Foto Packing
                        <input type="file" accept="image/*" class="hidden" onchange={(e) => tambahFotoPacking(e, transaksiTerpilih!.idTransaksi)} />
                    </label>

                    <button
                        type="button"
                        disabled={!fotoSiapUntukKirim}
                        onclick={kirimBarang}
                        class="w-full px-4 py-2.5 bg-slate-900 text-white text-xs font-bold uppercase tracking-wider rounded hover:bg-slate-800 disabled:opacity-30"
                    >
                        Proses Kirim
                    </button>
                </div>
			 {:else if pengirimanTerpilih}
    <!-- DETAIL PENGIRIMAN -->
    <div class="space-y-4 max-h-[80vh] overflow-y-auto pr-1">
        <!-- Header Detail -->
        <div class="border-b border-zinc-100 pb-3">
            <span class="text-[9px] font-bold tracking-widest text-zinc-400 uppercase font-mono block">
                MONITORING PENGIRIMAN
            </span>
            <h2 class="text-xs font-bold text-slate-900 mt-1">{pengirimanTerpilih.kodeOrderSistem}</h2>
            <p class="text-[10px] text-zinc-500 mt-0.5">
                {pengirimanTerpilih.namaBarang} 
                <span class="font-semibold text-zinc-700">({pengirimanTerpilih.kuantitas} unit)</span>
            </p>
        </div>

        <!-- Ringkasan Ekspedisi & Status -->
        <div class="bg-slate-50/80 border border-zinc-200 rounded p-3 text-[10px] space-y-2">
            <div class="flex items-center justify-between border-b border-zinc-200/80 pb-2">
                <span class="text-zinc-400 uppercase font-mono text-[9px] font-semibold">Status Kirim</span>
                <span class="font-bold font-mono text-xs text-slate-900 px-2 py-0.5 bg-white border border-zinc-200 rounded uppercase">
                    {pengirimanTerpilih.pengiriman.status}
                </span>
            </div>
            
            <div class="flex items-center justify-between pt-0.5">
                <span class="text-zinc-500">Layanan Kurir</span>
                <span class="font-semibold text-zinc-800">
                    {pengirimanTerpilih.pengiriman.jenisPengiriman} ({pengirimanTerpilih.pengiriman.kendaraanRequired})
                </span>
            </div>

            {#if pengirimanTerpilih.pengiriman.namaKurir}
                <div class="flex items-center justify-between">
                    <span class="text-zinc-500">Kurir</span>
                    <span class="font-semibold text-zinc-800">{pengirimanTerpilih.pengiriman.namaKurir}</span>
                </div>
            {/if}

            <div class="flex items-center justify-between">
                <span class="text-zinc-500">Jarak / Berat</span>
                <span class="font-mono text-zinc-800">
                    {pengirimanTerpilih.pengiriman.jarakTempuh} / {pengirimanTerpilih.pengiriman.beratBarang}g
                </span>
            </div>

            <div class="flex items-center justify-between pt-1 border-t border-zinc-200/60">
                <span class="text-zinc-500">Ongkir Kurir</span>
                <span class="font-mono font-bold text-slate-900">
                    {formatRupiah(pengirimanTerpilih.pengiriman.kurirPaid)}
                </span>
            </div>
        </div>

        <!-- Timeline / Jejak Pengiriman -->
        <div class="pt-1">
            <span class="text-[9px] font-bold tracking-widest text-zinc-400 uppercase font-mono block mb-3">
                JEJAK PELAKSANAAN LOGISTIK ({pengirimanTerpilih.jejak.length})
            </span>

            {#if pengirimanTerpilih.jejak.length === 0}
                <div class="p-3 bg-zinc-50 border border-dashed border-zinc-200 rounded text-center">
                    <p class="text-[10px] text-zinc-400 italic">Belum ada pembaruan jejak lokasi dari kurir.</p>
                </div>
            {:else}
                <div class="relative pl-4 border-l border-zinc-200 space-y-4 ml-1">
                    {#each pengirimanTerpilih.jejak as j, index (j.idJejakPengiriman)}
                        <div class="relative">
                            <!-- Dot Indikator -->
                            <div class="absolute -left-[21px] top-1 w-2.5 h-2.5 rounded-full border-2 border-white shadow-xs {index === 0 ? 'bg-slate-900 ring-2 ring-slate-900/20' : 'bg-zinc-300'}"></div>
                            
                            <div>
                                <p class="text-[10px] font-bold text-zinc-900 leading-tight">
                                    {j.lokasi || 'Lokasi tidak disebutkan'}
                                </p>
                                <p class="text-[10px] text-zinc-600 mt-0.5 leading-relaxed">
                                    {j.keterangan}
                                </p>
                                <p class="text-[9px] text-zinc-400 font-mono mt-1">
                                    {formatWaktu(j.createdAt)}
                                </p>

                                <!-- Foto Bukti Jejak Pengiriman jika ada -->
                                {#if j.fotoUrls && j.fotoUrls.length > 0}
                                    <div class="flex items-center gap-1.5 mt-2">
                                        {#each j.fotoUrls as imgUrl}
                                            <a 
                                                href={imgUrl} 
                                                target="_blank" 
                                                rel="noreferrer" 
                                                class="block w-12 h-12 rounded border border-zinc-200 overflow-hidden hover:opacity-80 transition-opacity bg-zinc-100"
                                            >
                                                <img src={imgUrl} alt="Bukti Jejak" class="w-full h-full object-cover" />
                                            </a>
                                        {/each}
                                    </div>
                                {/if}
                            </div>
                        </div>
                    {/each}
                </div>
            {/if}
        </div>
    </div>
{/if}
        </div>
    </div>
</section>