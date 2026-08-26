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
          <h1 class="text-4xl md:text-5xl font-extrabold tracking-tight text-end leading-tight">
            <span class="text-transparent bg-clip-text bg-gradient-to-r from-zinc-400 to-zinc-100">Bergabung Dengan Kami Hanya Butuh</span>
          </h1>
      </div>

      <!-- Card UI iOS / macOS Style -->
      <div class="w-full h-full px-2">
        <div class="px-4 flex-1 relative flex items-center justify-center overflow-hidden">
          <svg class="w-full h-full" viewBox="0 0 300 150" fill="none" xmlns="http://www.w3.org/2000/svg">
            <style>
              @keyframes pulse-glow {
                0%, 100% { opacity: 0.75; }
                50% { opacity: 1; filter: drop-shadow(0 0 3px rgba(244, 244, 245, 0.35)); }
              }
              @keyframes flow-fast { to { stroke-dashoffset: -24; } }
              .flow { animation: flow-fast 1s linear infinite; }
              .flow-slow { animation: flow-fast 1.6s linear infinite; }
              .glow { animation: pulse-glow 2.4s ease-in-out infinite; }
              text { font-family: 'Inter', ui-sans-serif, system-ui, sans-serif; }
              .title { fill: #fafafa; font-weight: 700; letter-spacing: 0.4px; }
              .sub { fill: #71717a; font-weight: 400; }
              .tag { fill: #d4d4d8; font-weight: 700; letter-spacing: 0.3px; }
            </style>

            <defs>
              <radialGradient id="coreGlow" cx="50%" cy="50%" r="50%">
                <stop offset="0%" stop-color="#f4f4f5" stop-opacity="0.18" />
                <stop offset="100%" stop-color="#f4f4f5" stop-opacity="0" />
              </radialGradient>
              <clipPath id="logoClip">
                <rect x="219" y="14" width="56" height="48" rx="6" />
              </clipPath>
            </defs>

            <circle cx="247" cy="38" r="55" fill="url(#coreGlow)" />

            <!-- WIRES: left column -> chip -->
            <path d="M188 15.5 H195" stroke="#3f3f46" stroke-width="1" />
            <path d="M188 15.5 H195" stroke="#e4e4e7" stroke-width="1" stroke-dasharray="3 8" class="flow" />
            <path d="M188 37.5 H195" stroke="#3f3f46" stroke-width="1" />
            <path d="M188 37.5 H195" stroke="#d4d4d8" stroke-width="1" stroke-dasharray="3 8" class="flow-slow" />
            <path d="M188 59.5 H195" stroke="#3f3f46" stroke-width="1" />
            <path d="M188 59.5 H195" stroke="#a1a1aa" stroke-width="1" stroke-dasharray="3 8" class="flow" />

            <!-- WIRES: bottom row -> chip -->
            <path d="M49.5 100 V88 H210 V70" stroke="#3f3f46" stroke-width="1" />
            <path d="M49.5 100 V88 H210 V70" stroke="#a1a1aa" stroke-width="1" stroke-dasharray="3 8" class="flow-slow" />
            <path d="M144.5 100 V80 H230 V70" stroke="#3f3f46" stroke-width="1" />
            <path d="M144.5 100 V80 H230 V70" stroke="#d4d4d8" stroke-width="1" stroke-dasharray="3 8" class="flow" />
            <path d="M244 100 V70" stroke="#3f3f46" stroke-width="1" />
            <path d="M244 100 V70" stroke="#e4e4e7" stroke-width="1" stroke-dasharray="3 8" class="flow-slow" />

            <!-- LEFT COLUMN: 3 stacked full-width cards, tinggi = tinggi chip (y6-70) -->
            <g transform="translate(4, 6)">
              <rect width="184" height="19" rx="5" fill="#09090b" stroke="#3f3f46" stroke-width="1" />
              <circle cx="8" cy="9.5" r="1.6" fill="#fafafa" class="glow" />
              <text x="14" y="8.5" class="title" font-size="5">TRAFFIC ADAPTER</text>
              <text x="14" y="15" class="sub" font-size="3.8">Multi-tenant HTTP/2 gateway</text>
              <rect x="156" y="4.5" width="24" height="9" rx="2" fill="none" stroke="#52525b" stroke-width="0.6" />
              <text x="168" y="10.5" text-anchor="middle" class="tag" font-size="3.5">LIVE</text>
            </g>

            <g transform="translate(4, 28)">
              <rect width="184" height="19" rx="5" fill="#09090b" stroke="#3f3f46" stroke-width="1" />
              <path d="M8 6.5h6l1 2.4h-8l1-2.4zm-1 2.4v4h8v-4" stroke="#a1a1aa" stroke-width="0.7" fill="none" />
              <text x="22" y="8.5" class="title" font-size="5">SELLER HUB</text>
              <text x="22" y="15" class="sub" font-size="3.8">Sinkronisasi stok otomatis</text>
              <rect x="156" y="4.5" width="24" height="9" rx="2" fill="none" stroke="#52525b" stroke-width="0.6" />
              <text x="168" y="10.5" text-anchor="middle" class="tag" font-size="3.5">SYNC</text>
            </g>

            <g transform="translate(4, 50)">
              <rect width="184" height="19" rx="5" fill="#09090b" stroke="#3f3f46" stroke-width="1" />
              <path d="M9 5.5l-3 5h3.2l-0.8 5 3.8-5.5h-3l1-4.5z" fill="none" stroke="#a1a1aa" stroke-width="0.7" stroke-linejoin="round" />
              <text x="22" y="8.5" class="title" font-size="5">PUSH ENGINE</text>
              <text x="22" y="15" class="sub" font-size="3.8">WebSocket instan, latensi rendah</text>
              <rect x="156" y="4.5" width="24" height="9" rx="2" fill="none" stroke="#52525b" stroke-width="0.6" />
              <text x="168" y="10.5" text-anchor="middle" class="tag" font-size="3.5">&lt;5MS</text>
            </g>

            <!-- BOTTOM ROW: 3 cards -->
            <g transform="translate(4, 100)">
              <rect width="91" height="42" rx="3" fill="#09090b" stroke="#3f3f46" stroke-width="1" />
              <path d="M8 12a1.3 1.3 0 100-2.6 1.3 1.3 0 000 2.6zm12 0a1.3 1.3 0 100-2.6 1.3 1.3 0 000 2.6zM6 10h2v-4h6v4h2l2.5 2.5v1.5h-1.2" stroke="#a1a1aa" stroke-width="0.7" fill="none" />
              <text x="23" y="14" class="title" font-size="5">AUTO-DISPATCH</text>
              <text x="6" y="28" class="sub" font-size="3.8">Routing kurir otomatis</text>
              <text x="6" y="34" class="tag" font-size="3.6">Tracking real-time</text>
            </g>

            <g transform="translate(99, 100)">
              <rect width="91" height="42" rx="3" fill="#09090b" stroke="#3f3f46" stroke-width="1" />
              <path d="M6 8l4-2 4 2-4 2-4-2zm0 4l4 2 4-2m-8 4l4 2 4-2" stroke="#a1a1aa" stroke-width="0.7" fill="none" />
              <text x="19" y="14" class="title" font-size="5">All Record Tracked</text>
              <text x="6" y="28" class="sub" font-size="3.8">Quorum queue, anti-rugi</text>
              <text x="6" y="34" class="tag" font-size="3.6">Zero-loss guarantee</text>
            </g>

            <g transform="translate(195, 100)">
              <rect width="98" height="42" rx="3" fill="#09090b" stroke="#3f3f46" stroke-width="1" />
              <path d="M8 8c0 .8 1.8 1.4 4 1.4s4-.6 4-1.4-1.8-1.4-4-1.4-4 .6-4 1.4zm0 0v6c0 .8 1.8 1.4 4 1.4s4-.6 4-1.4V8m-8 3c0 .8 1.8 1.4 4 1.4s4-.6 4-1.4" stroke="#a1a1aa" stroke-width="0.7" fill="none" />
              <text x="20" y="12" class="title" font-size="5">DATA ENGINE</text>
              <text x="6" y="22" class="sub" font-size="3.8">Persistensi high-availability</text>
              <text x="6" y="29" class="tag" font-size="3.6">99.9% Save Persist</text>
            </g>

            <!-- CHIP UTAMA — posisi tetap pojok kanan atas -->
            <g>
              <rect x="195" y="6" width="98" height="64" rx="10" fill="#09090b" stroke="#52525b" stroke-width="0.6" />
              <rect x="195" y="6" width="98" height="64" rx="10" fill="none" stroke="#e4e4e7" stroke-width="0.2" class="glow" />
              <rect x="201" y="58" width="86" height="8" rx="2" fill="#18181b" stroke="#3f3f46" stroke-width="0.2" />
              <text x="204" y="64" fill="#fafafa" font-weight="700" font-size="4" letter-spacing="0.6" font-family="Inter, ui-sans-serif, sans-serif">BURUNG REAL-TIME CORE</text>
            </g>

            <image
              href="src\lib\assets\LogoBurung.svg"
              x="110" y="-10" width="280" height="110"
              clip-path="url(#logoClip)"
              preserveAspectRatio="xMidYMid meet"
              class="glow"
            />
          </svg>
        </div>
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