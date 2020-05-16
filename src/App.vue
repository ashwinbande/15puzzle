<template>
  <div id="app" class="app">
    <div class="header">
      <h1>15 Puzzle</h1>
    </div>
    <div class="puzzle">
      <div class="puzzle__left">
        <div>
          Select Size:
          <select v-model="size" :disabled="!paused">
            <template v-for="value in [3, 4, 5, 6, 7, 8]">
              <option :key="value" :value="value">{{ value }}</option>
            </template>
          </select>
        </div>
        <button
          type="button"
          class="play__button"
          v-if="paused"
          @click="play"
        >
          {{ gameStarted ? 'Resume' : 'Play' }}
        </button>
        <button
          type="button"
          class="play__button"
          v-else
          @click="pause"
        >Pause</button>
        <button
          type="button"
          class="play__button"
          v-if="!paused"
          @click="restart"
        >
          Restart
        </button>
      </div>
      <div class="puzzle__main">
        <puzzle
          :paused="paused"
          :size="size"
          ref="puzzle"
          @moves="setMoves"
          @completed="gameCompleted"
        />
        <div class="puzzle__solved" v-if="gameComplete || paused">
          <span v-if="gameComplete">
            Congratulations! You Solved puzzle in
            {{ moves }} moves and {{ time }} seconds.
          </span>
          <span v-else>
            Paused
          </span>
        </div>
      </div>
      <div class="puzzle__right">
        <div class="puzzle__score">
          <div>
            <span>Time:</span>
            <span>{{ time }}s</span>
          </div>
          <div>
            <span>Moves:</span>
            <span>{{ moves }}</span>
          </div>
        </div>
      </div>
    </div>
    <div class="footer">
      <h4><a href="https://github.com/ashwinbande">© Ashwin Bande</a></h4>
    </div>
  </div>
</template>

<script>
import puzzle from '@/components/puzzle.vue';

export default {
  name: 'App',
  components: {
    puzzle,
  },
  data() {
    return {
      time: 0,
      moves: 0,
      size: 4,
      paused: true,
      gameStarted: false,
      interval: null,
      gameComplete: false,
    };
  },
  methods: {
    setMoves(valid) {
      if (valid) this.moves += 1;
    },
    gameCompleted() {
      clearInterval(this.interval);
      this.gameStarted = false;
      this.paused = true;
      this.gameComplete = true;
    },
    shuffle() {
      this.$refs.puzzle.shuffle();
    },
    propagateTime() {
      if (!this.paused) {
        this.time += 1;
      }
    },
    play() {
      this.paused = false;
      if (!this.gameStarted) this.restart();
    },
    pause() {
      this.paused = true;
    },
    restart() {
      this.shuffle();
      this.moves = 0;
      this.time = 0;
      this.gameStarted = true;
      this.gameComplete = false;
      clearInterval(this.interval);
      this.interval = setInterval(this.propagateTime, 1000);
    },
  },
  watch: {
    size() {
      this.restart();
      this.gameStarted = false;
    },
  },
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
  body {
    padding: 0;
    margin: 0;
    background-color: #f3f379;
  }
  .app {
    display: flex;
    flex-direction: column;
    .puzzle {
      display: flex;
      flex-direction: row;
      .puzzle__left, .puzzle__right {
        flex: 1;
        // background-color: deepskyblue;
      }
      .puzzle__left {
        font-size: 24px;
        font-weight: 700;
        select {
          font-size: 24px;
          font-weight: 700;
        }
        .play__button {
          border-radius: 6px;
          margin: 30px 20px;
          padding: 20px;
          width: 15vw;
          font-size: 24px;
          font-weight: 700;
          background-color: tomato;
          border: 0;
          box-shadow: 2px 3px 3px 2px rgba(0, 0, 0, 0.5);
        }
      }
      .puzzle__right {
        display: flex;
        .puzzle__score {
          background-color: #fff;
          border-radius: 6px;
          width: 15vw;
          margin: 10px auto auto auto;
          padding: 5px 0;
          box-shadow: 1px 1px 8px 4px rgba(0, 0, 0, 0.5);
          div {
            font-size: large;
            display: flex;
            padding: 5px 10px;
            font-weight: 600;
            span {
              flex: 1;
              text-align: left;
            }
            span:nth-child(even) {
              text-align: center;
            }
          }
        }
      }
      .puzzle__main {
        position: relative;
        .puzzle__solved {
          position: absolute;
          top: 0;
          left: 0;
          bottom: 0;
          right: 0;
          display: flex;
          flex-direction: column;
          justify-content: center;
          align-content: center;
          margin: 6px 0;
          padding: 20px;
          background-color: rgba(0, 0, 0, 0.8);
          border-radius: 6px;
          span {
            font-size: 28px;
            color: #fff;
          }
        }
      }
    }
  }

@media only screen and (max-width: 640px) {
  body {
    h1 {
      margin: 10px;
    }
    .app {
      // margin-top: 10px;
      .puzzle {
        flex-direction: column;

        .puzzle__left {
          font-size: 12px;

          select {
            font-size: 12px;
          }

          .play__button {
            border-radius: 6px;
            margin: 8px 4px;
            padding: 10px 5px;
            width: 30vw;
            font-size: 12px;
          }
        }

        .puzzle__right {
          .puzzle__score {
            width: 45vw;
            margin: 5px auto auto auto;

            div {
              padding: 2px 5px;
            }
          }
        }
      }
    }
  }
}
</style>
