<script>
    import style from 'svelte-inline-css'

    export let rows
</script>

{#each rows as row}
    {#if row.length === 1}
        {#if row[0].type === 'text'}
            <p class="break-words whitespace-pre-wrap" use:style={{
                textAlign: row[0].textAlign,
            }}>{row[0].value}</p>
        {/if}
    {:else}
        <ul class="flex">
            {#each row as column}
                {#if column.type === 'text'}
                    <li class="break-words flex-1 min-w-0 whitespace-pre-wrap" use:style={{
                        textAlign: row[0].textAlign,
                    }}>{column.value}</li>
                {/if}
            {/each}
        </ul>
    {/if}
{/each}