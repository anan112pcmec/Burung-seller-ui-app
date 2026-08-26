<script lang="ts">
  const items = [
    { ml: "ml-20", baseColor: "text-zinc-400", font: "font-medium tracking-tight", text: "klik saja, toko langsung laris" },
    { ml: "ml-18", baseColor: "text-zinc-500", font: "font-medium tracking-tight", text: "klik, produk langsung terpajang" },
    { ml: "ml-16", baseColor: "text-zinc-500", font: "font-medium tracking-tight", text: "klik, pesanan langsung terproses" },
    { ml: "ml-14", baseColor: "text-zinc-600", font: "font-medium tracking-tight", text: "klik saja, produk langsung tayang" },
    { ml: "ml-12", baseColor: "text-zinc-600", font: "font-medium tracking-tight", text: "klik saja, pembeli langsung masuk" },
    { ml: "ml-10", baseColor: "text-zinc-700", font: "font-medium tracking-tight", text: "klik sekali, toko jadi kian ramai" },
    { ml: "ml-8",  baseColor: "text-zinc-700", font: "font-medium tracking-tight", text: "klik saja, toko langsung dibanjiri" },
    { ml: "ml-6",  baseColor: "text-zinc-800", font: "font-semibold tracking-tight", text: "klik saja, produk langsung tersebar" },
    { ml: "ml-4",  baseColor: "text-zinc-900", font: "font-semibold tracking-tight", text: "klik saja, pesanan langsung terkirim" },
    { ml: "ml-2.5", baseColor: "text-zinc-900", font: "font-semibold tracking-tighter", text: "klik saja, bisnis langsung terangkat" },
    { ml: "ml-1", baseColor: "text-zinc-900", font: "font-black tracking-tighter", text: "klik saja, transaksi langsung selesai" }
  ];

  let hoveredIndex: number | null = $state<null | number>(null);
</script>

<section id="point-mitra" class="relative px-8 pt-10 h-[30rem] bg-gradient-to-b from-slate-950 to-zinc-900 text-white flex items-center overflow-hidden">
  <div class="grid grid-cols-[45%_17%_38%] w-full h-full">
    <div class="grid grid-rows-[40%_60%] w-full ">
      <div class="align-bottom">
          <h1 class="text-4xl md:text-[112px] font-extrabold tracking-tight text-end leading-tight">
            <span class="text-transparent bg-clip-text bg-gradient-to-r from-zinc-400 to-zinc-100">RASAKAN</span>
          </h1>
      </div>

      <!-- Card UI iOS / macOS Style -->
      <div class="w-full h-full px-2">
        
      </div>
    </div>

    <div class="w-full">
      <h1 class="leading-text font-sans font-black tracking-tighter text-[674px] top-[-242px] absolute z-100" style="transform: rotate(10deg);">1</h1>
    </div>

    <div class="w-full space-y-2">
      {#each items as item, i}
        <p
          onmouseenter={() => (hoveredIndex = i)}
          onmouseleave={() => (hoveredIndex = null)}
          class="{item.ml} {item.font} text-2xl transition-all duration-300 ease-out cursor-pointer {hoveredIndex === i
            ? 'text-zinc-200 translate-x-1 [text-shadow:0_0_18px_rgba(228,228,231,0.45)]'
            : hoveredIndex !== null
              ? 'opacity-40 ' + item.baseColor
              : item.baseColor}"
        >
          {item.text}
        </p>
      {/each}
    </div>
  </div>
</section>