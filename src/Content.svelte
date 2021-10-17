<script>
    import ExportPreview from './components/ExportPreview.svelte'
    import Help from './components/Help.svelte'
    import JsonPreview from './components/JsonPreview.svelte'
    import Preview from './components/Preview.svelte'
    import PropertiesEditor from './components/PropertiesEditor.svelte'
    import Settings from './components/Settings.svelte'
    import {
        faArrowDown,
        faArrowLeft,
        faArrowRight,
        faArrowUp,
        faCog,
        faColumns,
        faEdit,
        faEraser,
        faFile,
        faFileAlt,
        faFileExport,
        faMinus,
        faPlay,
        faPlus,
        faQuestion,
        faSave,
    } from '@fortawesome/free-solid-svg-icons'
    import html2canvas from 'html2canvas'
    import marked from 'marked'
    import { getContext } from 'svelte'
    import Fa from 'svelte-fa/src/fa.svelte'
    import style from 'svelte-inline-css'

    const { open } = getContext('simple-modal')

    let settings = localStorage.getItem('settings') === null
        ? createSettings()
        : { ...createSettings(), ...JSON.parse(localStorage.getItem('settings')) }
    let qapricot = {
        contents: [[createProperties()]],
        meta: {
            title: settings['title.default'],
        },
    }
    $: rows = qapricot.contents

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
            textAlign: 'left',
            type: 'text',
            value: '',
        }
    }

    function createSettings () {
        return {
            'style.exportedImage.padding': '10mm',
            'style.navigationBar.backgroundColor': { type: 'color', value: '#f9fafb' }, // gray-50
            'style.navigationBar.color': { type: 'color', value: '#000000' },
            'style.root.backgroundColor': { type: 'color', value: '#ffffff' },
            'style.root.borderColor': { type: 'color', value: '#e5e7eb' }, // gray-200
            'style.root.color': { type: 'color', value: '#000000' },
            'title.default': 'No name',
        }
    }

    // ref: https://stackoverflow.com/a/54555834
    function cropCanvas (canvas, x, y, width, height) {
        const result = document.createElement('canvas')
        result.height = height
        result.width = width
        const context = result.getContext('2d')
        context.drawImage(canvas, x, y, width, height, 0, 0, width, height)
        return result
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
        div.style.padding = settings['style.exportedImage.padding']
        div.style.minHeight = '297mm'
        div.style.width = '210mm'
        new Preview({
            props: { rows },
            target: div,
        })
        document.body.appendChild(div)
        const { height, minHeight, width } = getComputedStyle(div)
        const iHeight = parseInt(height.slice(0, -2))
        const iMinHeight = parseInt(minHeight.slice(0, -2))
        const iWidth = parseInt(width.slice(0, -2))
        const maxPage = Math.ceil(iHeight / iMinHeight)
        div.style.height = `${iMinHeight * maxPage}px`
        const canvas = await html2canvas(div)
        const canvases = [...Array(maxPage)]
            .map((_, i) => cropCanvas(canvas, 0, iMinHeight * i, iWidth, iMinHeight))
        const urls = canvases.map(canvas => canvas.toDataURL())
        open(ExportPreview, { title: qapricot.meta.title, urls })
        div.remove()
    }

    function openJson () {
        const input = document.createElement('input')
        input.type = 'file'
        input.addEventListener('change', event => {
            const { target } = event
            const { files } = target
            const file = files[0]
            if (file.type === 'application/json') {
                const reader = new FileReader()
                reader.addEventListener('load', () => {
                    const { result } = reader
                    qapricot = JSON.parse(result)
                }, false)
                reader.readAsText(file)
            }
        }, false)
        input.click()
    }

    function resetRows () {
        qapricot.contents = [[createProperties()]]
    }

    function saveJson () {
        const json = JSON.stringify(qapricot, null, 2)
        const blob = new Blob([json], { type: 'application/json' })
        const url = URL.createObjectURL(blob)
        const a = document.createElement('a')
        a.download = `${qapricot.meta.title}-${Date.now().toString()}.json`
        a.href = url
        a.click()
    }

    function showHelp () {
        open(Help)
    }

    function showJsonPreview () {
        open(JsonPreview, { qapricot })
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

    function showRootPropertiesEditor () {
        open(PropertiesEditor, { v: qapricot.meta }, {}, {
            onClosed: () => {
                updateRootProperties()
            }
        })
    }

    function showSettings () {
        open(Settings, { settings }, {}, {
            onClosed: () => {
                updateSettings()
                localStorage.setItem('settings', JSON.stringify(settings))
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

    function updateRootProperties () {
        qapricot.meta = qapricot.meta
    }

    function updateRows () {
        rows = rows
    }

    function updateSettings () {
        settings = settings
    }
</script>

<main>
    <div class="flex flex-col min-h-screen" use:style={{
        backgroundColor: settings['style.root.backgroundColor'].value,
        color: settings['style.root.color'].value,
    }}>
        <div class="flex-grow">
            {#each rows as row, i}
                <div class="border m-0.5em" use:style={{
                    borderColor: settings['style.root.borderColor'].value,
                }}>
                    <div class="flex flex-col">
                        <div class="flex w-full">
                            {#each row as column, j}
                                <div class="border flex-1 m-0.5em min-w-0" use:style={{
                                    borderColor: settings['style.root.borderColor'].value,
                                }}>
                                    <div class="flex flex-col h-full">
                                        {#if column.type === 'markdown'}
                                            <div class="break-words flex-1 whitespace-pre-wrap" use:style={{
                                                textAlign: row[0].textAlign,
                                            }}>{@html marked(column.value)}</div>
                                        {:else if column.type === 'text'}
                                            <p class="break-words flex-1 whitespace-pre-wrap" use:style={{
                                                textAlign: row[0].textAlign,
                                            }}>{column.value}</p>
                                        {/if}
                                        <div class="flex overflow-auto" use:style={{
                                            backgroundColor: settings['style.navigationBar.backgroundColor'].value,
                                            color: settings['style.navigationBar.color'].value,
                                        }}>
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
                        <div class="flex justify-end" use:style={{
                            backgroundColor: settings['style.navigationBar.backgroundColor'].value,
                            color: settings['style.navigationBar.color'].value,
                        }}>
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
        <div class="bottom-0 flex justify-end sticky w-full" use:style={{
            backgroundColor: settings['style.navigationBar.backgroundColor'].value,
            color: settings['style.navigationBar.color'].value,
        }}>
            <span class="flex-1 text-center">{qapricot.meta.title}</span>
            <button class="p-0.25em" on:click={showRootPropertiesEditor} title="Properties">
                <Fa fw icon={faEdit}></Fa>
            </button>
            <button class="p-0.25em" on:click={showPreview} title="Preview">
                <Fa fw icon={faPlay}></Fa>
            </button>
            <button class="p-0.25em" on:click={saveJson} title="Save">
                <Fa fw icon={faSave}></Fa>
            </button>
            <button class="p-0.25em" on:click={openJson} title="Open">
                <Fa fw icon={faFile}></Fa>
            </button>
            <button class="p-0.25em" on:click={exportImage} title="Export…">
                <Fa fw icon={faFileExport}></Fa>
            </button>
            <button class="p-0.25em" on:click={showJsonPreview} title="JSON Preview">
                <Fa fw icon={faFileAlt}></Fa>
            </button>
            <button class="p-0.25em" on:click={showSettings} title="Settings">
                <Fa fw icon={faCog}></Fa>
            </button>
            <button class="p-0.25em" on:click={resetRows} title="Reset">
                <Fa fw icon={faEraser}></Fa>
            </button>
            <button class="p-0.25em" on:click={showHelp} title="Help">
                <Fa fw icon={faQuestion}></Fa>
            </button>
            <button class="p-0.25em" on:click={addRow} title="Add">
                <Fa fw icon={faPlus}></Fa>
            </button>
        </div>
    </div>
</main>