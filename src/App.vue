<template>
  <v-app>

    <v-layout>
      <!-- AppBar-->
      <v-app-bar absolute dense color="primary">
        <v-app-bar-nav-icon></v-app-bar-nav-icon>
        <v-toolbar-title>Play with me - Darbuka</v-toolbar-title>
        
      </v-app-bar>

      <!-- NavBar -->
      <v-navigation-drawer
        floating
        permanent
      >
        <!-- tempo -->
        <div class="text-caption">{{Math.round(bpm)}} BPM</div>
        <v-slider
          min="40"
          max="240"
          v-model="bpm"
          prepend-icon="mdi-volume-high"
        ></v-slider>

        <!-- play/pausr-->
        <v-btn v-if="isPlaying" prepend-icon="mdi-pause" color="primary" @click="pausePlay">
          Pause
        </v-btn>
        <v-btn v-else prepend-icon="mdi-play" color="primary" @click="startPlay">
          Play
        </v-btn>

        <!-- add line -->
        <v-combobox
          label="AddLine"
          v-model="presetSelected"
          :items="Object.keys(presets)"
          @update:model-value="addLine"
        ></v-combobox>

      </v-navigation-drawer>
      <v-main>

        <!-- Cards -->
        <v-card v-for="item, k in sequence" :key="k">
          <v-card-subtitle>Repeat {{ item.repeat }}</v-card-subtitle>
          <v-row v-for="line, j in chunk(item.pattern,8)" :key="j">
            <v-col v-for="item, i in line" :key="i">
              <v-text-field
                :value="item"
                placeholder="*"
                :variant="counter.sequence == k && counter.pattern == j*8 + i ? 'solo' : 'default'"
              ></v-text-field>
            </v-col>
            <v-col>
              <v-btn
                v-if="j==0"
                key="delete"
                :id="k"
                icon="mdi-delete"
                color="error"
                @click="deleteLine"
              ></v-btn>
            </v-col>
          </v-row>
        </v-card>
          
      </v-main>
    </v-layout>
  </v-app>
</template>

<script>
import presetsData from "./assets/rythems.json"
export default {
  name: 'App',

  components: {
  },

  data: () => ({
    isPlaying: false,
    bpm: 90,
    myInterval: null,
    presets: presetsData,
    presetSelected: null,
    sequence: [
      { repeat: 1, pattern:["D", "T", "", "T", "D", "", "T", ""]},
      { repeat: 1, pattern:["T", "D", "", "D", "T", "", "D", ""]},
    ],
    sounds: {
      "D": new Audio(require('./assets/dum.mp3')),
      "T": new Audio(require('./assets/tek.mp3')),
      "1": new Audio(require('./assets/one.mp3')),      
    },
    counter: { sequence:0, repeat:0, pattern:0, note:0 }
  }),
  methods: {
    chunk(inputArray, chunkSize){
      // split array into
      var R = [];
      for (var i = 0; i < inputArray.length; i += chunkSize)
        R.push(inputArray.slice(i, i + chunkSize));
      return R;
    },

    tick(){
      const next = this.nextStep()
      this.playNotes(next)
      if (this.isPlaying)
          setTimeout(this.tick, 30000 / this.bpm)
    },

    nextStep(){
      // find next to play
      this.counter.pattern++
      if (this.counter.pattern>=this.sequence[this.counter.sequence].pattern.length){
        this.counter.sequence++
        this.counter.pattern=0
      }
      if (this.counter.sequence>=this.sequence.length) {
        this.counter.sequence=0
      }
      return this.sequence[this.counter.sequence].pattern[this.counter.pattern]
    },

    async playNotes(notes){
      for (const item in self.notes) {
        if (!this.sounds[item].paused){
          this.sounds[item].pause()
          this.sounds[item].currentTime = 0
        }
      }
      for (let i = 0; i < notes.length; i++) {
        const item = notes[i];
        await new Promise((resolve) => {
          if (item === "") {
          // insert desired number of milliseconds to pause here
          setTimeout(resolve, 250);
          } else {
            this.sounds[item].onended = resolve;
            this.sounds[item].play();
          }
        });
      }
    },

    startPlay(){
      this.isPlaying = true
      this.counter = { sequence:0, repeat:0, pattern:-1 }
      // this.myInterval = setInterval(this.tick, 30000 / this.bpm);
      setTimeout(this.tick, 30000 / this.bpm)
    },

    pausePlay(){
      this.isPlaying = false
      this.counter.pattern = -1 
      // clearInterval(this.myInterval);
    },

    addLine(value){
      this.sequence.push({ repeat: 1, pattern:this.presets[value]})
      this.presetSelected = ""
    },

    deleteLine(event){
      this.sequence.splice(event.currentTarget.id, 1)
    }
  }
}
</script>
