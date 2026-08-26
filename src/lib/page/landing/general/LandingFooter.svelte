<script lang="ts">
  const currentYear = new Date().getFullYear();

  const footerLinks = [
    {
      title: "Ekosistem",
      links: [
        { name: "Sarang Seller", href: "#" },
        { name: "Sangkar Belanja", href: "#" },
        { name: "Kicau Review", href: "#" },
        { name: "Merpati Express", href: "#" }
      ]
    },
    {
      title: "Solusi Seller",
      links: [
        { name: "Multi-Tier Account", href: "#" },
        { name: "Program Distributor", href: "#" },
        { name: "Brand Resmi", href: "#" },
        { name: "Integrasi & API", href: "#" }
      ]
    },
    {
      title: "Bantuan & Legal",
      links: [
        { name: "Syarat & Ketentuan", href: "#" },
        { name: "Kebijakan Privasi", href: "#" },
        { name: "Pusat Bantuan", href: "#" },
        { name: "Status Layanan", href: "#" }
      ]
    }
  ];
</script>

<footer id="footer" class="w-full bg-white text-zinc-950 border-t border-zinc-200 px-8 md:px-16 pt-16 pb-12">
  <div class="max-w-7xl mx-auto space-y-16">
    
    <!-- Top Row: Brand & Nav Links -->
    <div class="grid grid-cols-1 md:grid-cols-12 gap-12">
      
      <!-- Brand Info -->
      <div class="md:col-span-5 space-y-4">
        <h3 class="text-2xl font-black tracking-tighter text-zinc-950 uppercase">
          BURUNG <span class="text-xs font-semibold text-zinc-400 normal-case tracking-normal">/ Partner</span>
        </h3>
        <p class="text-zinc-500 text-xs leading-relaxed max-w-sm">
          Infrastruktur terpadu untuk peternak, seller, dan distributor burung di seluruh Indonesia. Transparan, adil, dan dalam kendali Anda sepenuhnya.
        </p>
        <div class="flex items-center gap-2 pt-2">
          <span class="w-2 h-2 rounded-full bg-emerald-500 inline-block"></span>
          <span class="text-[11px] font-bold text-zinc-500 tracking-wider uppercase">Sistem Operasional Normal</span>
        </div>
      </div>

      <!-- Links Grid -->
      <div class="md:col-span-7 grid grid-cols-2 md:grid-cols-3 gap-8">
        {#each footerLinks as group}
          <div class="space-y-3">
            <h4 class="text-xs font-black uppercase tracking-widest text-zinc-400">{group.title}</h4>
            <ul class="space-y-2">
              {#each group.links as link}
                <li>
                  <a href={link.href} class="text-xs font-medium text-zinc-600 hover:text-zinc-950 transition-colors">
                    {link.name}
                  </a>
                </li>
              {/each}
            </ul>
          </div>
        {/each}
      </div>

    </div>

    <!-- Bottom Row: Copyright & Socials -->
    <div class="pt-8 border-t border-zinc-200 flex flex-col md:flex-row items-center justify-between gap-4 text-xs font-medium text-zinc-400">
      <p>© {currentYear} Burung Ecosystem. Hak Cipta Dilindungi.</p>
      <div class="flex gap-6">
        <a href="#instagram" class="hover:text-zinc-950 transition-colors">Instagram</a>
        <a href="#whatsapp" class="hover:text-zinc-950 transition-colors">WhatsApp</a>
        <a href="#youtube" class="hover:text-zinc-950 transition-colors">YouTube</a>
      </div>
    </div>

  </div>
</footer>