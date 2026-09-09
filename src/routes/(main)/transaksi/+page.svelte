<script lang="ts">
  import ListHistoriTransaksi from "$lib/page/main/transaksi/ListHistoriTransaksi.svelte";
  import ListTransaksiAktif from "$lib/page/main/transaksi/ListTransaksiAktif.svelte";
  import InformasiVisualTransaksi from "$lib/page/main/transaksi/InformasiVisualTransaksi.svelte";

</script>

<main class="p-5 space-y-5">
<InformasiVisualTransaksi/>
<ListTransaksiAktif/>
<ListHistoriTransaksi/>
</main>