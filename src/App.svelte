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
`
 ggg
gyRy
 RRR
gyyyg
RyyygR
g   g
`,
`
 l
lLL
lll
`,
`
  l
 ll
lLLLl
lllLl
`,
`
   l
   ll
 lllLl
lLLlLl
llLLLl
 lllll
`
  ]

  let smallTrashWeight: number = $state(3)
  let smallTrashPauseSeconds: number = $state(0)
  let smallTrashScore: number = $state(1)
  let smallTrashMeterPct: number = $state(5)

  function smallTrashTime() {
    return smallTrashPauseSeconds + (smallTrashMeterPct / meterEmptyRate)
  }
  function smallTrashProb() {
    return smallTrashWeight / (smallTrashWeight + medTrashWeight + largeTrashWeight)
  }

  let medTrashWeight: number = $state(2)
  let medTrashPauseSeconds: number = $state(1)
  let medTrashScore: number = $state(5)
  let medTrashMeterPct: number = $state(7)

  function medTrashTime() {
    return medTrashPauseSeconds + (medTrashMeterPct / meterEmptyRate)
  }
  function medTrashProb() {
    return medTrashWeight / (smallTrashWeight + medTrashWeight + largeTrashWeight)
  }

  let largeTrashWeight: number = $state(1)
  let largeTrashPauseSeconds: number = $state(3)
  let largeTrashScore: number = $state(25)
  let largeTrashMeterPct: number = $state(10)

  function largeTrashTime() {
    return largeTrashPauseSeconds + (largeTrashMeterPct / meterEmptyRate)
  }
  function largeTrashProb() {
    return largeTrashWeight / (smallTrashWeight + medTrashWeight + largeTrashWeight)
  }

  let initialSpawnDelay: number = $state(5)
  let maxTrash: number = $state(20)
  let powerupAdjustment: number = $state(2)

  let meterEmptyRate: number = $state(10)

  let meterPct: number = 0

  function maxMathematicalDelay() {
    const avgTime = (smallTrashProb() * smallTrashTime()) +
      (medTrashProb() * medTrashTime()) +
      (largeTrashProb() * largeTrashTime())
    return avgTime
  }

  function rateInterpolation() {
    const initialRate = initialSpawnDelay
    const adjustedMaxDelay = maxMathematicalDelay() / powerupAdjustment

    return (initialRate - ((2 / Math.PI) * (initialRate - adjustedMaxDelay) * Math.atan(4 * (difficulty - 1))))
  }

  let currentDelay: number = $state(rateInterpolation())

  type Trash = {
    size: "small" | "medium" | "large",
    id: number,
    x: number,
    y: number
  }

  let nextTrashId: number = 1
  let collectingTicksLeft: number = 0
  let currentlyCollecting: number = -1
  const gravity: number = 1

  const trashes: Trash[] = []
  let lastSpawnTick = 0

  let x = 50
  let vx: 1 | -1 = 1

  function update() {
    if(!ticks) {
      meterPct = 0
      score = 0
      nextTrashId = 1
      collectingTicksLeft = 0
      currentlyCollecting = -1
      trashes.length = 0
      x = 50
      lastSpawnTick = ticks
    }
    
    currentDelay = rateInterpolation()

    // draw grass
    color("light_green")
    rect(0, 80, 100, 20)
    // draw pollution
    color("light_yellow")
    rect(0, 80, 100, trashes.length)
    color("black")

    const ticksToSpawn = 60 * currentDelay
    if((ticks - lastSpawnTick) >= ticksToSpawn) {
      if(trashes.length >= 20) end("Too Much Trash!")

      lastSpawnTick = ticks

      let spawnX = (Math.random() * (97 - 3)) + 3
      let spawnY = 60

      let roll = Math.floor(Math.random() * (smallTrashWeight + medTrashWeight + largeTrashWeight))
      let trashType: "small" | "medium" | "large";
      if(roll < smallTrashWeight) trashType = "small"
      else if(roll < (smallTrashWeight + medTrashWeight)) trashType = "medium"
      else trashType = "large"

      const ourId = nextTrashId
      nextTrashId++

      trashes.push({
        size: trashType,
        id: ourId,
        x: spawnX,
        y: spawnY
      })
    }

    char("a", x, 78, {
        mirror: { x: vx }
    })

    remove(trashes, trash => {
      let ticksNeededToSpend: number;
      let character: string;
      let meterToAdd: number;
      let scoreToAdd: number;
      switch(trash.size) {
        case 'small':
          ticksNeededToSpend = 60 * smallTrashPauseSeconds
          character = "b"
          meterToAdd = smallTrashMeterPct
          scoreToAdd = smallTrashScore
          break
        case 'medium':
          ticksNeededToSpend = 60 * medTrashPauseSeconds
          character = "c"
          meterToAdd = medTrashMeterPct
          scoreToAdd = medTrashScore
          break
        case 'large':
          ticksNeededToSpend = 60 * largeTrashPauseSeconds
          character = "d"
          meterToAdd = largeTrashMeterPct
          scoreToAdd = largeTrashScore
          break
      }

      trash.y += gravity
      if(trash.y > 79) trash.y = 79

      let collision = char(character, trash.x, trash.y)

      if(currentlyCollecting === -1 && collision.isColliding.char!["a"]) {
        currentlyCollecting = trash.id
        collectingTicksLeft = ticksNeededToSpend
      }

      if(currentlyCollecting === trash.id) {
        let collBarStart = trash.x - 3
        let collBarEnd = trash.x + 3
        let collBarY = trash.y - 5
        let collBarProgress = (collBarEnd - collBarStart) * (1 - (collectingTicksLeft / ticksNeededToSpend))

        color("purple")
        rect(collBarStart, collBarY, (collBarEnd - collBarStart), 1)
        color("light_purple")
        rect(collBarStart, collBarY, collBarProgress, 1)
        color("black")

        if(input.isPressed) {
          meterPct -= (meterEmptyRate / 60)
          meterPct = Math.max(0, meterPct)
        } else {
          if(collectingTicksLeft == 0) {
            meterPct += meterToAdd
            if(meterPct > 100) end("Meter Full!")
            currentlyCollecting = -1
            score += scoreToAdd
            return true;
          } else {
            collectingTicksLeft--
          }
        }
      }
    })

    if(currentlyCollecting === -1) {
      if(input.isJustReleased) {
          // @ts-ignore
          vx = -vx
      }
      if(x >= 97) vx = -1
      if(x <= 3) vx = 1

      if(input.isPressed) {
        meterPct -= (meterEmptyRate / 60)
        meterPct = Math.max(0, meterPct)
      } else {
        x += vx
      }
    }

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
    rect(91, 71 - pixelHeight, 6, pixelHeight)
    color("black")
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
    <input name="small-spawn-weight" type="number" min="0" bind:value={smallTrashWeight}>
    <label for="small-collection-time">Collect Time (sec):</label>
    <input name="small-collection-time" type="number" min="0" bind:value={smallTrashPauseSeconds}>
    <label for="small-score">Score:</label>
    <input name="small-score" type="number" min="0" bind:value={smallTrashScore}>
    <label for="small-meter">Meter (%):</label>
    <input name="small-meter" type="number" min="0" max="100" bind:value={smallTrashMeterPct}>
  </div>
  <div id="medium-trash-settings">
    <h3>Medium Trash</h3>
    <label for="medium-spawn-weight">Spawn Weight:</label>
    <input name="medium-spawn-weight" type="number" min="0" bind:value={medTrashWeight}>
    <label for="medium-collection-time">Collect Time (sec):</label>
    <input name="medium-collection-time" type="number" min="0" bind:value={medTrashPauseSeconds}>
    <label for="medium-score">Score:</label>
    <input name="medium-score" type="number" min="0" bind:value={medTrashScore}>
    <label for="medium-meter">Meter (%):</label>
    <input name="medium-meter" type="number" min="0" max="100" bind:value={medTrashMeterPct}>
  </div>
  <div id="large-trash-settings">
    <h3>Large Trash</h3>
    <label for="large-spawn-weight">Spawn Weight:</label>
    <input name="large-spawn-weight" type="number" min="0" bind:value={largeTrashWeight}>
    <label for="large-collection-time">Collect Time (sec):</label>
    <input name="large-collection-time" type="number" min="0" bind:value={largeTrashPauseSeconds}>
    <label for="large-score">Score:</label>
    <input name="large-score" type="number" min="0" bind:value={largeTrashScore}>
    <label for="large-meter">Meter (%):</label>
    <input name="large-meter" type="number" min="0" max="100" bind:value={largeTrashMeterPct}>
  </div>
  <div id="all-trash-settings">
    <h3>All Trash Settings</h3>
    <label for="initial-spawn-delay">Starting Delay (sec):</label>
    <input name="initial-spawn-delay" type="number" min="0" bind:value={initialSpawnDelay}>
    <label for="trash-spawn-limit">Max Trash:</label>
    <input name="trash-spawn-limit" type="number" min="1" bind:value={maxTrash}>
    <label for="powerup-adjustment" title="For example, 1.1 would mean that powerups allow for trash to in practice fall 10% faster than the mathematical limit">Powerup Adjustment:</label>
    <input name="powerup-adjustment" type="number" min="0" bind:value={powerupAdjustment}>
  </div>
  <div id="meter-powerup-settings">
    <h3>Meter/Powerup Settings</h3>
    <label for="meter-empty-rate">Meter Empty Rate (%/s):</label>
    <input name="meter-empty-rate" type="number" min="0" bind:value={meterEmptyRate}>
  </div>
  <div id="stats">
    <h3>Stats</h3>
    <div>
      Delay Between Trash: {currentDelay}
    </div>
    <div>
      Time Until Next Trash: {currentDelay - ((ticks - lastSpawnTick) / 60.0)}
    </div>
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
