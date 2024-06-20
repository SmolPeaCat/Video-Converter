<script lang="ts">
    // TODO add a way to convert images to gif
	import { FFmpeg } from '@ffmpeg/ffmpeg';
    import {tweened} from "svelte/motion"
    import {fade} from 'svelte/transition'
    import {confetti} from '@neoconfetti/svelte'
	import { onMount } from 'svelte';

    type State = 'loading' | 'loaded' | 'convert.start' | 'convert.error' | 'convert.done'

    let state: State = 'loading'
    let error = ""
    let ffmpeg: FFmpeg
    let progress = tweened(0)

    function downloadVideo(data: Uint8Array, type: string) {
        const a = document.createElement('a')
        a.href = URL.createObjectURL(new Blob([data.buffer], {type: 'video/mp4'}))
        a.download = `video.${type}`

        setTimeout(() => {
            a.click()
        }, 1000)
    }
    // Create an Array of bytes from the file
    async function readFile(file: File): Promise<Uint8Array> {
        return new Promise(resolve => {
            const fileReader = new FileReader()

            fileReader.onload= () => {
                const {result} = fileReader
                if (result instanceof ArrayBuffer) {
                    resolve(new Uint8Array(result))
                }
            }
            fileReader.onerror = () => {
                error = 'COuld not read file'

            }

            fileReader.readAsArrayBuffer(file)

        })

    }

    async function convertVideo(video: File) {
        state = 'convert.start'
        const videoData = await readFile(video)
        await ffmpeg.writeFile('input.webm', videoData)
        await ffmpeg.exec(['-i','input.webm','output.mp4'])
        const data   = await ffmpeg.readFile('output.mp4')
        state = 'convert.done'
        return data as Uint8Array  
    }

    async function convertImages(imgs: FileList) {
        state = 'convert.start'
        const len: number = imgs.length
        // Write each file in memory
        for (let i = 0; i < imgs.length; i++) {
            const imageData = await readFile(imgs[i])
            await ffmpeg.writeFile(`input${i}.png`, imageData)
        }

        console.log('input files created')
        // Use ffmpeg to transform the images into a gif
        await ffmpeg.exec(['-framerate', `${calculateFramerate(len,len)}`,
                            '-i', `input%d.png`,
                            'output.gif'])
        console.log('output gif created');

        // -- checks what is written in memory
        //console.log(ffmpeg.listDir('./'))

        const data = await ffmpeg.readFile('output.gif')
        console.log('data created');
        state = 'convert.done'
        return data as Uint8Array
    }

    function checkFilesType(files: FileList) {
        for (let i = 0; i< files.length; i++) {
            if (files[i].type != "image/png") 
            {
                error = "Not all files given are pngs"; 
                return;
            }
        }
        return true
    }

    function calculateFramerate(numImages: number, duration: number) {
        return Math.ceil(numImages/duration)
    }

    async function handleDrop(event: DragEvent) {
        const data = event.dataTransfer
        if (!data) return

        const files = data.files
        console.log(files)
        if (files.length > 1 && checkFilesType(files)) {
            error = ""
            const imagesList: FileList =  event.dataTransfer.files
            
            const data = await convertImages(imagesList)
            downloadVideo(data, 'gif')
            

        }

        else if ( ["video/webm","video/x-matroska"].includes(files[0].type)) {
            error = ""
            console.log(files)
            const [file] = event.dataTransfer.files
            const data = await convertVideo(file)
            downloadVideo(data, 'mp4')

        } else if (error != "") {
            error = "Only WebM and png image sequence are supported"
        }
    }

    async function loadFFmepg() {
        const baseURL = 'https://unpkg.com/@ffmpeg/core@0.12.6/dist/esm'

        ffmpeg = new FFmpeg()

        ffmpeg.on('progress', (event) => {
            $progress = event.progress * 100
        })

        await ffmpeg.load({
            coreURL: `${baseURL}/ffmpeg-core.js`,
            wasmURL: `${baseURL}/ffmpeg-core.wasm`,
        })

        state = 'loaded'
    }

    onMount(async () => {
        loadFFmepg()
    })
    $: console.log(state)
</script>

<h1 class="title">Online video converter</h1>



<!-- svelte-ignore a11y-no-static-element-interactions -->
<div
on:drop|preventDefault={handleDrop}
on:dragover|preventDefault={() => {}}
data-state = {state}
class= "drop"
>
{#if state == 'loading'}
    <p in:fade>Loading FFmpeg...</p>
{/if}

{#if state == 'loaded'}
    <p in:fade> Drag video here</p>
{/if}

{#if state == 'convert.start'}
    <p in:fade>Converting video</p>
    <div class="progress-bar">
        <div class="progress" style:--progress="{$progress}%" >
            {$progress.toFixed(0)}%
        </div>

    </div>
{/if}

{#if state == 'convert.done'}
    <div use:confetti>
        <p in:fade >Done !</p>
    </div>
{/if}

{#if error}
    <p in:fade class="error">{error}</p>
{/if}




</div>

<style>

    .title {
        text-align: center;
    }

    .drop {
        width: 600px;
        height: 400px;
        display: grid;
        place-content: center;
        margin-block-start: 2rem;
        border: 10px dashed hsl(220, 10%, 20%);
        border-radius: 8px;

        & p {
            font-size: 2rem;
            text-align: center;

            &.error {
                background-color: hsl(9, 100%, 64%);
                border-radius: 10px;
            }
        }

    }

    .progress-bar {
        --progress-bar-clr: hsl(180, 100%, 50%)
        --progress-txt-clr: hsl(0, 0%, 0%)

        width: 300px;
        height: 40px;
        position: relative;
        font-weight: 700;
        background-color: hsl(200 10% 14%);
        border-radius: 8px;

        & .progress {
            width: var(--progress);
            height: 100%;
            position: absolute;
            left: 0px;
            display: grid;
            place-content: center;
            background-color: var(--progress-bar-clr);
            color: var(--progress-txt-clr);
            border-radius: 8px;
        }
    }
</style>
