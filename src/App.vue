<template>
  <div id="app">
    <h1 @click="reset">{{ title }}</h1>
    <MinesweeperVue ref="minesweeper" :status.sync="status"></MinesweeperVue>
  </div>
</template>

<script>
import MinesweeperVue from './components/Minesweeper.vue'

export default {
  name: 'App',
  components: {
    MinesweeperVue
  },
  data() {
    return {
      status: 'ready'
    }
  },
  computed: {
    title() {
      return this.status === 'ready' ? '✨minesweeper✨' : this.status === 'win' ? '✨You Win✨' : '💥Game Over💥'
    }
  },
  methods: {
    reset() {
      this.$refs.minesweeper.reset();
    }
  }
}
</script>

<style>
#app {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
}

#app h1 {
  width: 100%;
  text-align: center;
  color: darkcyan;
  cursor: pointer;
}
</style>
