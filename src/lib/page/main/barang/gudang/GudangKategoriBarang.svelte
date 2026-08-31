<script lang="ts">
    // Type definitions
    type StokBreakdown = {
        Pending: number;   // Supply: Barang masuk / verifikasi
        Ready: number;     // Supply: Stok fisik siap jual
        Dipesan: number;   // Demand: Cart / Menunggu Pembayaran
        Diproses: number;  // Demand: Sedang dikemas gudang
        Terjual: number;   // Historical: Selesai ditransaksikan
        Down: number;      // Unavailable: Rusak / karantina
    };

    interface KategoriItem {
        id: number;
        nama: string;
        sku: string;
        stok: StokBreakdown;
        reorder_point: number;
        unit: string;
    }

    interface inPersenBreakdown {
        Pending: number;
        Ready: number;
        Dipesan: number;
        Diproses: number;
        Terjual: number;
        Down: number;
    }

    // Modern Zinc & Muted State Indicator Palette
    const warnaPending = "bg-teal-500";
    const warnaReady = "bg-zinc-800";
    const warnaDipesan = "bg-amber-500";
    const warnaDiproses = "bg-slate-600";
    const warnaTerjual = "bg-zinc-300";
    const warnaDown = "bg-rose-600";

    // Svelte 5 Runes State
    let searchKeyword = $state('');
    let selectedId = $state<number | null>(1);

    // Batch Action Quantities
    let qtyPending = $state(1);
    let qtyRelease = $state(1);
    let qtyAddReady = $state(1);
    let qtyDown = $state(1);

    // Mock Data
    let items = $state<KategoriItem[]>([
        {
            id: 1,
            nama: 'Kategori A - Kaos Polos Cotton 30s',
            sku: 'KTG-KAS-001',
            stok: { Pending: 40, Ready: 120, Dipesan: 90, Diproses: 60, Terjual: 450, Down: 10 },
            reorder_point: 100,
            unit: 'pcs'
        },
        {
            id: 2,
            nama: 'Kategori B - Jaket Hoodie Fleece',
            sku: 'KTG-JKT-002',
            stok: { Pending: 0, Ready: 250, Dipesan: 30, Diproses: 20, Terjual: 180, Down: 5 },
            reorder_point: 50,
            unit: 'pcs'
        },
        {
            id: 3,
            nama: 'Kategori C - Kemeja Casual Slimfit',
            sku: 'KTG-KMJ-003',
            stok: { Pending: 15, Ready: 50, Dipesan: 40, Diproses: 30, Terjual: 210, Down: 20 },
            reorder_point: 80,
            unit: 'pcs'
        }
    ]);

    // Derived State Computations
    let selectedItem = $derived(items.find(i => i.id === selectedId) || items[0]);
    
    let filteredItems = $derived(
        items.filter(i => 
            i.nama.toLowerCase().includes(searchKeyword.toLowerCase()) ||
            i.sku.toLowerCase().includes(searchKeyword.toLowerCase())
        )
    );

    let totalStokAgregat = $derived(
        items.reduce(
            (acc, item) => ({
                Pending: acc.Pending + item.stok.Pending,
                Ready: acc.Ready + item.stok.Ready,
                Dipesan: acc.Dipesan + item.stok.Dipesan,
                Diproses: acc.Diproses + item.stok.Diproses,
                Terjual: acc.Terjual + item.stok.Terjual,
                Down: acc.Down + item.stok.Down
            }),
            { Pending: 0, Ready: 0, Dipesan: 0, Diproses: 0, Terjual: 0, Down: 0 }
        )
    );

    // Helper Calculations
    function getActiveDemand(stok: StokBreakdown): number {
        return stok.Dipesan + stok.Diproses;
    }

    function getNetAvailable(stok: StokBreakdown): number {
        return stok.Ready - getActiveDemand(stok);
    }

    function getStockRatioStatus(stok: StokBreakdown) {
        const net = getNetAvailable(stok);
        if (net < 0) return { label: 'Defisit', color: 'bg-rose-50 text-rose-700 border-rose-200' };
        if (net === 0) return { label: 'Seimbang', color: 'bg-zinc-100 text-zinc-600 border-zinc-200' };
        return { label: 'Surplus', color: 'bg-teal-50 text-teal-700 border-teal-200' };
    }

    function getFullStock(stok: StokBreakdown): inPersenBreakdown {
        const total = stok.Pending + stok.Ready + stok.Dipesan + stok.Diproses + stok.Terjual + stok.Down;

        if (total === 0) {
            return { Pending: 0, Ready: 0, Dipesan: 0, Diproses: 0, Terjual: 0, Down: 0 };
        }

        return {
            Pending: (stok.Pending / total) * 100,
            Ready: (stok.Ready / total) * 100,
            Dipesan: (stok.Dipesan / total) * 100,
            Diproses: (stok.Diproses / total) * 100,
            Terjual: (stok.Terjual / total) * 100,
            Down: (stok.Down / total) * 100
        };
    }

    // Batch Action Handlers
    function handleSubtractPending(item: KategoriItem) {
        const amount = Math.max(1, qtyPending);
        item.stok.Pending = Math.max(0, item.stok.Pending - amount);
    }

    function handleAddPending(item: KategoriItem) {
        const amount = Math.max(1, qtyPending);
        item.stok.Pending += amount;
    }

    function handleAddReadyDirect(item: KategoriItem) {
        const amount = Math.max(1, qtyAddReady);
        item.stok.Ready += amount;
    }

    function handleReleaseToReady(item: KategoriItem) {
        const amount = Math.min(Math.max(1, qtyRelease), item.stok.Pending);
        item.stok.Pending -= amount;
        item.stok.Ready += amount;
    }

    function handleMarkToDown(item: KategoriItem, customAmount?: number) {
        const amount = customAmount !== undefined ? customAmount : Math.min(Math.max(1, qtyDown), item.stok.Ready);
        item.stok.Ready -= amount;
        item.stok.Down += amount;
    }
