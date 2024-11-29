<script lang="ts">
  import * as sss from 'sounds-some-sounds'
  (window as any).sss = sss
  import 'pixi.js'
  import 'pixi-filters'
  import 'crisp-game-lib'
  import { onMount } from 'svelte';

  const title: string = `TRASH`
  const description: string = `
[Tap] Turn
[Hold] Empty Trash
`

  const characters: string[] = [
    
  ]

  let smallTrashWeight: number = $state(3)
  let smallTrashPauseSeconds: number = $state(0)
  let smallTrashScore: number = $state(1)
  let smallTrashMeterPct: number = $state(5)

  let medTrashWeight: number = $state(2)
  let medTrashPauseSeconds: number = $state(1)
  let medTrashScore: number = $state(5)
  let medTrashMeterPct: number = $state(7)

  let largeTrashWeight: number = $state(1)
  let largeTrashPauseSeconds: number = $state(3)
  let largeTrashScore: number = $state(25)
  let largeTrashMeterPct: number = $state(10)

  let meterPct: number = 0

  function update() {
    // draw grass
    color("green")
    rect(0, 90, 100, 10)

    // draw meter
    //   outline
    color("black")
    rect(90, 20, 8, 52)
    //   empty part
    color("white")
    rect(91, 21, 6, 50)
    //   full part
    color("red")
    const pixelHeight = Math.floor(50 * (meterPct / 100))
    rect(91, 21 + pixelHeight, 6, pixelHeight)
  }

  onMount(() => {
    init({
      update,
      title,
      description,
      characters
    })
  })
</script>

<div id="settings-container">
  <h2>Settings</h2>
  <div id="small-trash-settings">
    <h3>Small Trash</h3>
    <label for="small-spawn-weight">Spawn Weight:</label>
    <input name="small-spawn-weight" type="number" min="0" value={smallTrashWeight}>
    <label for="small-collection-time">Collect Time (sec):</label>
    <input name="small-collection-time" type="number" min="0" value={smallTrashPauseSeconds}>
    <label for="small-score">Score:</label>
    <input name="small-score" type="number" min="0" value={smallTrashScore}>
    <label for="small-meter">Meter (%):</label>
    <input name="small-meter" type="number" min="0" max="100" value={smallTrashMeterPct}>
  </div>
  <div id="medium-trash-settings">
    <h3>Medium Trash</h3>
    <label for="medium-spawn-weight">Spawn Weight:</label>
    <input name="medium-spawn-weight" type="number" min="0" value={medTrashWeight}>
    <label for="medium-collection-time">Collect Time (sec):</label>
    <input name="medium-collection-time" type="number" min="0" value={medTrashPauseSeconds}>
    <label for="medium-score">Score:</label>
    <input name="medium-score" type="number" min="0" value={medTrashScore}>
    <label for="medium-meter">Meter (%):</label>
    <input name="medium-meter" type="number" min="0" max="100" value={medTrashMeterPct}>
  </div>
  <div id="large-trash-settings">
    <h3>Large Trash</h3>
    <label for="large-spawn-weight">Spawn Weight:</label>
    <input name="large-spawn-weight" type="number" min="0" value={largeTrashWeight}>
    <label for="large-collection-time">Collect Time (sec):</label>
    <input name="large-collection-time" type="number" min="0" value={largeTrashPauseSeconds}>
    <label for="large-score">Score:</label>
    <input name="large-score" type="number" min="0" value={largeTrashScore}>
    <label for="large-meter">Meter (%):</label>
    <input name="large-meter" type="number" min="0" max="100" value={largeTrashMeterPct}>
  </div>
</div>

<style>
  #settings-container {
    padding: 0 0.5rem;
    background: initial;
    color: initial;
    user-select: auto;
  }

  #settings-container input[type="number"] {
    width: 4rem;
  }
</style>
