<template>
  <div id="app">
      <div style="height: 600px; background-color: #ddd;">
          <VueScatter :vals="values" :data="scatterData"/>
      </div>
      <button @click="addValue">Add</button>
    <div style="height: 400px; background-color: #f7f7f7;">
      <VueLineChart :vals="values"/>
    </div>
    <div style="height: 400px; background-color: #f7f7f7;">
      <VueLineChart :vals="values" :options="{y:{type:'log'}}"/>
    </div>
    <button @click="addValue">Add</button>
  </div>
</template>

<script>
import VueLineChart from './components/VueLineChart.vue'
import VueScatter from './components/VueScatter.vue'

export default {
  name: 'app',
  components: {
    VueLineChart,
    VueScatter
  },
  data() {
    return {
      values: [
        [2, 3, 6, 8, 13],
        [2, 5, 7, 10, 25],
        [3, 4, 8, 10],
      ],
      scatterData: [
        [[1, 1], [2, 2], [3, 4]],
        [[0.1, 2], [0.2, 2.2], [0.3, 2.5]],
        [[1.1, 2], [0.6, 3.2], [10.6, 4.5], [3, 6]],
      ]
    };
  },
  methods: {
    addValue() {
      for (let j = 0; j < 20; j++) {
        for (let i = 0; i < this.values.length; i++) {
          // this.values[i].push(this.values[i][this.values[i].length - 1]*((i/20+.1)*Math.random()+1));
          this.values[i].push(Math.sin(this.values[i].length/20)*20 + 21);
        }
      }
      const newScatter = [];
      for (let i = 0; i < 100; i++) {
        newScatter.push([i, Math.sin(i/20)*20 + Math.random()*10 - 5 + 25]);
      }
      this.scatterData.push(newScatter);

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
