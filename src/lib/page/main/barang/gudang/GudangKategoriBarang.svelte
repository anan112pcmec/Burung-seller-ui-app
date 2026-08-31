<script lang="ts">
    // State lokal Svelte 5
    let searchKeyword = $state('');
    let selectedId = $state<number | null>(1);

    // Mock Data Supply & Demand per Kategori Barang
    let items = $state([
        {
            id: 1,
            nama: 'Kategori A - Kaos Polos Cotton 30s',
            sku: 'KTG-KAS-001',
            stok_gudang: 120,       // Current Supply
            supply_incoming: 50,    // In Transit Supply
            total_demand: 210,      // Total Active Demand (Reserved/Orders)
            reorder_point: 100,
            unit: 'pcs'
        },
        {
            id: 2,
            nama: 'Kategori B - Jaket Hoodie Fleece',
            sku: 'KTG-JKT-002',
            stok_gudang: 300,
            supply_incoming: 0,
            total_demand: 80,
            reorder_point: 50,
            unit: 'pcs'
        },
        {
            id: 3,
            nama: 'Kategori C - Kemeja Casual Slimfit',
            sku: 'KTG-KMJ-003',
            stok_gudang: 75,
            supply_incoming: 25,
            total_demand: 100,
            reorder_point: 80,
            unit: 'pcs'
        }
    ]);

    // Derived states untuk kalkulasi agregat
    let selectedItem = $derived(items.find(i => i.id === selectedId) || items[0]);
    
    let filteredItems = $derived(
        items.filter(i => 
            i.nama.toLowerCase().includes(searchKeyword.toLowerCase()) ||
            i.sku.toLowerCase().includes(searchKeyword.toLowerCase())
        )
    );

    // Helper fungsi status ratio
    function getStatus(supply: number, incoming: number, demand: number) {
        const totalSupply = supply + incoming;
        const diff = totalSupply - demand;
        if (diff < 0) return { label: 'Defisit', color: 'bg-red-100 text-red-700 border-red-200' };
        if (diff === 0) return { label: 'Seimbang', color: 'bg-zinc-100 text-zinc-700 border-zinc-200' };
        return { label: 'Surplus', color: 'bg-emerald-100 text-emerald-700 border-emerald-200' };
    }
</script>

