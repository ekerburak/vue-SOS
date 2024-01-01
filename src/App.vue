<template>
  <Grid :rowCount="rowCount" :columnCount="columnCount" :windowWidth="windowWidth"
    :currentPlayerColor="colors[currentPlayer]" @add-score="(add) => scores[currentPlayer] += add"
    @end-turn="currentPlayer ^= 1;" @end-game="handleGameOver" />
  <div class="btn-container">
    <input type="number" placeholder="# of rows" min="3" @input="handleRowCount">
    <input type="number" placeholder="# of columns" min="3" @input="handleColumnCount">
  </div>
  <div class="scoreboard">
    <p class="score-first" :style="{ backgroundColor: colors[0] }">
      Player 1: {{ scores[0] }}
    </p>
    <p class="score-second" :style="{ backgroundColor: colors[1] }">
      Player 2: {{ scores[1] }}
    </p>
  </div>
  <p class="turn-info" v-show="!gameOver" :style="{ backgroundColor: colors[currentPlayer] }">
    Player {{ currentPlayer + 1 }}'s turn
  </p>
  <p class="result-info" v-show="gameOver" :style="{ backgroundColor: colors[winner] }">
    {{ resultMessage }}
  </p>
</template>

<script>
import Grid from "./components/Grid.vue";
export default {
  data() {
    return {
      rowCount: 3,
      columnCount: 3,
      scores: [0, 0],
      currentPlayer: 0,
      gameOver: false,
      resultMessage: "",
      windowWidth: 0,
      winner: 2,
      colors: ["#482c70", "#c4721a", "#a2a667"]
    }
  },
  components: { Grid },
  methods: {
    handleRowCount(event) {
      if (!event.target.value)
        return;
      event.target.value = Math.max(event.target.value, 3);
      event.target.value = Math.min(event.target.value, 10);
      this.rowCount = event.target.value;
      this.scores = [0, 0];
      this.currentPlayer = 0;
      this.gameOver = false;
    },
    handleColumnCount(event) {
      if (!event.target.value)
        return;
      event.target.value = Math.max(event.target.value, 3);
      event.target.value = Math.min(event.target.value, 10);
      this.columnCount = event.target.value;
      this.scores = [0, 0];
      this.currentPlayer = 0;
      this.gameOver = false;
    },
    handleGameOver() {
      this.gameOver = true;
      if (this.scores[0] > this.scores[1]) {
        this.winner = 0;
        this.resultMessage = "Player 1 wins!";
      }
      else if (this.scores[1] > this.scores[0]) {
        this.winner = 1;
        this.resultMessage = "Player 2 wins!";
      }
      else {
        this.winner = 2;
        this.resultMessage = "It is a draw!";
      }
    }
  },
  mounted() {
    //there is apparently no way to access window width in vue without direct dom manipulation
    window.addEventListener('resize', () => this.windowWidth = window.innerWidth);
    this.windowWidth = window.innerWidth;
  }
};
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Nunito:ital,wght@0,200;0,300;0,400;0,500;0,600;0,700;0,800;0,900;0,1000;1,200;1,300;1,400;1,500;1,600;1,700;1,800;1,900;1,1000&display=swap");

:root {
  background-color: whitesmoke;
}

.btn-container,
.scoreboard {
  display: flex;
  justify-content: center;
  margin: 15px auto;
  width: 90vw;
  max-width: 500px;
}

.btn-container>input {
  flex-basis: 100%;
  min-width: 0;
  font-family: Nunito, sans-serif;
}

.scoreboard>p {
  flex-basis: 100%;
  height: 100%;
  min-width: 0;
  padding: 3px 1px;
  margin: 0;
  text-align: center;
  color: whitesmoke;
  font-family: Nunito, sans-serif;
  font-weight: 700;
}

.turn-info,
.result-info {
  text-align: center;
  font-family: Nunito, sans-serif;
  padding: 3px 1px;
  color: whitesmoke;
  margin: 0 auto;
  width: 90vw;
  max-width: 500px;
  font-weight: 700;
  border-radius: 6px;
}

.score-first {
  border-radius: 6px 0 0 6px;
}

.score-second {
  border-radius: 0 6px 6px 0;
}
</style>