<script>
    export let v
</script>

{#each Object.keys(v) as k}
    <div class="flex flex-col">
        {#if k === 'type'}
            <label>{k} <select bind:value={v[k]}>
                {#each [
                    'text',
                ] as value}
                    <option {value}>{value}</option>
                {/each}
            </select></label>
        {:else if typeof v[k] === 'boolean'}
            <label>{k} <input bind:checked={v[k]} type="checkbox"></label>
        {:else if typeof v[k] === 'string'}
            <label>{k} <textarea bind:value={v[k]} class="align-top border resize-none" cols="50" rows="3"></textarea></label>
        {/if}
    </div>
{/each}