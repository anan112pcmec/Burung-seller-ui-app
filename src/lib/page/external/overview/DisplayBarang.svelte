<script lang="ts">
    import ProductCard from "../general/ProductCard.svelte";

    const ulang = 10;
</script>

<!-- ==========================================
     SECTION: BARANG UNGGULAN SELLER
=========================================== -->
<section id="display-hot-barang" class="w-full bg-white border border-zinc-950/10 flex flex-col">
    
    <!-- Header Section -->
    <div class="flex items-center justify-between border-b border-zinc-950/10 p-4 md:px-6 bg-zinc-50/50">
        <h2 class="font-sans font-bold text-slate-950 text-sm md:text-base tracking-tight">
            Barang Unggulan Seller
        </h2>
        
        <!-- Icon & Metadata -->
        <div class="flex items-center gap-2 font-mono text-[10px] text-zinc-500 uppercase tracking-widest font-bold">
            <!-- Warna SVG sekarang mengikuti warna text container berkat fill="currentColor" -->
            <svg class="w-4 h-4 text-slate-900" viewBox="0 0 25 25" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M8.66699 7.37109C8.66699 6.12845 9.67435 5.12109 10.917 5.12109H14.0837C15.3263 5.12109 16.3337 6.12845 16.3337 7.37109V8.87109H8.66699V7.37109Z" fill="currentColor"/>
                <path d="M7.83398 10.3711C6.59134 10.3711 5.58398 11.3785 5.58398 12.6211V14.1211H11.7507V10.3711H7.83398Z" fill="currentColor"/>
                <path d="M4.75 15.6211C3.50736 15.6211 2.5 16.6285 2.5 17.8711V19.3711C2.5 19.7853 2.83579 20.1211 3.25 20.1211H8.29132V15.6211H4.75Z" fill="currentColor"/>
                <path d="M9.79132 20.1211H15.2083V15.6211H9.79132V20.1211Z" fill="currentColor"/>
                <path d="M16.7083 20.1211H21.7497C22.1639 20.1211 22.4997 19.7853 22.4997 19.3711V17.8711C22.4997 16.6285 21.4923 15.6211 20.2497 15.6211H16.7083V20.1211Z" fill="currentColor"/>
                <path d="M19.4167 14.1211V12.6211C19.4167 11.3785 18.4093 10.3711 17.1667 10.3711H13.2507V14.1211H19.4167Z" fill="currentColor"/>
            </svg>
            <span>20 Barang</span>
        </div>
    </div>

    <!-- Scrollable Horizontal Container -->
    <!-- Menambahkan pb-4 dan pt-6 agar jarak kartu bernapas, shrink-0 ditambahkan dalam loop agar kartu tidak gepeng -->
    <div class="flex gap-4 overflow-x-auto scrollbar-none items-stretch px-4 py-6 md:px-6 w-full">
        {#each Array(ulang) as _, i}
            <div class="shrink-0">
                <ProductCard 
                    id={i} 
                    path={"/details/produk-seller"} 
                    fun={() => {}}
                />
            </div>
        {/each}
    </div>
    
</section>