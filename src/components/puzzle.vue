<template>
  <div class="puzzle__container">
    <div
      class="puzzle__box"
      :style="{
        'grid-template-columns': '1fr '.repeat(size),
        'grid-template-rows': '1fr '.repeat(size),
      }"
    >
      <template v-for="(number, index) in state">
        <div
          :key="index"
          class="puzzle__tile"
          :class="{
            empty: number === size * size,
            correct: number === index + 1,
            highlighted: size > 6 && highlighted === number,
            paused,
            completed,
          }"
          :style="{ 'font-size': `${32 - (Math.floor(size * 1.8))}px`}"
          @click="move(number)"
          @mouseenter="highlighted = number + 1"
          @mouseleave="highlighted = null"
        >
          <transition name="tile">
            <span v-if="paused && !completed"></span>
            <span v-else-if="number === size * size && completed">{{ number }}</span>
            <span v-else-if="number === size * size && !completed"></span>
            <span v-else>{{ number }}</span>
          </transition>
        </div>
      </template>
    </div>
  </div>
</template>

<script>
export default {
  name: 'puzzle',
  props: {
    size: {
      type: Number,
      required: false,
      default: 3,
    },
    paused: {
      type: Boolean,
      required: true,
    },
  },
  data() {
    return {
      state: [],
      highlighted: null,
    };
  },
  computed: {
    completedState() {
      return Array.from(Array(this.size * this.size), (e, i) => i + 1);
    },
    completed() {
      return this.state
        .reduce((acc, cur, index) => acc && cur === this.completedState[index], true);
    },
  },
  methods: {
    swap(i, j) {
      const temp = this.state[i];
      this.$set(this.state, i, this.state[j]);
      this.$set(this.state, j, temp);
    },
    shuffle() {
      const loopSize = Math.floor(Math.random() * 1000 * this.size) + 1;
      const keys = ['up', 'dn', 'lt', 'rt'];
      let pKey = 'up';
      for (let i = 0; i < loopSize; i += 1) {
        let key = keys[Math.floor(Math.random() * keys.length)];
        if (key === 'up' && pKey === 'dn') key = 'lt';
        else if (key === 'dn' && pKey === 'up') key = 'rt';
        else if (key === 'rt' && pKey === 'lt') key = 'up';
        else if (key === 'lt' && pKey === 'rt') key = 'dn';
        pKey = key;
        this.arrow(key, true);
      }
    },
    inSameRow(a, b) {
      let pMax = 0;
      let result = false;
      Array.from(Array(this.size), (e, i) => this.size * (i + 1))
        .forEach((max) => {
          if (a <= max && b <= max && a > pMax && b > pMax) result = true;
          pMax = max;
        });
      return result;
    },
    move(number, exception = false) {
      if (this.completed || this.paused) {
        if (!exception) return;
      }
      const iNum = (this.state.indexOf(number) + 1);
      const i16 = (this.state.indexOf(this.size * this.size) + 1);
      if (Math.abs(iNum - i16) === this.size) this.swap(iNum - 1, i16 - 1);
      else if (Math.abs(iNum - i16) === 1 && this.inSameRow(iNum, i16)) {
        this.swap(iNum - 1, i16 - 1);
      }
      this.$emit('moves', !exception);
    },
    arrow(type, exception = false) {
      const i16 = (this.state.indexOf(this.size * this.size) + 1);
      let tile;
      if (type === 'up') tile = (this.state[i16 + this.size - 1]);
      if (type === 'dn') tile = (this.state[i16 - this.size - 1]);
      if (type === 'rt') tile = (this.state[i16 - 1 - 1]);
      if (type === 'lt') tile = (this.state[i16 + 1 - 1]);
      this.move(tile, exception);
    },
  },
  watch: {
    size: {
      immediate: true,
      handler(s) {
        this.state = Array.from(Array(s * s), (e, i) => i + 1);
        // this.$nextTick().then(() => this.shuffle());
      },
    },
    completed(c) {
      if (c) this.$emit('completed', c);
    },
  },
  created() {
    document.addEventListener('keydown', (e) => {
      if (e.key === 'ArrowUp') this.arrow('up');
      if (e.key === 'ArrowDown') this.arrow('dn');
      if (e.key === 'ArrowRight') this.arrow('rt');
      if (e.key === 'ArrowLeft') this.arrow('lt');
    });
  },
};
</script>

<style lang="scss">
  .puzzle__container {
    flex: 1;
    width: 30vw;
    height: 30vw;
    display: block;
    margin: 6px auto;
    border: 4px solid green;
    padding: 6px;
    border-radius: 6px;
    box-shadow: 3px 3px 10px 6px rgba(0, 0, 0, 0.5);
    background-color: #fff;
    .puzzle__box {
      // margin: 6px;
      height: 100%;
      display: grid;
      grid-gap: 4px;
      .puzzle__tile {
        background-color: green;
        border-radius: 6px;
        display: flex;
        justify-content: center;
        align-items: center;
        color: white;
        cursor: pointer;
        user-select: none;
        font-size: 24px;
        &:not(.empty) {
          box-shadow: 2px 2px 2px 1px rgba(0, 0, 0, 0.5);
        }
        &.correct {
          background-color: #097080;
        }
        &.highlighted {
          transform: scale(1.1);
        }
        &.empty:not(.completed) {
          background-color: transparent;
        }
        &.paused:not(.completed) {
          background-color: #2c3e50;
        }
      }
    }
  }
  @media only screen and (max-width: 640px) {
    .puzzle__container {
      width: 95vw;
      height: 95vw;
    }
  }

  .tile-enter-active, .tile-leave-active {
    transition: opacity 5s;
  }
  .tile-enter, .tile-leave-to /* .fade-leave-active below version 2.1.8 */ {
    opacity: 0;
  }
</style>
