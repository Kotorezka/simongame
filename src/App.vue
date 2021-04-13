<template>
  <div id="app">
  <audio ref="sound1" src="https://s3.amazonaws.com/freecodecamp/simonSound1.mp3"></audio>
  <audio ref="sound2" src="https://s3.amazonaws.com/freecodecamp/simonSound2.mp3"></audio>
  <audio ref="sound3" src="https://s3.amazonaws.com/freecodecamp/simonSound3.mp3"></audio>
  <audio ref="sound4" src="https://s3.amazonaws.com/freecodecamp/simonSound4.mp3"></audio>
  <div id="simon">
    <div id="center">
      <div id="title">
        <h1>Simon</h1>
      </div>
      <div id="controls">
        <a id="start" class="btn" @click="start"></a>
        <div id="counter">
          <span :class="{lit: power}" v-text="showError ? '! !' : (showWin ? '* *' : displayCount)"></span>
        </div>
        <a id="strict" class="btn" :class="{strict: strict}" @click="toggleStrict"></a>
      </div>
      <div id="power">
        <p>Off</p>
        <div id="switch" :class="{on: power}" @click="togglePower"></div>
        <p>On</p>
      </div>
    </div>
    <div id="buttons">
      <div class="button" :class="{highlight: hlGreen}" id="green" @click="input(1)"></div>
      <div class="button" :class="{highlight: hlRed}" id="red" @click="input(2)"></div>
      <div class="button" :class="{highlight: hlYellow}" id="yellow" @click="input(3)"></div>
      <div class="button" :class="{highlight: hlBlue}" id="blue" @click="input(4)"></div>
    </div>
  </div>
  <select v-model="level" name="level" id="levelSelector">
    <option v-for="level in levels" :value="level" :key="level.id">{{ level }}</option>
 </select>  
</div>
</template>

<script>


export default {
  data () {
    return{
    level: null,
    levels: [
      'Easy','Medium', 'Hard'
    ], 
    power: false,   // Is the power turned on?
    started: false, // Has the game started?
    strict: false,  // Is strict mode enabled?
    count: 0,       // What's the current count?
    series: [],     // The current series of tones
    playingSeries: false, // Is the game currently outputting the series of tones?
    userInput: [],
    hlRed: false,
    hlGreen: false,
    hlYellow: false,
    hlBlue: false,
    allowInput: false,
    showError: false,
    showWin: false,
    totalToWin: 20,
  }},
  computed: {
    displayCount() {
      if (this.count == 0)
        return "- -"
      else
        return this.count
    }
  },
  methods: {
    reset() {
      // Reset the game state
      this.started = false
      this.count = 0
      this.series = []
      this.userInput = []
      this.allowInput = false
      this.playingSeries = false
      this.showError = false
      this.showWin = false
      this.hlGreen = false
      this.hlRed = false
      this.hlYellow = false
      this.hlBlue = false
    },
    // This method executes when the user wins the game
    winner() {
      this.showWin = true
      let self = this
      let levelSelected = {
        "Easy" : 1500,
        'Medium': 1000,
        'Hard': 400
      };
      let delay = levelSelected[this.level];
      console.log(delay);
      this.hlGreen = true; 
      for (let x = 1; x <= 6; x++) {
        setTimeout(function() { self.playTone(1) }, delay)
        if (x == 6)
          setTimeout(function() { self.hlGreen = false; self.showWin = false; self.reset() }, delay + 500)
        delay += 500
      }
    },
    input(tone) {
      if (!this.allowInput)
        return
      this.playTone(tone)
      this.userInput.push(tone)
      // Check if this was the wrong input
      if (this.userInput[this.userInput.length - 1] != this.series[this.userInput.length - 1]) {
        this.userInput = []
        this.allowInput = false
        let self = this
        this.showError = true
        // Check if we are in strict mode
        if (this.strict) {
          // Strict mode enabled & wrong entry -> end the game
          setTimeout(this.reset, 1000)
          setTimeout(this.start, 2000)
        } else
          setTimeout(function() { self.showError = false; self.playSeries() }, 1000)
        return
      }
      // If this was the final input
      if (this.userInput.length == this.series.length) {
        let self = this
        this.userInput = []
        // Check if we won (20 correct inputs)
        if (this.series.length == this.totalToWin) {
          // User has won the game
          setTimeout(this.winner, 500)
        } else {
          setTimeout(function() { 
            self.addTone()
            self.playSeries()
          }, 1000)
        }
      }
    },
    togglePower() {
      // Toggle power
      this.power = !this.power
      // Reset if power is turned off
      if (!this.power) {
        this.reset()
        this.strict = false
        this.stopTones()
      }
    },
    toggleStrict() {
      if (this.power)
        this.strict = !this.strict
    },
    start() {
      if (this.power && this.count == 0) {
        this.started = true
        this.addTone()
        this.playSeries()
      }
    },
    addTone() {
      this.count++
      this.series.push(this.randomTone())
    },
    playSeries() {
      this.allowInput = false
      this.playingSeries = true
      let self = this
      let levelSelected = {
        "Easy" : 1500,
        'Medium': 1000,
        'Hard': 400
      };
      let delay = levelSelected[this.level];
      console.log(delay);
      this.series.forEach(function(tone, index, array) {
        if (index == array.length - 1)
          setTimeout(function() { 
            if (self.started) {
            self.playTone(tone)
            self.allowInput = true
            self.playingSeries = false
            }
          }, delay)
        else
          setTimeout(function() { self.playTone(tone) }, delay)
        delay += 1000
      })
      this.playingSeries = false
    },
    clearHighlights() {
      this.hlGreen = false
      this.hlRed = false
      this.hlYellow = false
      this.hlBlue = false
    },
    // Plays the tone & highlights the color
    playTone(tone) {
      if (!this.power)
        return
      switch (tone) {
        case 1:
          this.$refs.sound1.pause()
          this.$refs.sound1.currentTime = 0
          this.$refs.sound1.play()
          this.hlGreen = true
          break;
        case 2:
          this.$refs.sound2.pause()
          this.$refs.sound2.currentTime = 0
          this.$refs.sound2.play()
          this.hlRed = true
          break;
        case 3:
          this.$refs.sound3.pause()
          this.$refs.sound3.currentTime = 0
          this.$refs.sound3.play()
          this.hlYellow = true
          break;
        case 4:
          this.$refs.sound4.pause()
          this.$refs.sound4.currentTime = 0
          this.$refs.sound4.play()
          this.hlBlue = true
          break;
      }
      if (!this.showWin)
        setTimeout(this.clearHighlights, 750)
    },
    stopTones() {
      this.$refs.sound1.pause()
      this.$refs.sound2.pause()
      this.$refs.sound3.pause()
      this.$refs.sound4.pause()
      this.$refs.sound1.currentTime = 0
      this.$refs.sound2.currentTime = 0
      this.$refs.sound3.currentTime = 0
      this.$refs.sound4.currentTime = 0
    },
    randomTone() {
      return Math.floor(Math.random() * 4) + 1
    }
  }
}
</script>

