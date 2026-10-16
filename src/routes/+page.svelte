<script>
    import { goto } from '$app/navigation';
    import { pdfStore } from '#lib/pdfStore.svelte.js';
    import { onMount } from 'svelte';
    
    import lightLogo from '#lib/assets/logo/logo_rvb_cbg.svg';
    import darkLogo from '#lib/assets/logo/logo_rvb_dbg.svg';
    import "./Button.css";

    import Button from '#lib/components/+Button.svelte';

    function handleFileSelect(event) {
        const fileInput = document.getElementById('fileInput');
        
        event.preventDefault();
        // Logic to handle file selection goes here
        fileInput?.click(); // Trigger the hidden file input
        console.log("File selection button clicked");
    }

    async function handleFileChange(event) {
        const file = event.target.files[0];
        if (file) {
            console.log("Selected file:", file.name);
            pdfStore.bytes = new Uint8Array(await file.arrayBuffer());
            pdfStore.name = file.name;
            await goto("/editor");
        }
    }
</script>

<div class="container">
    <header>
    <div class="wrapper">
        <a href="/" class="logo-container">
        <img src={darkLogo} alt="Dark Logo" srcset="" class="logo darklogo">
    </a>
    </div>
    </header>

    <section class="content">
        <div class="wrapper">
            <h1 class="title">Fais signer <span>facilement</span> tes documents <span>en ligne</span></h1>

            <form class="card file-importer">
                <input type="file" id="fileInput" class="file-input" accept="application/pdf" onchange={handleFileChange} />
                <button type="button" class="btn cta" onclick={handleFileSelect}>Selectionner un fichier</button>
                <p class="card-content">Importe le document a faire signer</p>
            </form>

        </div>
    </section>
</div>

<style>

*{
    --border-card: 1px dashed hsl(0, 0%, 20%);
}

.file-input{
    display: none;
}
.card{
    padding: 1rem;
    border: var(--border-card);
    border-radius: 1rem;
    display: grid;
    grid-template-rows: auto 1fr;
    gap: .8rem;
    max-width: 100%;
}

*{
    --color-text: hsl(0, 0%, 30%);
    --white: hsl(0, 0%, 100%);
    --color-accent: hsl(75, 100%, 50%);
}

h1, h2, h3, h4, h5, h6{
    color: var(--white);
}

.container{
    font-family: "Geist Variable", sans-serif;
    color: var(--color-text);
    width: 100%;
    margin-inline: auto;
}

.wrapper{
    width: 100%;
    max-width: 1200px;
    padding-inline: 1rem;
}

header{
    padding-block: 2rem;
    /* background-color: var(--color-bg); */
}

header .wrapper{
    display: flex;
    justify-content: center;
}
    /* LOGOS */
    .logo-container .logo{
        height: 5rem;
    }

    .content{
        width: 100%;
    }
    .content .wrapper{
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        gap: 1rem;
        gap: 2rem;
    }
    .title{
        max-width: 35rem;
        font-size:3rem;
        baseline-shift: 0.2;
        line-height: .9;
        letter-spacing: -0.02em;
        text-align: center;
    }

    .title span{
        color: var(--color-accent);
    }

    p{
        text-align: center;
        font-size: 1.2rem;
        color: var(--color-text);
    }

</style>