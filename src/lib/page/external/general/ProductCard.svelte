<script lang="ts">
    import { goto } from "$app/navigation";
    import type { Action } from "svelte/action";

    let { id, path, fun }: { id: number; path: string, fun?: () => void } = $props();
    const DetailsAction: Action = (node) =>{
       const handleClick = () => {
            if(fun !== undefined){
                fun()
            }
            goto(path); 
        };

        node.addEventListener("click", handleClick)

         return {
            destroy() {
                node.removeEventListener("click", handleClick)
            }   
        };
    }
</script>

<div use:DetailsAction class="w-[25.5rem] h-[15rem] bg-white rounded-sm border border-zinc-200 hover:border-zinc-800 transition duration-300 overflow-hidden grid grid-cols-2 group product-card-{id}">
    
    <!-- ─── KOLOM KIRI: GAMBAR & BADGE ─── -->
    <div class="relative bg-zinc-100 overflow-hidden flex items-center justify-center">
        <!-- Foto Produk dengan Zoom on Hover (Selaras dengan Seller Card) -->
        <img 
            src="src/constant/hm-removebg-preview.png" 
            alt="Product" 
            class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-500 z-0" 
        />
        
        <!-- Gradient Overlay tipis di bawah untuk memperjelas teks harga -->
        <div class="absolute inset-0 bg-gradient-to-t from-black/40 via-transparent to-transparent pointer-events-none"></div>

        <!-- Tombol Wishlist (Love) -->
        <button 
            onclick={(e) => { e.preventDefault(); e.stopPropagation(); }}
            class="absolute top-2.5 right-2.5 z-10 bg-white/90 hover:bg-white text-zinc-600 hover:text-rose-600 shadow-xs hover:scale-110 transition duration-200 rounded-full flex items-center justify-center w-7 h-7"
            title="Suka"
        >
            <svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24" class="w-3.5 h-3.5">
                <path d="M11.645 20.91l-.007-.003-.022-.012a15.247 15.247 0 01-.383-.218 25.18 25.18 0 01-4.244-3.17C4.688 15.36 2.25 12.174 2.25 8.25 2.25 5.322 4.714 3 7.688 3A5.5 5.5 0 0112 5.052 5.5 5.5 0 0116.313 3c2.973 0 5.437 2.322 5.437 5.25 0 3.925-2.438 7.111-4.739 9.256a25.175 25.175 0 01-4.244 3.17 15.247 15.247 0 01-.383.219l-.022.012-.007.004-.003.001a.752.752 0 01-.704 0l-.003-.001z" />
            </svg>
        </button>

        <!-- Harga Badge di Atas Gambar / Pojok Kiri Bawah Gambar -->
        <div class="absolute bottom-2.5 left-3 z-10">
            <span class="bg-slate-950 text-white text-[11px] font-bold px-2 py-0.5 rounded-xs tracking-tight shadow-xs">
                Rp 150.000
            </span>
        </div>
    </div>

    <!-- ─── KOLOM KANAN: DETAIL TEKS & AKSI ─── -->
    <div class="p-3.5 flex flex-col justify-between bg-white min-w-0">
        
        <!-- Bagian Atas: Nama Seller & Nama Barang -->
        <div class="flex flex-col gap-0.5 min-w-0">
            <div class="text-slate-400 text-[9px] uppercase tracking-[0.15em] font-mono font-bold truncate">
                Mekar Jaya Tech
            </div>
            <h3 class="text-slate-900 font-bold text-sm tracking-tight truncate group-hover:text-black">
                Tas Kulit Premium Minimalis
            </h3>
            <p class="text-xs text-slate-500 font-light leading-relaxed line-clamp-2 mt-0.5">
                Tas kulit premium dengan bahan berkualitas tinggi, desain minimalis modern cocok digunakan untuk berbagai aktivitas harian.
            </p>
        </div>

        <!-- Bagian Tengah: Lokasi & Metrik (Rating, Terjual, Stok) -->
        <div class="flex flex-col gap-2 my-1">
            <!-- Lokasi Toko -->
            <div class="flex items-center gap-1 text-[10px] text-zinc-500 truncate min-w-0">
                <svg class="w-3 h-3 text-slate-800 shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                    <path stroke-linecap="square" stroke-linejoin="miter" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z" />
                    <circle cx="12" cy="11" r="2" fill="currentColor" />
                </svg>
                <span class="truncate font-medium text-slate-600">Kota Bogor</span>
            </div>

            <!-- Metrik: Rating, Terjual, Stok (Desain Border Tipis Konsisten) -->
            <div class="grid grid-cols-3 text-[10px] text-zinc-500 font-medium border-t border-zinc-100 pt-1.5 gap-1">
                <div class="flex items-center gap-0.5">
                    <span class="text-amber-500 font-bold">★</span>
                    <span class="text-slate-800 font-semibold">4.8</span>
                </div>
                <div class="text-center truncate text-zinc-500">
                    2.1k <span class="text-[9px]">Terjual</span>
                </div>
                <div class="text-right truncate text-zinc-500">
                    Stok: <span class="text-slate-800 font-semibold">12</span>
                </div>
            </div>
        </div>

        <!-- Bagian Bawah: Tombol Tambah Keranjang -->
        <div>
            <button 
                onclick={(e) => {
                    e.preventDefault();
                    e.stopPropagation();
                }} 
                class="w-full flex items-center justify-center gap-2 bg-zinc-50 hover:bg-slate-900 text-slate-800 hover:text-white text-[11px] font-bold border border-zinc-200 hover:border-slate-900 py-1.5 px-2 rounded-xs transition-all duration-300 group/btn"
            >
                <span class="tracking-wide uppercase truncate">Tambah Keranjang</span>
                <span class="transition-transform duration-300 group-hover/btn:translate-x-0.5 shrink-0">
                    <svg class="w-3.5 h-3.5" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                        <path fill-rule="evenodd" clip-rule="evenodd" d="M1.56641 4C1.56641 3.58579 1.90219 3.25 2.31641 3.25H3.49696C4.61854 3.25 5.56885 4.07602 5.72504 5.18668L5.7862 5.62161H19.7507C21.3714 5.62161 22.4605 7.28344 21.8137 8.76953L19.1464 14.8979C18.789 15.719 17.9788 16.25 17.0833 16.25L7.72179 16.25C6.60021 16.25 5.6499 15.424 5.49371 14.3133L4.23965 5.39556C4.18759 5.02534 3.87082 4.75 3.49696 4.75H2.31641C1.90219 4.75 1.56641 4.41421 1.56641 4Z" fill="currentColor"/>
                    </svg>
                </span>
            </button>
        </div>

    </div>
</div>