<script>
  import * as Tone from "tone";

  const samples = {
    kick: "kick.wav",
    snare: "snare.wav",
    hihat: "hat.wav",
    clap: "clap.wav",
  };

  let stepSequencer = Array(4)
    .fill(null)
    .map(() => Array(8).fill(false));

  const bootsAndPants = [[0,3],[4],[2,6],[4]];

  bootsAndPants.forEach((onsetArray, i)=>{
    onsetArray.forEach(onset=>{
      stepSequencer[i][onset] = true;
    })
  })
  
  let sampler = new Tone.Sampler({
    urls: {
      C1: samples.kick,
      D1: samples.snare,
      E1: samples.hihat,
      F1: samples.clap,
    },
    baseUrl: "",
    onload: () => console.log("Samples loaded!"),
  }).toDestination();

  let index = 0;

  let userStarted = false;
  let playing = false;
  function startStop() {
    if (userStarted == false) {
      userStarted = true;
      Tone.start()
        .then(() => {
          playing = true;
          console.log("AudioContext started successfully");
          Tone.Transport.start();
          Tone.Transport.bpm.value = 120;
          Tone.Transport.scheduleRepeat((time) => {
            stepSequencer.forEach((part, partIndex) => {
              if (part[index]) {
                const note = ["C1", "D1", "E1", "F1"][partIndex];
                sampler.triggerAttackRelease(note, "8n", time);
              }
            });
            index = (index + 1) % stepSequencer[0].length;
          }, "8n");
        })
        .catch((err) => console.error("Error starting AudioContext:", err));
    } else {
      if (playing == false) {
        playing = true;
        index = 0;
        Tone.start() && Tone.Transport.start();
      } else if (playing == true) {
        playing = false;
        index = -1;
        index = 0;
        Tone.Transport.stop();
      }
    }
  }

  function toggleStep(part, step) {
    stepSequencer[part][step] = !stepSequencer[part][step];
  }

  function setColor(stepIndex, index, step, element) {
    let style = "";
    let steps = stepSequencer[0].length;
    if (playing && stepIndex == (steps + index - 1) % 8) {
      if (step) {
        style = style = `background: hsl(${
          240 + (120 * stepIndex) / 8
        } 100% 35%); transition: ease .001s;`;
      } else {
        style = `background: hsl(${
          240 + (120 * stepIndex) / 8
        } 50% 20%); transition: ease .001s;`;
      }
    } else {
      if (step) {
        style = "background: white; color: black; transition: ease .001s;";
      }
    }
    return style;
  }
</script>

<div class="interface">
  {#each stepSequencer as part, partIndex}
    <div class="part">
      {#each part as step, stepIndex}
        <button
          class="square"
          style={setColor(stepIndex, index, step, this)}
          on:click={() => toggleStep(partIndex, stepIndex)}
        >
        </button>
      {/each}
    </div>
  {/each}
  <div class="controls">
    <button on:click={() => startStop()}>{playing ? "Click to Stop" : "Click to Start"}</button>
  </div>
</div>

<style>
  button {
    transition: ease-out 0.5s;
    font-weight: bold;
  }
  .square {
    aspect-ratio: 1/1;
    flex-grow: 1;
  }
  .part,
  .controls {
    display: flex;
    gap: 0.5rem;
  }
  .controls > button {
    flex-grow: 1;
    font-size: 2rem;
  }
  .interface {
    display: grid;
    grid-template-rows: repeat(5, 1fr);
    gap: 0.5rem;
    padding: 0rem;
  }
  button:active {
    background: white;
  }
</style>
