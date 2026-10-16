<script>
  import { onMount } from 'svelte';
  import { PDFDocument } from 'pdf-lib';
  import SignaturePad from 'signature_pad';

  const SCALE = 1.5; // facteur d'affichage du PDF

  let { bytes, name = 'document.pdf' } = $props();

  let pdfjs;
  let pdfDoc = null;

  let pdfCanvas;
  let sigCanvas;
  let pad;

  let pageNum = $state(1);
  let totalPages = $state(0);
  let canvasW = $state(1);
  let canvasH = $state(1);

  let zone = $state(null);   // zone validée : { page, x, y, w, h } en pixels du canvas
  let draft = $state(null);  // zone en cours de tracé
  let start = null;
  
onMount(() => {
  pad = new SignaturePad(sigCanvas, { penColor: '#111827' });
  resizeSigCanvas();

  (async () => {
    pdfjs = await import('pdfjs-dist');
    const worker = (await import('pdfjs-dist/build/pdf.worker.min.mjs?url')).default;
    pdfjs.GlobalWorkerOptions.workerSrc = worker;
    await loadPdf();
  })();

  return () => pad.off();
});

async function loadPdf() {
  // .slice() : pdfjs consomme le buffer, on garde l'original pour pdf-lib
  pdfDoc = await pdfjs.getDocument({ data: bytes.slice() }).promise;
  totalPages = pdfDoc.numPages;
  pageNum = 1;
  await renderPage();
}

  function resizeSigCanvas() {
    const ratio = Math.max(window.devicePixelRatio || 1, 1);
    sigCanvas.width = sigCanvas.offsetWidth * ratio;
    sigCanvas.height = sigCanvas.offsetHeight * ratio;
    sigCanvas.getContext('2d').scale(ratio, ratio);
    pad.clear();
  }

  async function renderPage() {
    const page = await pdfDoc.getPage(pageNum);
    const viewport = page.getViewport({ scale: SCALE });
    canvasW = pdfCanvas.width = viewport.width;
    canvasH = pdfCanvas.height = viewport.height;
    await page.render({
      canvasContext: pdfCanvas.getContext('2d'),
      viewport
    }).promise;
  }

  async function changePage(delta) {
    pageNum += delta;
    await renderPage();
  }

  // 2. Définir la zone de signature (glisser sur le PDF)
  function pos(e) {
    const r = pdfCanvas.getBoundingClientRect();
    const k = pdfCanvas.width / r.width; // compense un éventuel redimensionnement CSS
    return { x: (e.clientX - r.left) * k, y: (e.clientY - r.top) * k };
  }

  function down(e) {
    e.currentTarget.setPointerCapture(e.pointerId);
    start = pos(e);
    draft = { x: start.x, y: start.y, w: 0, h: 0 };
  }

  function move(e) {
    if (!start) return;
    const p = pos(e);
    draft = {
      x: Math.min(start.x, p.x),
      y: Math.min(start.y, p.y),
      w: Math.abs(p.x - start.x),
      h: Math.abs(p.y - start.y)
    };
  }

  function up() {
    if (draft && draft.w > 30 && draft.h > 15) {
      zone = { page: pageNum, ...draft };
    }
    start = null;
    draft = null;
  }

  // 3. Signer et télécharger
  async function signAndDownload() {
    if (!zone || pad.isEmpty()) return;

    const doc = await PDFDocument.load(bytes);
    const page = doc.getPage(zone.page - 1);
    const { width: pw, height: ph } = page.getSize();

    // pixels du canvas -> points PDF
    const sx = pw / canvasW;
    const sy = ph / canvasH;

    const png = await doc.embedPng(pad.toDataURL('image/png'));

    // La signature garde ses proportions et est centrée dans la zone
    const boxW = zone.w * sx;
    const boxH = zone.h * sy;
    const ratio = Math.min(boxW / png.width, boxH / png.height);
    const w = png.width * ratio;
    const h = png.height * ratio;

    const x = zone.x * sx + (boxW - w) / 2;
    // L'axe Y du PDF part du BAS de la page, celui du canvas du HAUT
    const y = ph - (zone.y + zone.h) * sy + (boxH - h) / 2;

    page.drawImage(png, { x, y, width: w, height: h });

    const out = await doc.save();
    const url = URL.createObjectURL(new Blob([out], { type: 'application/pdf' }));
    const a = document.createElement('a');
    a.href = url;
    a.download = 'document-signe.pdf';
    a.click();
    setTimeout(() => URL.revokeObjectURL(url), 1000);
  }
</script>

<input type="file" accept="application/pdf" onchange={onFile} />

{#if totalPages}
  <div class="nav">
    <button onclick={() => changePage(-1)} disabled={pageNum <= 1}>←</button>
    <span>Page {pageNum} / {totalPages}</span>
    <button onclick={() => changePage(1)} disabled={pageNum >= totalPages}>→</button>
  </div>
{/if}

<p>1. Trace un rectangle sur le PDF pour définir la zone de signature.</p>

<div class="stage">
  <canvas bind:this={pdfCanvas}></canvas>

  <!-- Calque pour dessiner la zone -->
  <div
    class="overlay"
    role="presentation"
    onpointerdown={down}
    onpointermove={move}
    onpointerup={up}
  >
    {#each [draft, zone?.page === pageNum ? zone : null] as r}
      {#if r}
        <div
          class="zone"
          style="left:{(r.x / canvasW) * 100}%; top:{(r.y / canvasH) * 100}%;
                 width:{(r.w / canvasW) * 100}%; height:{(r.h / canvasH) * 100}%"
        ></div>
      {/if}
    {/each}
  </div>
</div>

<p>2. Dessine ta signature :</p>
<canvas bind:this={sigCanvas} class="sig"></canvas>

<div class="actions">
  <button onclick={() => pad.clear()}>Effacer</button>
  <button onclick={signAndDownload} disabled={!zone}>Signer et télécharger</button>
</div>

<style>
  .stage {
    position: relative;
    display: inline-block;
    max-width: 100%;
  }
  .stage canvas {
    display: block;
    max-width: 100%;
    height: auto;
    border: 1px solid #e2e8f0;
  }
  .overlay {
    position: absolute;
    inset: 0;
    cursor: crosshair;
    touch-action: none; /* indispensable pour tracer au doigt */
  }
  .zone {
    position: absolute;
    border: 2px dashed #2563eb;
    background: rgb(37 99 235 / 0.12);
    pointer-events: none;
  }
  .sig {
    width: 400px;
    height: 160px;
    max-width: 100%;
    border: 1px solid #94a3b8;
    border-radius: 8px;
    touch-action: none;
    background: white;
  }
  .nav, .actions {
    display: flex;
    gap: 0.75rem;
    align-items: center;
    margin: 0.75rem 0;
  }
</style>