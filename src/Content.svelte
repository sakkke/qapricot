<script>
    import JsonPreview from './components/JsonPreview.svelte'
    import Preview from './components/Preview.svelte'
    import PropertiesEditor from './components/PropertiesEditor.svelte'
    import {
        faArrowDown,
        faArrowLeft,
        faArrowRight,
        faArrowUp,
        faColumns,
        faEdit,
        faFileAlt,
        faFileExport,
        faMinus,
        faPlay,
        faPlus,
        faSave,
    } from '@fortawesome/free-solid-svg-icons'
    import html2canvas from 'html2canvas'
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

    async function exportImage () {
        const div = document.createElement('div')
        div.style.height = '297mm'
        div.style.width = '210mm'
        new Preview({
            props: { rows },
            target: div,
        })
        document.body.appendChild(div)
        const canvas = await html2canvas(div)
        const url = canvas.toDataURL()
        const a = document.createElement('a')
        a.download = `${Date.now().toString()}.png`
        a.href = url
        a.click()
        div.remove()
    }

    function saveJson () {
        const json = JSON.stringify(rows, null, 2)
        const blob = new Blob([json], { type: 'application/json' })
        const url = URL.createObjectURL(blob)
        const a = document.createElement('a')
        a.download = `${Date.now().toString()}.json`
        a.href = url
        a.click()
    }

    function showJsonPreview () {
        open(JsonPreview, { rows })
    }

    function showPreview () {
        open(Preview, { rows })
    }

    function showPropertiesEditor (v) {
        open(PropertiesEditor, { v }, {}, {
            onClosed: () => {
                updateRows()
            },
        })
    }

    function tradeColumn (i, j, k) {
        [rows[i][j], rows[i][k]] = [rows[i][k], rows[i][j]]
        updateRows()
    }

    function tradeRow (i, j) {
        [rows[i], rows[j]] = [rows[j], rows[i]]
        updateRows()
    }

    function updateRows () {
        rows = rows
    }
</script>

<main>
    <div class="flex flex-col min-h-screen">
        <div class="flex-grow">
            {#each rows as row, i}
                <div class="border m-0.5em">
                    <div class="flex flex-col">
                        <div class="flex w-full">
                            {#each row as column, j}
                                <div class="border flex-1 m-0.5em min-w-0">
                                    <div class="flex flex-col h-full">
                                        <p class="break-words flex-1 whitespace-pre-wrap">{column.text}</p>
                                        <div class="bg-gray-50 flex overflow-auto">
                                            <button class="first:ml-auto p-0.25em" on:click={() => showPropertiesEditor(column)} title="Properties">
                                                <Fa fw icon={faEdit}></Fa>
                                            </button>
                                            {#if j !== 0}
                                                <button class="p-0.25em" on:click={() => tradeColumn(i, j, j - 1)} title="Left">
                                                    <Fa fw icon={faArrowLeft}></Fa>
                                                </button>
                                            {/if}
                                            {#if j !== row.length - 1}
                                                <button class="p-0.25em" on:click={() => tradeColumn(i, j, j + 1)} title="Right">
                                                    <Fa fw icon={faArrowRight}></Fa>
                                                </button>
                                            {/if}
                                            {#if row.length !== 1}
                                                <button class="p-0.25em" on:click={() => deleteColumn(i, j)} title="Remove">
                                                    <Fa fw icon={faMinus}></Fa>
                                                </button>
                                            {/if}
                                        </div>
                                    </div>
                                </div>
                            {/each}
                        </div>
                        <div class="bg-gray-50 flex justify-end">
                            {#if i !== 0}
                                <button class="p-0.25em" on:click={() => tradeRow(i, i - 1)} title="Up">
                                    <Fa fw icon={faArrowUp}></Fa>
                                </button>
                            {/if}
                            {#if i !== rows.length - 1}
                                <button class="p-0.25em" on:click={() => tradeRow(i, i + 1)} title="Down">
                                    <Fa fw icon={faArrowDown}></Fa>
                                </button>
                            {/if}
                                <button class="p-0.25em" on:click={() => addColumn(i)} title="Split">
                                    <Fa fw icon={faColumns}></Fa>
                                </button>
                            {#if rows.length !== 1}
                                <button class="p-0.25em" on:click={() => deleteRow(i)} title="Remove">
                                    <Fa fw icon={faMinus}></Fa>
                                </button>
                            {/if}
                        </div>
                    </div>
                </div>
            {/each}
        </div>
        <div class="bg-gray-50 bottom-0 flex justify-end sticky w-full">
            <button class="p-0.25em" on:click={showPreview} title="Preview">
                <Fa fw icon={faPlay}></Fa>
            </button>
            <button class="p-0.25em" on:click={saveJson} title="Save">
                <Fa fw icon={faSave}></Fa>
            </button>
            <button class="p-0.25em" on:click={exportImage} title="Export">
                <Fa fw icon={faFileExport}></Fa>
            </button>
            <button class="p-0.25em" on:click={showJsonPreview} title="JSON Preview">
                <Fa fw icon={faFileAlt}></Fa>
            </button>
            <button class="p-0.25em" on:click={addRow} title="Add">
                <Fa fw icon={faPlus}></Fa>
            </button>
        </div>
    </div>
</main>