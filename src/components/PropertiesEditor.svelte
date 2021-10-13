<script>
    export let v
</script>

{#each Object.keys(v) as k}
    {#if typeof v[k] === 'boolean'}
        <label>{k} <input bind:checked={v[k]} type="checkbox"></label>
    {:else if typeof v[k] === 'string'}
        <label>{k} <textarea bind:value={v[k]} class="align-top border resize-none" cols="50" rows="3"></textarea></label>
    {/if}
{/each}