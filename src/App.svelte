<script>
  import * as Tone from "tone";

  const DEFAULT_DECIBEL = -15;
  let playing = false;
  let alt = false;

  function toFreq(index) {
    // 69 => 440Hz
    return 440 * Math.pow(2, (index - 69) / 12);
  }

  class Beeper {
    constructor() {
      this.playing = false;
      this.key = 69;
      this.freq = 440;
      this.db = DEFAULT_DECIBEL;
      this.sineGain = new Tone.Gain(Tone.dbToGain(this.db)).toDestination();
      this.sine = new Tone.Oscillator(this.freq).connect(this.sineGain);
    }
    snapFreq() {
      this.key = Math.round(69 + Math.log2(this.freq / 440) * 12);
      this.updateFreq();
    }
    updateFreq() {
      this.freq = toFreq(this.key);
      this.sine.frequency.rampTo(this.freq, 0.01);
      this.db = DEFAULT_DECIBEL;
    }
    togglePlay() {
      this.playing = !this.playing;
      if (this.playing) {
        this.sine.start();
      } else {
        this.sine.stop();
      }
    }
    beep(time) {
      this.sineGain.gain.rampTo(Tone.dbToGain(this.db), 0.05, time);
      this.sineGain.gain.rampTo(0, 0.05, time + 0.5);
    }
  }

  let beeper1 = new Beeper();
  let beeper2 = new Beeper();

  let ToneLoop = new Tone.Loop((time) => {
    alt = !alt;
    if (alt) {
      beeper2.beep(time);
    } else {
      beeper1.beep(time);
    }
  }, 1);
</script>

<div class="vol-container">
  <button
    on:click={() => {
      beeper1.db += 1;
    }}>+1</button
  >
  <button
    on:click={() => {
      beeper2.db += 1;
    }}>+1</button
  >
  <button
    on:click={() => {
      beeper1.db += 0.1;
    }}>+0.1</button
  >
  <button
    on:click={() => {
      beeper2.db += 0.1;
    }}>+0.1</button
  >
</div>
<div class="control-container">
  <input
    type="range"
    bind:value={beeper1.key}
    min="16"
    max="133"
    on:input={() => {
      beeper1.updateFreq();
    }}
  />
  <input
    type="number"
    bind:value={beeper1.freq}
    on:blur={() => {
      beeper1.snapFreq();
    }}
  />
  <span>{beeper1.db - DEFAULT_DECIBEL}dB</span>
  <button
    on:click={() => {
      beeper1.togglePlay();
      beeper2.togglePlay();

      playing = !playing;
      if (playing) {
        ToneLoop.start(0);
        Tone.Transport.start();
      } else {
        ToneLoop.stop(0);
        Tone.Transport.stop();
      }
    }}>{playing ? "⏹️" : "▶️"}</button
  >
  <span>{beeper2.db - DEFAULT_DECIBEL}dB</span>
  <input
    type="number"
    bind:value={beeper2.freq}
    on:blur={() => {
      beeper2.snapFreq();
    }}
  />
  <input
    type="range"
    bind:value={beeper2.key}
    min="16"
    max="133"
    on:input={() => {
      beeper2.updateFreq();
    }}
  />
</div>
<div class="toggle-container">
  <div class:hilight={!alt} class="toggle">BASE</div>
  <div class:hilight={alt} class="toggle">ALT</div>
</div>
<div class="vol-container">
  <button
    on:click={() => {
      beeper1.db -= 0.1;
    }}>-0.1</button
  >
  <button
    on:click={() => {
      beeper2.db -= 0.1;
    }}>-0.1</button
  >
  <button
    on:click={() => {
      beeper1.db -= 1;
    }}>-1</button
  >
  <button
    on:click={() => {
      beeper2.db -= 1;
    }}>-1</button
  >
</div>

<style>
  div.vol-container button {
    flex: 0 0 50%;
  }
  input[type="range"] {
    width: 100%;
  }
  div.toggle-container {
    display: flex;
    flex-wrap: wrap;
  }
  div.vol-container {
    display: flex;
    flex-wrap: wrap;
  }
  div.control-container {
    width: 100%;
    text-align: center;
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