<section id="gudang-kategori-barang" class="p-6 bg-zinc-50 min-h-screen font-sans space-y-6">
    <!-- Header Section -->
    <div class="flex flex-col md:flex-row md:items-center justify-between gap-4 bg-white p-5 rounded-lg border border-zinc-200 shadow-sm">
        <div>
            <h1 class="text-xl font-bold text-zinc-900 tracking-tight">Interaksi Supply & Demand Gudang</h1>
            <p class="text-xs text-zinc-500 font-mono mt-0.5">Pantau dan kelola rasio ketersediaan stok terhadap alokasi permintaan barang.</p>
        </div>
        
        <!-- Quick Stats Overview -->
        <div class="flex items-center gap-4 text-xs font-mono">
            <div class="px-3 py-2 bg-zinc-50 rounded border border-zinc-200">
                <span class="block text-[10px] text-zinc-400 uppercase">Total Items</span>
                <span class="font-bold text-zinc-900 text-base">{items.length} Kategori</span>
            </div>
            <div class="px-3 py-2 bg-red-50 rounded border border-red-100">
                <span class="block text-[10px] text-red-500 uppercase">Item Defisit</span>
                <span class="font-bold text-red-700 text-base">
                    {items.filter(i => (i.stok_gudang + i.supply_incoming) < i.total_demand).length}
                </span>
            </div>
        </div>
    </div>

    <!-- Main Content Layout -->
    <div class="grid grid-cols-1 lg:grid-cols-12 gap-6">
        
        <!-- Left Side: List Kategori & Supply-Demand Bar (Col 7) -->
        <div class="lg:col-span-7 space-y-4">
            <!-- Search & Filter Bar -->
            <div class="bg-white p-3 rounded-lg border border-zinc-200 flex items-center gap-3">
                <input
                    type="text"
                    bind:value={searchKeyword}
                    placeholder="Cari berdasarkan nama kategori atau SKU..."
                    class="w-full text-xs font-mono bg-zinc-50 border border-zinc-200 rounded px-3 py-2 focus:bg-white focus:border-zinc-900 focus:outline-none"
                />
            </div>

            <!-- List Items -->
            <div class="space-y-3">
                {#each filteredItems as item (item.id)}
                    {@const totalSupply = item.stok_gudang + item.supply_incoming}
                    {@const status = getStatus(item.stok_gudang, item.supply_incoming, item.total_demand)}
                    {@const demandPercent = Math.min(Math.round((item.total_demand / (totalSupply || 1)) * 100), 100)}

                    <button
                        type="button"
                        onclick={() => selectedId = item.id}
                        class="w-full text-left bg-white p-4 rounded-lg border transition-all cursor-pointer block hover:border-zinc-400
                        {selectedId === item.id ? 'border-zinc-900 ring-1 ring-zinc-900 shadow-sm' : 'border-zinc-200'}"
                    >
                        <!-- Item Header -->
                        <div class="flex items-start justify-between mb-3">
                            <div>
                                <span class="text-[10px] font-mono px-1.5 py-0.5 rounded bg-zinc-100 text-zinc-600 border border-zinc-200">
                                    {item.sku}
                                </span>
                                <h3 class="text-sm font-bold text-zinc-900 mt-1">{item.nama}</h3>
                            </div>
                            <span class="text-[10px] font-mono font-semibold px-2 py-0.5 rounded border {status.color}">
                                {status.label}
                            </span>
                        </div>

                        <!-- Supply vs Demand Metrics -->
                        <div class="grid grid-cols-3 gap-2 text-xs font-mono mb-3 bg-zinc-50 p-2.5 rounded border border-zinc-100">
                            <div>
                                <span class="block text-[9px] text-zinc-400 uppercase">Total Supply</span>
                                <span class="font-bold text-zinc-900">{totalSupply} {item.unit}</span>
                                {#if item.supply_incoming > 0}
                                    <span class="text-[9px] text-zinc-500 block">(+{item.supply_incoming} transit)</span>
                                {/if}
                            </div>
                            <div>
                                <span class="block text-[9px] text-zinc-400 uppercase">Total Demand</span>
                                <span class="font-bold text-zinc-900">{item.total_demand} {item.unit}</span>
                            </div>
                            <div>
                                <span class="block text-[9px] text-zinc-400 uppercase">Net Balance</span>
                                <span class="font-bold {totalSupply - item.total_demand < 0 ? 'text-red-600' : 'text-zinc-900'}">
                                    {totalSupply - item.total_demand > 0 ? '+' : ''}{totalSupply - item.total_demand} {item.unit}
                                </span>
                            </div>
                        </div>

                        <!-- Progress Bar Visualizer -->
                        <div class="space-y-1">
                            <div class="flex justify-between text-[10px] font-mono text-zinc-400">
                                <span>Fulfillment Ratio</span>
                                <span>{demandPercent}% Demand Covered</span>
                            </div>
                            <div class="w-full h-2 bg-zinc-100 rounded-full overflow-hidden flex">
                                <!-- Supply Portion -->
                                <div 
                                    class="h-full transition-all duration-300 {totalSupply < item.total_demand ? 'bg-red-500' : 'bg-zinc-900'}"
                                    style="width: {Math.min((totalSupply / (item.total_demand || 1)) * 100, 100)}%"
                                ></div>
                            </div>
                        </div>
                    </button>
                {/each}
            </div>
        </div>

        <!-- Right Side: Interactive Management & Action Panel (Col 5) -->
        <div class="lg:col-span-5 space-y-4">
            {#if selectedItem}
                {@const totalSupply = selectedItem.stok_gudang + selectedItem.supply_incoming}
                {@const status = getStatus(selectedItem.stok_gudang, selectedItem.supply_incoming, selectedItem.total_demand)}

                <div class="bg-white p-5 rounded-lg border border-zinc-200 shadow-sm space-y-5 sticky top-6">
                    <div>
                        <span class="text-[10px] font-mono text-zinc-400 uppercase tracking-wider block">Panel Aksi Supply/Demand</span>
                        <h2 class="text-base font-bold text-zinc-900">{selectedItem.nama}</h2>
                        <span class="text-xs font-mono text-zinc-500">SKU: {selectedItem.sku}</span>
                    </div>

                    <!-- Visual Interaction Matrix -->
                    <div class="border-t border-b border-zinc-100 py-4 space-y-3">
                        <span class="text-[10px] font-mono text-zinc-400 uppercase tracking-wider block">Rincian Komponen</span>
                        
                        <!-- Ready Supply Input/Adjust -->
                        <div class="flex items-center justify-between text-xs font-mono">
                            <span class="text-zinc-600">Stok Gudang (Ready):</span>
                            <div class="flex items-center gap-2">
                                <span class="font-bold text-zinc-900">{selectedItem.stok_gudang} {selectedItem.unit}</span>
                            </div>
                        </div>

                        <!-- Supply in transit -->
                        <div class="flex items-center justify-between text-xs font-mono">
                            <span class="text-zinc-600">Supply dalam Transit:</span>
                            <span class="font-bold text-zinc-900">+{selectedItem.supply_incoming} {selectedItem.unit}</span>
                        </div>

                        <!-- Demand -->
                        <div class="flex items-center justify-between text-xs font-mono">
                            <span class="text-zinc-600">Permintaan Aktif:</span>
                            <span class="font-bold text-red-600">-{selectedItem.total_demand} {selectedItem.unit}</span>
                        </div>

                        <!-- Resulting Net Status -->
                        <div class="flex items-center justify-between text-xs font-mono pt-2 border-t border-zinc-100">
                            <span class="font-semibold text-zinc-700">Status Stok:</span>
                            <span class="font-bold px-2 py-0.5 rounded border text-[11px] {status.color}">
                                {status.label} ({totalSupply - selectedItem.total_demand} {selectedItem.unit})
                            </span>
                        </div>
                    </div>

                    <!-- Quick Simulation Form -->
                    <div class="space-y-3">
                        <span class="text-[10px] font-mono text-zinc-400 uppercase tracking-wider block">Tindakan Cepat</span>

                        <div class="grid grid-cols-2 gap-2">
                            <button
                                type="button"
                                class="w-full py-2 px-3 bg-zinc-900 hover:bg-zinc-800 text-white font-mono text-xs font-bold rounded transition-colors text-center"
                            >
                                + Tambah Supply
                            </button>
                            <button
                                type="button"
                                class="w-full py-2 px-3 bg-white border border-zinc-300 hover:border-zinc-900 text-zinc-800 font-mono text-xs font-bold rounded transition-colors text-center"
                            >
                                Alokasi Demand
                            </button>
                        </div>
                    </div>

                    <!-- Audit info -->
                    <div class="pt-2 text-[10px] font-mono text-zinc-400">
                        * Reorder threshold diset pada: <strong class="text-zinc-700">{selectedItem.reorder_point} {selectedItem.unit}</strong>
                    </div>
                </div>
            {:else}
                <div class="bg-white p-8 rounded-lg border border-zinc-200 text-center text-xs font-mono text-zinc-400">
                    Pilih kategori barang untuk melihat detail interaksi supply & demand.
                </div>
            {/if}
        </div>
    </div>
</section>