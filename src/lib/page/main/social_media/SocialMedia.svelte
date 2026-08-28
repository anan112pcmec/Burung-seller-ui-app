<script lang="ts">
  interface SocialMedia {
    jenis_social_media: string;
    username?: string;
    status: 'terhubung' | 'tidak terhubung';
  }

  let dataSosmeds = $state<SocialMedia[]>([
    { jenis_social_media: "Facebook", username: "@tokosaya.official", status: "terhubung" },
    { jenis_social_media: "Instagram", username: "@tokosaya.id", status: "terhubung" },
    { jenis_social_media: "TikTok", username: "", status: "tidak terhubung" },
    { jenis_social_media: "Twitter / X", username: "", status: "tidak terhubung" },
    { jenis_social_media: "WhatsApp Business", username: "+62 812-3456-7890", status: "terhubung" }
  ]);

  function toggleStatus(index: number) {
    dataSosmeds[index].status = dataSosmeds[index].status === "terhubung" ? "tidak terhubung" : "terhubung";
  }
</script>

{#snippet SocialMediaRow(i: number, data: SocialMedia)}
  {@const isConnected = data.status === 'terhubung'}
  <!-- Card membentang 100% (w-full) dari ujung kiri ke kanan -->
  <div class="w-full bg-white border border-zinc-200 rounded-lg p-4 flex flex-col sm:flex-row items-start sm:items-center justify-between gap-4 shadow-2xs hover:border-zinc-300 transition-colors shrink-0">
    
    <!-- Informasi Sosmed (Kiri) -->
    <div class="flex items-center gap-3">
      <div class="w-2.5 h-2.5 rounded-full shrink-0 {isConnected ? 'bg-emerald-500' : 'bg-zinc-300'}"></div>
      <div>
        <div class="flex items-center gap-2">
          <h2 class="text-sm font-bold text-zinc-900 capitalize leading-tight">
            {data.jenis_social_media}
          </h2>
          <span class="px-2 py-0.5 rounded-full text-[9px] font-mono font-medium uppercase tracking-wider {isConnected ? 'bg-emerald-50 text-emerald-700 border border-emerald-200' : 'bg-zinc-100 text-zinc-500 border border-zinc-200'}">
            {isConnected ? 'Aktif' : 'Pasif'}
          </span>
        </div>
        <p class="text-xs text-zinc-400 font-mono mt-0.5">
          {data.username ? data.username : "Belum terhubung"}
        </p>
      </div>
    </div>

    <!-- Tombol & Status (Kanan) -->
    <div class="flex items-center gap-3 w-full sm:w-auto justify-between sm:justify-end border-t sm:border-t-0 pt-3 sm:pt-0 border-zinc-100">
      <span class="text-[10px] text-zinc-400 font-mono">
        Status: <strong class="text-zinc-700 font-semibold">{data.status}</strong>
      </span>
      <button
        onclick={() => toggleStatus(i)}
        class="px-3 py-1.5 text-xs font-medium rounded transition-all cursor-pointer {isConnected ? 'bg-rose-50 text-rose-600 hover:bg-rose-100 border border-rose-200' : 'bg-zinc-900 text-white hover:bg-zinc-800 shadow-2xs'}"
      >
        {isConnected ? 'Putuskan' : 'Hubungkan'}
      </button>
    </div>

  </div>
{/snippet}

<section id="social-media" class="p-6 bg-white text-zinc-800 font-sans w-full mx-auto flex flex-col space-y-4">
  <!-- Header Section -->
  <div class="mb-4 shrink-0">
    <h1 class="text-xl font-bold uppercase tracking-tight text-zinc-900">
      Integrasi Social Media
    </h1>
    <p class="text-xs text-zinc-400 mt-1">
      Kelola akun media sosial dalam daftar vertikal.
    </p>
  </div>

  <!-- Container Vertikal Scroll (1 baris 1 card) -->
  <div class="flex flex-col gap-3 w-full  pr-2 flex-1 border-t border-zinc-100 pt-3">
    {#each dataSosmeds as data_sosmed, i (data_sosmed.jenis_social_media)}
      {@render SocialMediaRow(i, data_sosmed)}
    {/each}
  </div>
</section>