</script>

<section id="gudang-kategori-barang" class="p-6 bg-zinc-50/60 min-h-screen font-sans text-zinc-900 space-y-6 antialiased">
    <!-- Top Bar & Header -->
    <div class="flex flex-col lg:flex-row lg:items-center justify-between gap-4 border-b border-zinc-200 pb-5">
        <div>
            <div class="flex items-center gap-2">
                <span class="w-2 h-2 rounded-full bg-zinc-900"></span>
                <h1 class="text-base font-bold text-zinc-900 tracking-tight uppercase font-mono">
                    NIKE A1 FORCE <span class="text-zinc-400 font-normal">/ Category Stock Breakdown</span>
                </h1>
            </div>
            <p class="text-xs text-zinc-500 font-mono mt-1">Real-time monitoring mutasi & rasio ketersediaan stok barang.</p>
        </div>
        
        <!-- Quick Stats Overview (Agregat Total 6 State Minimalist Grid) -->
        <div class="grid grid-cols-3 sm:grid-cols-6 gap-2 text-xs font-mono">
            <div class="px-3 py-2 bg-white border border-zinc-200 rounded-md shadow-2xs">
                <div class="flex items-center gap-1.5 mb-1">
                    <span class="w-1.5 h-1.5 rounded-full bg-teal-500"></span>
                    <span class="text-[9px] font-semibold text-zinc-400 uppercase tracking-wider">Pending</span>
                </div>
                <span class="font-extrabold text-zinc-900 text-sm leading-none block">{totalStokAgregat.Pending}</span>
            </div>

            <div class="px-3 py-2 bg-white border border-zinc-200 rounded-md shadow-2xs">
                <div class="flex items-center gap-1.5 mb-1">
                    <span class="w-1.5 h-1.5 rounded-full bg-zinc-800"></span>
                    <span class="text-[9px] font-semibold text-zinc-400 uppercase tracking-wider">Ready</span>
                </div>
                <span class="font-extrabold text-zinc-900 text-sm leading-none block">{totalStokAgregat.Ready}</span>
            </div>

            <div class="px-3 py-2 bg-white border border-zinc-200 rounded-md shadow-2xs">
                <div class="flex items-center gap-1.5 mb-1">
                    <span class="w-1.5 h-1.5 rounded-full bg-amber-500"></span>
                    <span class="text-[9px] font-semibold text-zinc-400 uppercase tracking-wider">Dipesan</span>
                </div>
                <span class="font-extrabold text-zinc-900 text-sm leading-none block">{totalStokAgregat.Dipesan}</span>
            </div>

            <div class="px-3 py-2 bg-white border border-zinc-200 rounded-md shadow-2xs">
                <div class="flex items-center gap-1.5 mb-1">
                    <span class="w-1.5 h-1.5 rounded-full bg-slate-600"></span>
                    <span class="text-[9px] font-semibold text-zinc-400 uppercase tracking-wider">Diproses</span>
                </div>
                <span class="font-extrabold text-zinc-900 text-sm leading-none block">{totalStokAgregat.Diproses}</span>
            </div>

            <div class="px-3 py-2 bg-white border border-zinc-200 rounded-md shadow-2xs">
                <div class="flex items-center gap-1.5 mb-1">
                    <span class="w-1.5 h-1.5 rounded-full bg-zinc-300"></span>
                    <span class="text-[9px] font-semibold text-zinc-400 uppercase tracking-wider">Terjual</span>
                </div>
                <span class="font-extrabold text-zinc-900 text-sm leading-none block">{totalStokAgregat.Terjual}</span>
            </div>

            <div class="px-3 py-2 bg-white border border-zinc-200 rounded-md shadow-2xs">
                <div class="flex items-center gap-1.5 mb-1">
                    <span class="w-1.5 h-1.5 rounded-full bg-rose-600"></span>
                    <span class="text-[9px] font-semibold text-zinc-400 uppercase tracking-wider">Down</span>
                </div>
                <span class="font-extrabold text-zinc-900 text-sm leading-none block">{totalStokAgregat.Down}</span>
            </div>
        </div>
    </div>

    <!-- Main Section Split Layout -->
    <div class="grid grid-cols-1 lg:grid-cols-12 gap-6 items-start">
        
        <!-- Left Side: Item Catalog & Progress Visualizer (Col 7) -->
        <div class="lg:col-span-7 space-y-3">
            <!-- Search Input -->
            <div class="relative">
                <input
                    type="text"
                    bind:value={searchKeyword}
                    placeholder="Cari berdasarkan nama kategori atau SKU..."
                    class="w-full text-xs font-mono bg-white border border-zinc-200 rounded-lg px-3.5 py-2.5 text-zinc-900 placeholder:text-zinc-400 focus:border-zinc-900 focus:outline-none focus:ring-1 focus:ring-zinc-900 shadow-2xs transition-all"
                />
            </div>

            <!-- Item List Cards -->
            <div class="space-y-3">
                {#each filteredItems as item (item.id)}
                    {@const activeDemand = getActiveDemand(item.stok)}
                    {@const netAvailable = getNetAvailable(item.stok)}
                    {@const statusRatio = getStockRatioStatus(item.stok)}
                    {@const demandCoveragePercent = Math.min(Math.round((item.stok.Ready / (activeDemand || 1)) * 100), 100)}
                    {@const stokPercentage = getFullStock(item.stok)}
                    {@const isLowStock = netAvailable <= item.reorder_point}

                    <button
                        type="button"
                        onclick={() => selectedId = item.id}
                        class="w-full text-left bg-white p-3.5 rounded-xl border transition-all cursor-pointer block hover:border-zinc-400 shadow-2xs grid grid-cols-12 gap-3.5 items-center
                        {selectedId === item.id ? 'border-zinc-900 ring-1 ring-zinc-900 bg-zinc-50/30' : 'border-zinc-200'}"
                    >
                        <!-- Image Thumbnail (Col 3) -->
                        <div class="col-span-3 aspect-square bg-zinc-100 rounded-lg border border-zinc-200/80 flex flex-col items-center justify-center p-2 text-center text-zinc-400">
                            <svg class="w-6 h-6 stroke-zinc-400 mb-1" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
                                <path stroke-linecap="round" stroke-linejoin="round" d="m21 7.5-9-5.25-9 5.25m18 0-9 5.25m9-5.25v9l-9 5.25m-9-9v9l9 5.25m0-9v9m0-14.25v9" />
                            </svg>
                            <span class="text-[9px] font-mono leading-tight uppercase font-medium">Foto Barang</span>
                        </div>

                        <!-- Content Details (Col 9) -->
                        <div class="col-span-9 space-y-2.5">
                            <!-- Item Title & Status Badge -->
                            <div class="flex items-start justify-between gap-2">
                                <div>
                                    <div class="flex items-center gap-1.5">
                                        <span class="text-[10px] font-mono px-1.5 py-0.5 rounded bg-zinc-100 text-zinc-600 font-medium">
                                            {item.sku}
                                        </span>
                                        {#if isLowStock}
                                            <span class="text-[9px] font-mono font-semibold text-rose-600 bg-rose-50 border border-rose-200 px-1.5 py-0.5 rounded">
                                                Reorder Warning
                                            </span>
                                        {/if}
                                    </div>
                                    <h3 class="text-xs font-bold text-zinc-900 mt-1">{item.nama}</h3>
                                </div>
                                <span class="text-[10px] font-mono font-bold px-2 py-0.5 rounded border shrink-0 {statusRatio.color}">
                                    {statusRatio.label}
                                </span>
                            </div>

                            <!-- Metrics Breakdown Grid -->
                            <div class="grid grid-cols-3 gap-2 text-xs font-mono bg-zinc-50/80 p-2 rounded-lg border border-zinc-100">
                                <div>
                                    <span class="block text-[9px] text-zinc-400 uppercase font-medium">Ready Supply</span>
                                    <span class="font-bold text-zinc-900">{item.stok.Ready} {item.unit}</span>
                                    {#if item.stok.Pending > 0}
                                        <span class="text-[9px] text-teal-700 block mt-0.5 font-medium">(+{item.stok.Pending} Pending)</span>
                                    {/if}
                                </div>
                                <div>
                                    <span class="block text-[9px] text-zinc-400 uppercase font-medium">Demand Active</span>
                                    <span class="font-bold text-zinc-900">{activeDemand} {item.unit}</span>
                                    <span class="text-[9px] text-zinc-400 block mt-0.5">({item.stok.Dipesan} Psn / {item.stok.Diproses} Prs)</span>
                                </div>
                                <div>
                                    <span class="block text-[9px] text-zinc-400 uppercase font-medium">Net Available</span>
                                    <span class="font-bold {netAvailable < 0 ? 'text-rose-600' : 'text-teal-700'}">
                                        {netAvailable > 0 ? '+' : ''}{netAvailable} {item.unit}
                                    </span>
                                </div>
                            </div>

                            <!-- Stacked Progress Bar -->
                            <div class="space-y-1">
                                <div class="flex justify-between text-[10px] font-mono text-zinc-400">
                                    <span>Distribusi Status Stok</span>
                                    <span class="font-medium text-zinc-500">{demandCoveragePercent}% Covered</span>
                                </div>
                                
                                <div class="w-full h-1.5 bg-zinc-100 rounded-full overflow-hidden flex gap-0.5" title="Distribusi Status Stok">
                                    {#if stokPercentage.Pending > 0}
                                        <div class="{warnaPending} h-full transition-all rounded-xs" style="width: {stokPercentage.Pending}%" title="Pending: {stokPercentage.Pending.toFixed(1)}%"></div>
                                    {/if}
                                    {#if stokPercentage.Ready > 0}
                                        <div class="{warnaReady} h-full transition-all rounded-xs" style="width: {stokPercentage.Ready}%" title="Ready: {stokPercentage.Ready.toFixed(1)}%"></div>
                                    {/if}
                                    {#if stokPercentage.Dipesan > 0}
                                        <div class="{warnaDipesan} h-full transition-all rounded-xs" style="width: {stokPercentage.Dipesan}%" title="Dipesan: {stokPercentage.Dipesan.toFixed(1)}%"></div>
                                    {/if}
                                    {#if stokPercentage.Diproses > 0}
                                        <div class="{warnaDiproses} h-full transition-all rounded-xs" style="width: {stokPercentage.Diproses}%" title="Diproses: {stokPercentage.Diproses.toFixed(1)}%"></div>
                                    {/if}
                                    {#if stokPercentage.Terjual > 0}
                                        <div class="{warnaTerjual} h-full transition-all rounded-xs" style="width: {stokPercentage.Terjual}%" title="Terjual: {stokPercentage.Terjual.toFixed(1)}%"></div>
                                    {/if}
                                    {#if stokPercentage.Down > 0}
                                        <div class="{warnaDown} h-full transition-all rounded-xs" style="width: {stokPercentage.Down}%" title="Down: {stokPercentage.Down.toFixed(1)}%"></div>
                                    {/if}
                                </div>
                            </div>
                        </div>
                    </button>
                {/each}
            </div>
        </div>

        <!-- Right Side: Detailed Control Panel (Col 5 - Restricted Color Palette: Zinc, Slate, White, Red, Teal) -->
        <div class="lg:col-span-5 space-y-4">
            {#if selectedItem}
                {@const activeDemand = getActiveDemand(selectedItem.stok)}
                {@const netAvailable = getNetAvailable(selectedItem.stok)}
                {@const statusRatio = getStockRatioStatus(selectedItem.stok)}

                <div class="bg-white p-5 rounded-xl border border-zinc-200 shadow-2xs space-y-5 sticky top-6">
                    <!-- Panel Header -->
                    <div class="border-b border-zinc-100 pb-3.5">
                        <div class="flex items-center justify-between">
                            <span class="text-[10px] font-mono font-semibold text-zinc-400 uppercase tracking-wider block">Supply Controller</span>
                            <span class="text-[10px] font-mono font-bold px-2 py-0.5 rounded border {statusRatio.color}">
                                {statusRatio.label}
                            </span>
                        </div>
                        <h2 class="text-sm font-bold text-zinc-900 mt-1">{selectedItem.nama}</h2>
                        <span class="text-xs font-mono text-zinc-400">SKU: {selectedItem.sku}</span>
                    </div>

                    <!-- Selected Metric Cards -->
                    <div class="grid grid-cols-2 gap-2 text-xs font-mono">
                        <div class="p-3 bg-zinc-50 border border-zinc-200/80 rounded-lg">
                            <span class="block text-[9px] text-zinc-500 font-bold uppercase tracking-wider">Stok Pending</span>
                            <span class="text-base font-extrabold text-teal-700 mt-0.5 block">{selectedItem.stok.Pending} {selectedItem.unit}</span>
                        </div>
                        <div class="p-3 bg-zinc-50 border border-zinc-200/80 rounded-lg">
                            <span class="block text-[9px] text-zinc-500 font-bold uppercase tracking-wider">Stok Ready</span>
                            <span class="text-base font-extrabold text-zinc-900 mt-0.5 block">{selectedItem.stok.Ready} {selectedItem.unit}</span>
                        </div>
                    </div>

                    <!-- Action Controls -->
                    <div class="space-y-3.5 text-xs font-mono">
                        
                        <!-- Action 1: Pending Management -->
                        <div class="p-3.5 bg-zinc-50/60 border border-zinc-200/80 rounded-xl space-y-2">
                            <div class="flex justify-between items-center">
                                <span class="font-bold text-zinc-800">1. Kelola Pending</span>
                                <span class="text-[9px] text-zinc-400 font-medium">Bebas (+ / -)</span>
                            </div>
                            <div class="flex items-center gap-2">
                                <input 
                                    type="number" 
                                    min="1" 
                                    bind:value={qtyPending}
                                    class="w-16 px-2 py-1.5 bg-white border border-zinc-300 rounded-md font-bold text-center text-zinc-900 focus:outline-none focus:border-zinc-900 shadow-2xs"
                                />
                                <button 
                                    type="button" 
                                    disabled={selectedItem.stok.Pending <= 0}
                                    onclick={() => handleSubtractPending(selectedItem)}
                                    class="flex-1 py-1.5 bg-white border border-zinc-300 text-rose-600 hover:bg-rose-50 disabled:bg-zinc-100 disabled:text-zinc-300 font-bold rounded-md transition-colors text-center cursor-pointer disabled:cursor-not-allowed shadow-2xs"
                                >
                                    - Kurangi
                                </button>
                                <button 
                                    type="button" 
                                    onclick={() => handleAddPending(selectedItem)}
                                    class="flex-1 py-1.5 bg-zinc-900 hover:bg-zinc-800 text-white font-bold rounded-md transition-colors text-center cursor-pointer shadow-2xs"
                                >
                                    + Tambah
                                </button>
                            </div>
                        </div>

                        <!-- Action 2: Supply & Release to Ready -->
                        <div class="p-3.5 bg-zinc-50/60 border border-zinc-200/80 rounded-xl space-y-3">
                            <div class="flex justify-between items-center">
                                <span class="font-bold text-zinc-800">2. Pasok & Rilis Ready</span>
                                <span class="text-[9px] font-semibold text-zinc-400 uppercase">Forward Only</span>
                            </div>
                            
                            <!-- Pasok Ready Direct -->
                            <div class="space-y-1">
                                <span class="text-[10px] text-zinc-500 font-medium">Pasok Stok Direct ke Ready:</span>
                                <div class="flex items-center gap-2">
                                    <input 
                                        type="number" 
                                        min="1" 
                                        bind:value={qtyAddReady}
                                        class="w-16 px-2 py-1.5 bg-white border border-zinc-300 rounded-md font-bold text-center text-zinc-900 focus:outline-none focus:border-zinc-900 shadow-2xs"
                                    />
                                    <button 
                                        type="button" 
                                        onclick={() => handleAddReadyDirect(selectedItem)}
                                        class="flex-1 py-1.5 bg-teal-700 hover:bg-teal-800 text-white font-bold rounded-md transition-colors text-center cursor-pointer shadow-2xs"
                                    >
                                        + Pasok Ready Direct
                                    </button>
                                </div>
                            </div>

                            <!-- Rilis Pending -> Ready -->
                            <div class="space-y-1 pt-2.5 border-t border-zinc-200/60">
                                <span class="text-[10px] text-zinc-500 font-medium">Rilis Stok Pending ke Ready:</span>
                                <div class="flex items-center gap-2">
                                    <input 
                                        type="number" 
                                        min="1" 
                                        max={selectedItem.stok.Pending}
                                        bind:value={qtyRelease}
                                        class="w-16 px-2 py-1.5 bg-white border border-zinc-300 rounded-md font-bold text-center text-zinc-900 focus:outline-none focus:border-zinc-900 shadow-2xs"
                                    />
                                    <button 
                                        type="button" 
                                        disabled={selectedItem.stok.Pending <= 0 || qtyRelease > selectedItem.stok.Pending}
                                        onclick={() => handleReleaseToReady(selectedItem)}
                                        class="flex-1 py-1.5 bg-slate-800 hover:bg-slate-900 disabled:bg-zinc-200 disabled:text-zinc-400 text-white font-bold rounded-md transition-colors text-center cursor-pointer disabled:cursor-not-allowed shadow-2xs"
                                    >
                                        Rilis ke Ready
                                    </button>
                                </div>
                            </div>
                        </div>

                        <!-- Action 3: Down & Quarantine -->
                        <div class="p-3.5 bg-zinc-50/60 border border-zinc-200/80 rounded-xl space-y-2">
                            <div class="flex justify-between items-center">
                                <span class="font-bold text-zinc-800">3. Status Down / Rusak</span>
                                <span class="text-[9px] font-bold text-zinc-400 uppercase">Quarantine</span>
                            </div>
                            <div class="flex items-center gap-2">
                                <input 
                                    type="number" 
                                    min="1" 
                                    max={selectedItem.stok.Ready}
                                    bind:value={qtyDown}
                                    class="w-16 px-2 py-1.5 bg-white border border-zinc-300 rounded-md font-bold text-center text-zinc-900 focus:outline-none focus:border-zinc-900 shadow-2xs"
                                />
                                <button 
                                    type="button" 
                                    disabled={selectedItem.stok.Ready <= 0 || qtyDown > selectedItem.stok.Ready}
                                    onclick={() => handleMarkToDown(selectedItem)}
                                    class="flex-1 py-1.5 bg-slate-800 hover:bg-slate-900 disabled:bg-zinc-200 disabled:text-zinc-400 text-white font-bold rounded-md transition-colors text-center cursor-pointer disabled:cursor-not-allowed shadow-2xs"
                                >
                                    Set Down
                                </button>
                                <button 
                                    type="button" 
                                    disabled={selectedItem.stok.Ready <= 0}
                                    onclick={() => handleMarkToDown(selectedItem, selectedItem.stok.Ready)}
                                    class="py-1.5 px-3 bg-rose-600 hover:bg-rose-700 disabled:bg-zinc-200 disabled:text-zinc-400 text-white font-bold rounded-md transition-colors text-center cursor-pointer disabled:cursor-not-allowed shrink-0 shadow-2xs"
                                    title="Set seluruh stok Ready ke Down"
                                >
                                    Semua ({selectedItem.stok.Ready})
                                </button>
                            </div>
                        </div>

                    </div>

                    <!-- Bottom Summary Indicator -->
                    <div class="bg-zinc-50 p-3 rounded-lg border border-zinc-200/80 space-y-1 text-xs font-mono">
                        <div class="flex justify-between text-zinc-500">
                            <span>Net Available (Ready - Demand):</span>
                            <span class="font-bold text-zinc-900">{netAvailable > 0 ? '+' : ''}{netAvailable} {selectedItem.unit}</span>
                        </div>
                        <div class="flex justify-between text-zinc-500">
                            <span>Reorder Threshold:</span>
                            <span class="font-bold text-zinc-700">{selectedItem.reorder_point} {selectedItem.unit}</span>
                        </div>
                    </div>
                </div>
            {:else}
                <div class="bg-white p-8 rounded-xl border border-zinc-200 text-center text-xs font-mono text-zinc-400 shadow-2xs">
                    Pilih kategori barang untuk mengelola suplai stok.
                </div>
            {/if}
        </div>
    </div>
</section>