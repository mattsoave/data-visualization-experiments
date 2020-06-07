<template>
  <div id="app">
    <div style="height: 600px; background-color: #f3f3f3;">
      <Chart2D :vals="values" :data="scatterData2" :options="{
            x:{type:'linear'},
            y:{type:'linear'}
          }"/>
    </div>
    <button @click="addValue">Add</button>
<!--    <div style="height: 600px; background-color: #ddd;">-->
<!--      <VueScatter :vals="values" :data="scatterData"/>-->
<!--    </div>-->
<!--    <button @click="addValue">Add</button>-->
<!--    <div style="height: 400px; background-color: #f7f7f7;">-->
<!--      <VueLineChart :vals="values"/>-->
<!--    </div>-->
<!--    <div style="height: 400px; background-color: #f7f7f7;">-->
<!--      <VueLineChart :vals="values" :options="{y:{type:'log'}}"/>-->
<!--    </div>-->
<!--    <button @click="addValue">Add</button>-->
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
        {
          values: [],
          options: {
            type: 'line',
            sort: true,
          },
        },
        {
          values: [],
          options: {
            type: 'scatter',
          },
        },
        // {
        //   values: [[1.1, 2], [0.6, 3.2], [10.6, 4.5], [3, 11]],
        //   options: {},
        // },
      ],
      scatterData2: [[]]
    };
  },
  created() {
    this.scatterData2 = this.scatterData.map(a=>a);
  },
  methods: {
    addValue() {
      let i = -50;
      setInterval(() => {
        this.scatterData2[0].values.push([i, Math.sin(i/100)*20 + 5]);
        this.scatterData2[1].values.push([i, Math.sin(i/100 + 1)*20 + Math.random()*10]);
        // const rand = Math.random()*20;
        // this.scatterData2[0].push([Math.sin(rand)*50, Math.cos(rand)*50]);
        // this.scatterData2[0].push([
        //   (Math.random() + Math.random() + Math.random() + Math.random())*50,
        //   (Math.random() + Math.random() + Math.random() + Math.random())*50,
        // ]);
        i += 1;
      }, 50);
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
