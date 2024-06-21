<script>
import axios from "axios";

export default {
  name: "App",
  data() {
    return {
      gameState: {},
      isWordError: false,
      wordErrorMessage: "Word Error.",
      isWordSuccess: false,
      wordSuccessMessage: "Guessed correctly.",
      remainingGuess: 0,
      guessWords: [],
      guessWord: "",
      guessResult: {},
      scrambleLetter: "",
    };
  },
  methods: {
    async getAnswer() {
      const { data } = await axios.get("http://localhost:8080/api/game/new");
      this.gameState = data;
      this.remainingGuess = data.remaining_words;
      this.scrambleLetter = data.scramble_word.split('').join(' ');
    },
    async handleClick(stateId, guessWord) {
        
      this.isWordSuccess = false;

      if(!guessWord || guessWord==='') {
          this.isWordError = true;
          this.wordErrorMessage = "Word Error."
          return;
      } else if(this.guessWords.includes(guessWord)) {
          this.isWordError = true;
          this.wordErrorMessage = "Word already guessed."
          return;
      } else {
          this.isWordError = false;
      }


      const { data } = await axios.post("http://localhost:8080/api/game/guess",
          {
            id: stateId,
            word: guessWord,
          });
      this.guessResult = data;
      this.remainingGuess = data.remaining_words;
      this.guessWords = data.guessed_words;

      if(data.result.indexOf('incorrect') > -1) {
          this.isWordError = true;
          this.wordErrorMessage = data.result;
      } else {
          this.isWordSuccess = true;
          this.wordSuccessMessage = data.result;
      };
    },
    newGame() {
        location.reload();
    },
  },
  beforeMount() {
    this.getAnswer();
  },
};
</script>

<template>
  <div class="bg-light rounded p-4">
        <div class="d-flex justify-content-between mb-4">
            <h2 class="text-primary-emphasis">Guess Words</h2>
        </div>
        <div v-if="!gameState" class="text-left">
            <p>Click <a class="refresh-game" @click.prevent="newGame">here</a> to start game.</p>
        </div>
        <div v-else class="col-sm-12 col-xl-12">
            <div v-if="!guessWords.length" class="bg-light rounded h-100 p-4">
                <p>No word guessed yet.</p>
            </div>
            <div v-else class="bg-light rounded h-100 p-4">
                <p>Words guessed: {{ guessWords.length }}</p>
                <ol>
                    <li v-for="item in guessWords"><span>{{ item }}</span></li>
                </ol>
            </div>
            <div v-if="remainingGuess == 0" class="bg-light rounded h-100 p-4">
                <p>Original: {{ gameState.original_word }}</p>
                <p>Scrambled letters: {{ scrambleLetter }}</p>
                <p v-if="isWordSuccess" class="text-success">{{ wordSuccessMessage }}</p>
                <p>Click <a class="refresh-game" @click.prevent="newGame">here</a> to start game.</p>
            </div>
            <div v-else class="bg-light rounded h-100 p-4">
                <p>Total possible:  {{ gameState.total_words }}</p>
                <p>Remaining words: {{ remainingGuess }}</p>
                <p>Original: {{ gameState.original_word }}</p>
                <p>Scrambled letters: {{ scrambleLetter }}</p>
                <form id="frmGameBoard" method="post">
                    <div class="row mb-3">
                        <label for="iptWord" class="col-sm-2 col-form-label">Word</label>
                        <div class="col-sm-6">
                            <input type="text" v-model="guessWord" class="form-control" id="iptWord" aria-describedby="iptWordFeedback" />
                        </div>
                        <label v-if="isWordError" id="iptWordFeedback" class="col-sm-4 text-danger is-invalid">{{ wordErrorMessage }}</label>
                        <label v-if="isWordSuccess" id="iptWordFeedback" class="col-sm-4 text-success is-valid">{{ wordSuccessMessage }}</label>
                    </div>
                    <div class="row mb-3">
                        <span class="col-sm-2"></span>
                        <div class="col-sm-6">
                            <button type="submit" class="btn btn-primary" @click.prevent="handleClick(gameState.id, guessWord)">Guess</button>
                        </div>
                    </div>
                </form>
            </div>
        </div>
    </div>
</template>

<style scoped>
</style>
