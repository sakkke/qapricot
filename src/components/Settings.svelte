<script>
    import PropertiesEditor from './PropertiesEditor.svelte'

    export let settings

    function importSettings () {
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
                    console.log(settings, JSON.parse(result))
                    const parsed = JSON.parse(result)
                    Object.keys(parsed).forEach(k => {
                        settings[k] = parsed[k]
                    })
                }, false)
                reader.readAsText(file)
            }
        }, false)
        input.click()
    }
</script>

<PropertiesEditor v={settings}></PropertiesEditor>
<button class="border p-0.25em" on:click={importSettings}>Import from file</button>