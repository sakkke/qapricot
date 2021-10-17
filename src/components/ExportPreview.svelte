<script>
    import { faFileImage } from '@fortawesome/free-solid-svg-icons'
    import Fa from 'svelte-fa/src/fa.svelte'

    export let title
    export let urls

    function downloadFile (href, download) {
        const a = document.createElement('a')
        a.download = download
        a.href = href
        a.click()
    }
</script>

<div class="flex flex-wrap">
    {#each urls as url, i}
        <div class="even:pl-0.25em flex flex-col odd:pr-0.25em p-0.5em w-1/2">
            <img alt={i + 1} class="shadow" src={url}>
            <div class="flex justify-end mt-0.5em">
                <span class="flex-1 text-center">{i + 1}</span>
                <button class="p-0.25em" on:click={() => downloadFile(
                    url,
                    `${title}-${i + 1}-${Date.now().toString()}.png`
                )} title="Export as image">
                    <Fa fw icon={faFileImage}></Fa>
                </button>
            </div>
        </div>
    {/each}
</div>