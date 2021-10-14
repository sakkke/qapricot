<script>
    import PropertiesEditor from './components/PropertiesEditor.svelte'
    import {
        faArrowDown,
        faArrowLeft,
        faArrowRight,
        faArrowUp,
        faColumns,
        faEdit,
        faMinus,
        faPlus,
    } from '@fortawesome/free-solid-svg-icons'
    import { getContext } from 'svelte'
    import Fa from 'svelte-fa/src/fa.svelte'

    const { open } = getContext('simple-modal')

    let rows = [[createProperties()]]

    function addRow () {
        rows.push([createProperties()])
        updateRows()
    }

    function addColumn (i) {
        rows[i].push(createProperties())
        updateRows()
    }

    function createProperties () {
        return {
            text: '',
        }
    }

    function deleteColumn (i, j) {
        rows[i].splice(j, 1)
        updateRows()
    }

    function deleteRow (i) {
        rows.splice(i, 1)
        updateRows()
    }

    function showPropertiesEditor (v) {
        open(PropertiesEditor, { v }, {}, {
            onClosed: () => {
                updateRows()
            },
        })
    }

    function updateRows () {
        rows = rows
    }

    function tradeColumn (i, j, k) {
        [rows[i][j], rows[i][k]] = [rows[i][k], rows[i][j]]
        updateRows()
    }

    function tradeRow (i, j) {
        [rows[i], rows[j]] = [rows[j], rows[i]]
        updateRows()
    }
</script>

<main>
    <div class="border m-0.5em">
        <div class="flex flex-row-reverse">
            <div class="flex">
                <button class="p-0.25em" on:click={addRow}>
                    <Fa fw icon={faPlus}></Fa>
                </button>
            </div>
        </div>
        {#each rows as row, i}
            <div class="border m-0.5em">
                <div class="flex flex-row-reverse">
                    <div class="flex">
                        {#if i !== 0}
                            <button class="p-0.25em" on:click={() => tradeRow(i, i - 1)}>
                                <Fa fw icon={faArrowUp}></Fa>
                            </button>
                        {/if}
                        {#if i !== rows.length - 1}
                            <button class="p-0.25em" on:click={() => tradeRow(i, i + 1)}>
                                <Fa fw icon={faArrowDown}></Fa>
                            </button>
                        {/if}
                        <button class="p-0.25em" on:click={() => addColumn(i)}>
                            <Fa fw icon={faColumns}></Fa>
                        </button>
                        {#if i !== 0}
                            <button class="p-0.25em" on:click={() => deleteRow(i)}>
                                <Fa fw icon={faMinus}></Fa>
                            </button>
                        {/if}
                    </div>
                    {#each row as column, j}
                        <div class="border flex-1 m-0.5em min-w-0">
                            <div class="flex flex-row-reverse">
                                <div class="flex">
                                    <button class="p-0.25em" on:click={() => showPropertiesEditor(column)}>
                                        <Fa fw icon={faEdit}></Fa>
                                    </button>
                                    {#if j !== row.length - 1}
                                        <button class="p-0.25em" on:click={() => tradeColumn(i, j, j + 1)}>
                                            <Fa fw icon={faArrowLeft}></Fa>
                                        </button>
                                    {/if}
                                    {#if j !== 0}
                                        <button class="p-0.25em" on:click={() => tradeColumn(i, j, j - 1)}>
                                            <Fa fw icon={faArrowRight}></Fa>
                                        </button>
                                    {/if}
                                    {#if row.length !== 1}
                                        <button class="p-0.25em" on:click={() => deleteColumn(i, j)}>
                                            <Fa fw icon={faMinus}></Fa>
                                        </button>
                                    {/if}
                                </div>
                                <p class="break-words flex-1 overflow-auto whitespace-pre-wrap">{column.text}</p>
                            </div>
                        </div>
                    {/each}
                </div>
            </div>
        {/each}
    </div>
</main>