<style>
body {
  font-family: 'Open Sans', sans-serif;
  user-select: none;
  background-color: white;
}
h1 {
  font-family: 'Russo One', sans-serif;
  font-size: 3rem;
  cursor: default;
  margin: 1rem 0;
}
#simon {
  margin: 2rem auto 0 auto;
  max-width: 680px;
  min-width: 680px;
  max-height: 680px;
  min-height: 680px;
  background-color: black;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  box-shadow: 0px 0px 18px rgba(0,0,0,.6)
}
#levelSelector {
  margin: 2rem auto 0 auto;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  box-shadow: 0px 0px 18px rgba(0,0,0,.6)
}
#buttons {
  display: flex;
  flex-wrap: wrap;
  max-width: 620px;
  min-width: 620px;
  justify-content: space-between;
}
.button {
  cursor: pointer;
  min-width: 300px;
  min-height: 300px;
  max-width: 300px;
  max-height: 300px;
}
#green { 
  background-color: green; 
  border-top-left-radius: 300px;
  margin-bottom: 20px;
  box-shadow: inset 3px 3px 10px rgba(255,255,255,.3);
}
#blue { 
  background-color: blue; 
  border-bottom-right-radius: 300px;
  box-shadow: inset -3px -3px 10px rgba(255,255,255,.3);
}
#yellow { 
  background-color: yellow; 
  border-bottom-left-radius: 300px;
  box-shadow: inset 3px -3px 10px rgba(255,255,255,.3);
}
#red { 
  background-color: red; 
  border-top-right-radius: 300px;
  margin-bottom: 20px;
  box-shadow: inset -3px 3px 10px rgba(255,255,255,.3);
}
#green.highlight { background-color:rgba(0,255,0,0.5); }
#red.highlight { background-color: rgba(255,0,0,0.5); }
#blue.highlight { background-color: rgba(0,0,255,0.5); }
#yellow.highlight { background-color: rgba(255,255,0,0.5); }


#center {
  border: 20px solid black;
  box-sizing: border-box;
  position: absolute;
  min-width: 300px;
  max-width: 300px;
  min-height: 300px;
  max-height: 300px;
  border-radius: 50%;
  background-color: white;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  color: black;
}
#title {
  margin-top: 20px;
  text-align: center;
  background-color: white;
  width: 200px;
  border-top-left-radius: 50%;
  border-top-right-radius: 50%;
}
#controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  min-width: 180px;
  margin-top: .5rem;
}
#counter {
  display: inline-block;
  width: 60px;
  height: 40px;
  color: #ccc;
  background-color: black;
  border: 4px solid #666;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  font-size: 1.6rem;
}
.lit { color: red; }
#power {
  margin-top: 1rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 110px;
  font-size: .875rem;
  text-transform: uppercase;
  font-weight: bold;
}
#switch { 
  cursor: pointer;
  display: inline-block; 
  width: 50px;
  height: 25px;
  background-color: #666;
  position: relative;
}
#switch:after {
  content: '';
  position: absolute;
  top: 0;
  height: 19px;
  width: 19px;
  border: 3px solid #666; 
  background-color: red;
}
#switch.off:after { left: 0 }
#switch.on:after { right: 0 }
.btn {
  cursor: pointer;
  position: relative;
  display: inline-block;
  width: 24px;
  height: 24px;
  background-color: yellow;
  border-radius: 50%;
  border: 4px solid #666;
  margin-top: 1rem;
  box-shadow: 2px 2px 3px rgba(0,0,0,.4);
}
.btn::after {
  position: absolute;
  top: -20px;
  left: -25%;
  font-size: 12px;
  text-transform: uppercase;
  font-weight: bold;
}
.btn:hover { background-color: rgba(255,255,0,0.5); }
#start.btn { background-color: red; }
#start.btn:hover { background-color: rgba(255,0,0,0.5); }
#strict.btn::before {
  position: absolute;
  content: '';
  top: -34px;
  left: 7px;
  display: inline-block;
  width: 6px;
  height: 6px;
  border: 2px solid #666;
  border-radius: 50%;
  background-color: black;
}
#strict.strict.btn::before { background-color: yellow; }
#start::after { content: 'Start'; }
#strict::after { content: 'Strict'; }

footer {
  padding: 3rem 0;
  text-align: center;
  font-size: .875rem;
}
</style>
