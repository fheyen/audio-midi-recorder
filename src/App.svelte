<script>
  import {recordAudio, recordMidi} from "musicvis-lib"
  import { Temporal } from '@js-temporal/polyfill'
  import { onMount } from "svelte"

  // Recorders
  let audioRecorder
  let midiRecorder
  // Settings
  let oldnames = localStorage.getItem("oldnames")
  ? JSON.parse(localStorage.getItem("oldnames"))
  : []
  let addDate =  localStorage.getItem("adddate") === "true"
  // Data
  let fileName = ""
  let audio
  let notes

  onMount(async ()=>{
    try {
      audioRecorder = await recordAudio()
    } catch(e) {
      console.error("Cannot enable audio recording")
      console.error(e)
    }
    try {
      midiRecorder = await recordMidi()
    } catch(e) {
      console.error("Cannot enable MIDI recording")
      console.error(e)
    }
  })


  const start = ()=>{
    console.log("start")
    midiRecorder.start()
    audioRecorder.start()
  }

  const stop = async ()=>{
    console.log("stop")
    audio = await audioRecorder.stop()
    notes = midiRecorder.stop()
  }

  const download = () => {
    const now = Temporal.Now.plainDateTimeISO().toJSON();
    const name = addDate
    ? `${fileName} ${now.substring(0, 16)
      .replace(":", "-")}`
    : fileName
    const json = {
      name: fileName,
      date: now,
      notes
    }
    downloadTextFile(JSON.stringify(json), `${name}.json`)
    downloadBlob(audio, name)
    // Store filename
    let oldnames = []
    try {
      oldnames = JSON.parse(localStorage.getItem("oldnames"))
    } catch (e) {
      console.log(e)
    }
    const oldnames2 = [...new Set([fileName, oldnames])]
    oldnames = oldnames2
    localStorage.setItem("oldnames", JSON.stringify(oldnames2))
  }

  /**
   * Download a text file
   * @param {string} text JSON content
   * @param {string} fileName file name
   */
  const downloadTextFile = (text, fileName) => {
    const a = document.createElement("a")
    a.href = "data:text/plaincharset=utf-8," + encodeURIComponent(text)
    // a.href = "data:json/plaincharset=utf-8," + encodeURIComponent(text)
    a.download = fileName
    document.body.appendChild(a)
    a.click()
    // document.removeChild(a)
  }

  /**
   * Download a blob as binary file
   * @param {Blob} blob content
   * @param {string} fileName file name
   */
  const downloadBlob=(blob, fileName) =>{
    const a = document.createElement("a")
    const url = window.URL.createObjectURL(blob)
    a.href = url
    a.download = fileName
    document.body.appendChild(a)
    a.click()
    // document.removeChild(a)
    window.URL.revokeObjectURL(url)
  }
</script>

<main>
  <div>
    <button on:click={start}>start</button>
    <button on:click={stop}>stop</button>

    <input type="text" placeholder="name" list="oldnames" title={fileName} bind:value={fileName} />
    <datalist id="oldnames">
      {#each oldnames as name}
        <option value={name}>{name}</option>
      {/each}
    </datalist>
    <button
      on:click={()=>{
        addDate = !addDate;
        localStorage.setItem("adddate", addDate?"true":"false")
      }}
    >
      {addDate? "date added to name":"date not added to name"}
    </button>

    <button
      on:click={download}
      disabled={!audio || !notes ||  fileName.length===0}
    >
      download
    </button>

    <button
      on:click={()=> {
        localStorage.removeItem("oldnames");
        oldnames = []
      }}
    >
      clear names
    </button>
  </div>
  <!-- <div>
    {#if !audioRecorder}
      audio disabled
    {/if}
    {#if !midiRecorder}
      MIDI disabled
    {/if}
  </div> -->
  <div>
    Make sure your browser supports <a href="https://caniuse.com/midi" target="_blank" rel="noreferrer">Web MIDI</a>.
    You can test your MIDI input with <a href="https://github.com/fheyen/midi-pianoroll" target="_blank" rel="noreferrer">this tool</a>.
  </div>
  <div>
    Usage: Press <i>start</i> to start recording, <i>stop</i> to stop it. Give your recording a name and download the audio and MIDI files.
  </div>
  <div>
    Source code at <a href="https://github.com/fheyen/audio-midi-recorder" target="_blank" rel="noreferrer">GitHub</a>.
  </div>
</main>
