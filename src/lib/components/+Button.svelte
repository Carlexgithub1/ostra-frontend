<script>
    let { Children } = $props();
</script>

<button>
    {@render Children?.()}
</button>

<style>
*{
    --color-btn-bg:  hsl(255, 100%, 50%);
}
    button {
        background-color: var(--color-btn-bg);
        color: var(--white);
        border: none;
        padding: 0.5rem 1rem;
        border-radius: 4px;
        cursor: pointer;
        font-size: 1rem;
    }

    button:hover {
        opacity: 0.8;
    }
</style>