<template>
  <div id="app">
    <div style="height: 400px; background-color: #f7f7f7;">
      <VueLineChart :vals="values"/>
    </div>
    <button @click="addValue">Add</button>
  </div>
</template>

<script>
import VueLineChart from './components/VueLineChart.vue'

export default {
  name: 'app',
  components: {
    VueLineChart
  },
  data() {
    return {
      values: [
        [1, 3, 6, 8, 13],
        [2, 5, 7, 10, 25],
        [3, 4, 8, 10],
      ]
    };
  },
  methods: {
    addValue() {
      for (let j = 0; j < 30; j++) {
        for (let i = 0; i < this.values.length; i++) {
          this.values[i].push(this.values[i][this.values[i].length - 1]*(0.2*Math.random()+1));
        }
      }

    },
  }
}
</script>

<style lang="scss">
  @import 'css/reset';
#app {
  padding: 30px;
  font-family: "Arial";
  color: #111;
  font-size: 15px;
  line-height: 20px;
}
</style>
