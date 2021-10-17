<script>
    export let v

    const choicesByProperty = {
        textAlign: [
            'center',
            'end',
            'justify',
            'left',
            'right',
            'start',
        ],
        type: [
            'html',
            'image',
            'markdown',
            'text',
        ],
    }
</script>

{#each Object.keys(v) as k}
    <div class="flex flex-col">
        {#if choicesByProperty.hasOwnProperty(k)}
            <label>{k} <select bind:value={v[k]}>
                {#each choicesByProperty[k] as value}
                    <option {value}>{value}</option>
                {/each}
            </select></label>
        {:else if typeof v[k] === 'boolean'}
            <label>{k} <input bind:checked={v[k]} type="checkbox"></label>
        {:else if typeof v[k] === 'object'}
            {#if v[k].type === 'color'}
                <label>{k} <input bind:value={v[k].value} type="color"></label>
            {:else if v[k].type === 'file'}
                <label>{k} <input on:change={event => {
                    const { target } = event
                    const { files } = target
                    const file = files[0]
                    if (!!file) {
                        const reader = new window.FileReader()
                        reader.addEventListener('load', () => {
                            const { result } = reader
                            v[k].value = result
                        }, false)
                        reader.readAsDataURL(file)
                    }
                }} type="file"></label>
            {:else if choicesByProperty.hasOwnProperty(v[k].type)}
                <label>{k} <select bind:value={v[k].value}>
                    {#each choicesByProperty[v[k].type] as value}
                        <option {value}>{value}</option>
                    {/each}
                </select></label>
            {/if}
        {:else if typeof v[k] === 'string'}
            <label>{k} <textarea bind:value={v[k]} class="align-top border resize-none" cols="50" rows="3"></textarea></label>
        {/if}
    </div>
{/each}