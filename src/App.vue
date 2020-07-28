<template>
  <div id="app">
    <div class="simon">
      <h2 class="simon__header">{{ messageToPlayer }}</h2>
      <span class="simon__action"> {{ actionNow }} </span>
      <div class="simon__field">
        <div class="simon__button" v-for="button in buttons" 
          :key="button.id"
          :class="{ lightenUp: (currentLighten === button.id || clickedId === button.id) }"
          @click="repeatSequence(button)"
        >
        </div>
      </div>
      <div class="simon__controls">
        <div class="simon__difficulty">
          <label class="simon__radio-label" v-for="mod in difficulty"
            :class ="{disabled: gameStarted}"
            :key="mod.id"
            :for="mod.id"
          >
            <input  type="radio" name="difficulty"
              :disabled="gameStarted"
              :id="mod.id" 
              :checked="mod.checked"
              @click="changeDifficulty(mod.interval)"
            >
            {{ mod.text }}
          </label>
        </div>
        <button class="simon__start"
          :class ="{disabled: gameStarted}"
          @click="startGame"
        >Начать</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data: function() {
    return {
      messageToPlayer: 'Начните игру',
      actionNow: '',

      gameStarted: false,
      sequenceShown: false,
      level: 0,
      difficultyMod: 1500,

      clickedId: null,
      currentLighten: null,

      sequences: {
        game: [],
        player: [],
      },
      currentStep: 0,

      buttons: [
        { id: 0, sound: 'soundA' },
        { id: 1, sound: 'soundB' },
        { id: 2, sound: 'soundC' },
        { id: 3, sound: 'soundD' },
      ],
      difficulty: [
        {
          id:         'easy',
          text:       'Легкий',
          interval:   1500,
          checked:    true,
        },
        {
          id:         'normal',
          text:       'Средний',
          interval:    1000,
          checked:     false,
        },
        {
          id:         'hard',
          text:       'Сложный',
          interval:    400,
          checked:     false,
        }
      ],
    }
  },
  methods: {

    startGame() {
      this.level++
      this.messageToPlayer = `Уровень ${this.level}`;
      this.gameStarted = true;
      this.sequences.game.push(this.randomButton(4));
      this.currentStep = 0;
      this.sequences.player = [];
      this.showSequence();
    },

    endGame() {
      this.messageToPlayer = "Поражение"
      this.gameStarted = false;
      this.currentStep = 0;
      this.sequences.game = [];
      this.sequences.player = [];
      this.level = 0;
    },

    changeDifficulty(interval) {
      this.difficultyMod = interval;
    },

    showSequence() {
      this.sequenceShown = true;
      this.actionNow = 'Показываем последовательность';
      let i = 0;
      let delayLoop = () => {
        let timer = setTimeout(() => {
          this.currentLighten = null;
          let stepTimer = setTimeout(() => {
            if (i < this.sequences.game.length) {
              this.playSound(this.buttons[this.sequences.game[i]]);
              this.currentLighten = this.sequences.game[i];
              delayLoop();
            } else {
              this.actionNow = 'Ваша очередь';
              this.sequenceShown = false;
            }
            i++;
            clearTimeout(stepTimer);
          }, this.difficultyMod);
          clearTimeout(timer);
        }, 500);
      }
      delayLoop()
    },
    
    randomButton(max) {
      return Math.floor(Math.random() * Math.floor(max));
    },

    playSound(button) {
      let note = new Audio(require(`@/assets/${button.sound}.ogg`));
      note.addEventListener("canplaythrough", () => { 
        note.play();
      });
    },

    lightButton(button) {
      this.clickedId = button.id;
      this.playSound(button);
      let timer = setTimeout(() => {
        this.clickedId = null;
        clearTimeout(timer);
      }, 300)
    },
      
    checkSequence() {
      if (this.sequences.player[this.currentStep] === this.sequences.game[this.currentStep]) {
        this.currentStep++;
        this.sequences.player.length === this.sequences.game.length ? this.startGame(): null;
      } else {
        this.gameStarted ? this.endGame(): null;
      }
    },

    repeatSequence(button) {
      if (!this.sequenceShown) {
        this.lightButton(button);
        this.sequences.player.push(button.id);
        this.checkSequence();
      }
    }
  }
}
</script>

<style src="./App.css" />
