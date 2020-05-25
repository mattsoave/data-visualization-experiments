<template>
  <div id="app">
      <div style="height: 600px; background-color: #ddd;">
          <Chart2D :vals="values" :data="scatterData2" :options="{x:{type:'linear'}, y:{type:'log'}}"/>
      </div>
      <button @click="addValue">Add</button>
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
import Chart2D from './components/Chart2D.vue'

export default {
  name: 'app',
  components: {
    VueLineChart,
    VueScatter,
    Chart2D
  },
  data() {
    return {
      values: [
        [2, 3, 6, 8, 13],
        [2, 5, 7, 10, 25],
        [3, 4, 8, 10],
      ],
      scatterData: [
        [[1, -1], [1.3, 0.3], [2, 2], [3, 4]],
        [[-0.1, 2], [0.2, 2.2], [0.3, 2.5]],
        [[1.1, 2], [0.6, 3.2], [10.6, 4.5], [3, 6]],
      ],
      scatterData2: [[]]
    };
  },
  created() {
    this.scatterData2 = this.scatterData.map(a=>a);
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
      for (let i = 1; i < 10; i += 0.1) {
        newScatter.push([i, Math.pow(2+ Math.random()/10, i ) + Math.random()*10]);
      }
      this.scatterData2.push(newScatter);

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
