<script>
  import * as Tone from "tone";

  const DEFAULT_DECIBEL = -15;
  let playing = false;
  let key = 69;
  let freq = 440;
  let db = DEFAULT_DECIBEL;
  let alt = false;

  const noiseGain = new Tone.Gain(Tone.dbToGain(db))//.toDestination();
  const noiseFilter = new Tone.Filter({
    frequency: freq,
    type: "bandpass",
    // Q: 10,
  }).connect(noiseGain);
  const noise = new Tone.Noise("pink").connect(noiseFilter);
  const sineGain = new Tone.Gain(Tone.dbToGain(db)).toDestination();
  const sine = new Tone.Oscillator(freq).connect(sineGain);
  function toFreq(index) {
    // 69 => 440Hz
    return 440 * Math.pow(2, (index - 69) / 12);
  }
  function snapFreq() {
    key = Math.round(69 + Math.log2(freq / 440) * 12);
    updateFreq();
  }
  function updateFreq() {
    freq = toFreq(key);
    sine.frequency.rampTo(freq, 0.01);
    noiseFilter.frequency.rampTo(freq, 0.01);
  }
  function togglePlay() {
    playing = !playing;
    if (playing) {
      noise.start();
      sine.start();
      new Tone.Loop((time) => {
        alternateGain(time);
      }, 2).start(0);
      Tone.Transport.start();
    } else {
      noise.stop();
      sine.stop();
      Tone.Transport.stop();
    }
  }
  function alternateGain(time) {
    let gain = Tone.dbToGain(alt ? DEFAULT_DECIBEL : db);
    sineGain.gain.rampTo(gain, 0.01, time);
    noiseGain.gain.rampTo(gain, 0.01, time);
    sineGain.gain.rampTo(0, 0.1, time + 1);
    noiseGain.gain.rampTo(0, 0.1, time + 1);
    alt = !alt;
  }
  function changeVolume(change) {
    db += change;
  }
</script>

<div class="vol">
  <button on:click={() => changeVolume(1)}>+1</button>
  <button on:click={() => changeVolume(0.1)}>+0.1</button>
  <button on:click={() => changeVolume(0.01)}>+0.01</button>
</div>
<div>
  <input
    type="range"
    bind:value={key}
    min="16"
    max="133"
    on:input={updateFreq}
  />
  <input type="number" bind:value={freq} on:blur={snapFreq} />
  <button on:click={togglePlay}>
    {playing ? "⏹️" : "▶️"}
  </button>
  <span>{db}dB</span>
</div>
<div class="toggle-container">
  <div class:hilight={!alt} class="toggle">BASE</div>
  <div class:hilight={alt} class="toggle">ALT</div>
</div>
<div class="vol">
  <button on:click={() => changeVolume(-0.01)}>-0.01</button>
  <button on:click={() => changeVolume(-0.1)}>-0.1</button>
  <button on:click={() => changeVolume(-1)}>-1</button>
</div>

<style>
  div.vol button {
    width: 100%;
  }
  input[type="range"] {
    width: 100%;
  }
  div.toggle-container {
    display: flex;
  }
  div.toggle {
    flex: 0 0 50%;
    padding: 1em;
  }
  div.hilight {
    border-radius: 0.5em;
    background-color: rgb(192, 224, 255);
  }
  * {
    margin: 0;
    box-sizing: border-box;
  }
</style>
