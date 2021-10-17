<script>
    import marked from 'marked'
    import style from 'svelte-inline-css'

    export let rows
</script>

{#each rows as row}
    {#if row.length === 1}
        {#if row[0].type === 'html'}
            <div class="break-words whitespace-pre-wrap" use:style={{
                textAlign: row[0].textAlign,
            }}>{@html row[0].value}</div>
        {:else if row[0].type === 'markdown'}
            <div class="break-words whitespace-pre-wrap" use:style={{
                textAlign: row[0].textAlign,
            }}>{@html marked(row[0].value)}</div>
        {:else if row[0].type === 'text'}
            <p class="break-words whitespace-pre-wrap" use:style={{
                textAlign: row[0].textAlign,
            }}>{row[0].value}</p>
        {/if}
    {:else}
        <div class="flex">
            {#each row as column}
                {#if column.type === 'html'}
                    <div class="break-words flex-1 min-w-0 whitespace-pre-wrap" use:style={{
                        textAlign: row[0].textAlign,
                    }}>{@html column.value}</div>
                {:else if column.type === 'markdown'}
                    <div class="break-words flex-1 min-w-0 whitespace-pre-wrap" use:style={{
                        textAlign: row[0].textAlign,
                    }}>{@html marked(column.value)}</div>
                {:else if column.type === 'text'}
                    <p class="break-words flex-1 min-w-0 whitespace-pre-wrap" use:style={{
                        textAlign: row[0].textAlign,
                    }}>{column.value}</p>
                {/if}
            {/each}
        </div>
    {/if}
{/each}