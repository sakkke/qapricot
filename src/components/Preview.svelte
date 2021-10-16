<script>
    export let rows
</script>

{#each rows as row}
    {#if row.length === 1}
        <p class="break-words whitespace-pre-wrap">{row[0].text}</p>
    {:else}
        <ul class="flex">
            {#each row as column}
                <li class="break-words flex-1 min-w-0 whitespace-pre-wrap">{column.text}</li>
            {/each}
        </ul>
    {/if}
{/each}