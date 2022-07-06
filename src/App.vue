<template>
  <BoxVisualizer :boxList="[
    [-1, -2, 0],
    [0, -3, 0],
    [0, -3, 1],
    [1, -2, 0]
  ]" />
</template>

<script>
import BoxVisualizer from './components/BoxVisualizer.vue'

export default {
  name: 'App',
  components: {
    BoxVisualizer
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  margin-top: 60px;
}
</style>
