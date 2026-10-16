<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  import { pdfStore } from '#lib/pdfStore.svelte.js';
  import PdfSigner from '#lib/components/PdfSigner.svelte';

  // Si on arrive ici sans fichier (rafraîchissement, lien direct), retour à l'accueil
  onMount(() => {
    if (!pdfStore.bytes) goto('/');
  });
</script>

{#if pdfStore.bytes}
  <a href="/">← Changer de fichier</a>
  <h2>{pdfStore.name}</h2>
  <PdfSigner bytes={pdfStore.bytes} name={pdfStore.name} />
{/if}