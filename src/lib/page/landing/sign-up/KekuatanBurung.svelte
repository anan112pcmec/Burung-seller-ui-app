<script lang="ts">
</script>

<section id="kekuatan-burung" class="p-5">
  <div class="px-4 flex-1 relative flex items-center justify-center overflow-hidden select-none">
    <svg class="w-full h-full" viewBox="0 0 300 150" fill="none" xmlns="http://www.w3.org/2000/svg">
      <style>
        /* Flow pulsa titik cahaya teratur */
        @keyframes pulse-flow {
          0% { stroke-dashoffset: 36; opacity: 0.2; }
          50% { opacity: 0.95; }
          100% { stroke-dashoffset: 0; opacity: 0.2; }
        }

        @keyframes pulse-subtle-glow {
          0%, 100% { opacity: 0.92; filter: drop-shadow(0 0 1px rgba(9, 9, 11, 0.15)); }
          50% { opacity: 1; filter: drop-shadow(0 0 3px rgba(9, 9, 11, 0.35)); }
        }

        /* Base Wire Style */
        .wire-bg { 
          stroke: #e4e4e7; 
          stroke-width: 0.8; 
          stroke-linecap: round; 
        }

        /* Flow Particle: Lingkaran titik berjarak */
        .flow-particle {
          stroke: #09090b;
          stroke-width: 1.2;
          stroke-dasharray: 2 16;
          stroke-linecap: round;
          animation: pulse-flow 2.2s ease-in-out infinite;
        }

        .flow-particle-slow {
          stroke: #71717a;
          stroke-width: 1;
          stroke-dasharray: 2 20;
          stroke-linecap: round;
          animation: pulse-flow 3.2s ease-in-out infinite;
        }

        .glow { animation: pulse-subtle-glow 2.5s ease-in-out infinite; }
        text { font-family: 'Inter', ui-sans-serif, system-ui, sans-serif; }
        .title { fill: #09090b; font-weight: 700; letter-spacing: 0.5px; }
        .sub { fill: #71717a; font-weight: 400; letter-spacing: 0.1px; }
        .tag { fill: #09090b; font-weight: 700; letter-spacing: 0.5px; }
      </style>

      <defs>
        <clipPath id="logoClip">
          <rect x="219" y="14" width="56" height="48" rx="2" />
        </clipPath>
      </defs>

      <!-- WIRES: Left Column -> Main Chip -->
      <g id="wires-left">
        <path d="M 188 15.5 H 195" class="wire-bg" />
        <path d="M 188 15.5 H 195" class="flow-particle" />

        <path d="M 188 37.5 H 195" class="wire-bg" />
        <path d="M 188 37.5 H 195" class="flow-particle-slow" />

        <path d="M 188 59.5 H 195" class="wire-bg" />
        <path d="M 188 59.5 H 195" class="flow-particle" />
      </g>

      <!-- WIRES: Bottom Row -> Main Chip -->
      <g id="wires-bottom">
        <path d="M 49.5 98 C 49.5 76, 210 88, 210 70" class="wire-bg" />
        <path d="M 49.5 98 C 49.5 76, 210 88, 210 70" class="flow-particle-slow" />

        <path d="M 144.5 98 C 144.5 78, 230 84, 230 70" class="wire-bg" />
        <path d="M 144.5 98 C 144.5 78, 230 84, 230 70" class="flow-particle" />

        <path d="M 244 98 C 244 88, 244 80, 244 70" class="wire-bg" />
        <path d="M 244 98 C 244 88, 244 80, 244 70" class="flow-particle-slow" />
      </g>

      <!-- LEFT COLUMN: 3 stacked cards -->
      <g id="left-column">
        <g transform="translate(4, 6)">
          <rect width="184" height="19" rx="2" fill="#ffffff" stroke="#d4d4d8" stroke-width="0.75" />
          <rect x="0" y="0" width="2" height="19" rx="1" fill="#09090b" />
          <text x="10" y="8.5" class="title" font-size="4.8">TRAFFIC ADAPTER</text>
          <text x="10" y="14.8" class="sub" font-size="3.6">Multi-tenant HTTP/2 gateway</text>
          <rect x="156" y="4.5" width="24" height="10" rx="1" fill="#f4f4f5" stroke="#09090b" stroke-width="0.5" />
          <text x="168" y="11" text-anchor="middle" class="tag" font-size="3.4">LIVE</text>
        </g>

        <g transform="translate(4, 28)">
          <rect width="184" height="19" rx="2" fill="#ffffff" stroke="#d4d4d8" stroke-width="0.75" />
          <rect x="0" y="0" width="2" height="19" rx="1" fill="#3f3f46" />
          <text x="10" y="8.5" class="title" font-size="4.8">SELLER HUB</text>
          <text x="10" y="14.8" class="sub" font-size="3.6">Sinkronisasi stok otomatis</text>
          <rect x="156" y="4.5" width="24" height="10" rx="1" fill="#f4f4f5" stroke="#71717a" stroke-width="0.5" />
          <text x="168" y="11" text-anchor="middle" class="tag" font-size="3.4">SYNC</text>
        </g>

        <g transform="translate(4, 50)">
          <rect width="184" height="19" rx="2" fill="#ffffff" stroke="#d4d4d8" stroke-width="0.75" />
          <rect x="0" y="0" width="2" height="19" rx="1" fill="#09090b" />
          <text x="10" y="8.5" class="title" font-size="4.8">PUSH ENGINE</text>
          <text x="10" y="14.8" class="sub" font-size="3.6">WebSocket instan, latensi rendah</text>
          <rect x="156" y="4.5" width="24" height="10" rx="1" fill="#f4f4f5" stroke="#09090b" stroke-width="0.5" />
          <text x="168" y="11" text-anchor="middle" class="tag" font-size="3.4">&lt;5MS</text>
        </g>
      </g>

      <!-- BOTTOM ROW: 3 cards -->
      <g id="bottom-row">
        <g transform="translate(4, 98)">
          <rect width="91" height="42" rx="2" fill="#ffffff" stroke="#d4d4d8" stroke-width="0.75" />
          <path d="M8 12a1 1 0 100-2 1 1 0 000 2zm12 0a1 1 0 100-2 1 1 0 000 2zM6 10h2v-4h6v4h2l2.5 2.5v1.5h-1.2" stroke="#09090b" stroke-width="0.75" fill="none" />
          <text x="23" y="13.5" class="title" font-size="4.8">AUTO-DISPATCH</text>
          <text x="6" y="27" class="sub" font-size="3.6">Routing kurir otomatis</text>
          <text x="6" y="35" class="tag" font-size="3.4">Tracking real-time</text>
        </g>

        <g transform="translate(99, 98)">
          <rect width="91" height="42" rx="2" fill="#ffffff" stroke="#d4d4d8" stroke-width="0.75" />
          <path d="M6 8l4-2 4 2-4 2-4-2zm0 4l4 2 4-2m-8 4l4 2 4-2" stroke="#09090b" stroke-width="0.75" fill="none" />
          <text x="19" y="13.5" class="title" font-size="4.8">All Record Tracked</text>
          <text x="6" y="27" class="sub" font-size="3.6">Quorum queue, anti-rugi</text>
          <text x="6" y="35" class="tag" font-size="3.4">Zero-loss guarantee</text>
        </g>

        <g transform="translate(195, 98)">
          <rect width="98" height="42" rx="2" fill="#ffffff" stroke="#d4d4d8" stroke-width="0.75" />
          <path d="M8 8c0 .8 1.8 1.4 4 1.4s4-.6 4-1.4-1.8-1.4-4-1.4-4 .6-4 1.4zm0 0v6c0 .8 1.8 1.4 4 1.4s4-.6 4-1.4V8m-8 3c0 .8 1.8 1.4 4 1.4s4-.6 4-1.4" stroke="#09090b" stroke-width="0.75" fill="none" />
          <text x="20" y="12" class="title" font-size="4.8">DATA ENGINE</text>
          <text x="6" y="24" class="sub" font-size="3.6">Persistensi high-availability</text>
          <text x="6" y="32" class="tag" font-size="3.4">99.9% Save Persist</text>
        </g>
      </g>

      <!-- MAIN CHIP CONTAINER -->
      <g id="main-chip">
        <rect x="198" y="9" width="92" height="58" rx="1.5" fill="none" stroke="#e4e4e7" stroke-width="0.75" />
      </g>

      <!-- LOGO OVERLAY -->
      <image
        href="src/lib/assets/LogoBurung.svg"
        x="110" y="-10" width="280" height="110"
        clip-path="url(#logoClip)"
        preserveAspectRatio="xMidYMid meet"
        class="glow"
      />
    </svg>
  </div>
</section>