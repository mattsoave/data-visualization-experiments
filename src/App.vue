<template>
  <div id="app">
    <div style="height: 600px; background-color: #f3f3f3;">
      <Chart2D :vals="values" :data="weightData" :options="{
            x:{
              type:'linear',
              min: 18000,
            },
            y:{
              type:'linear',
              max: 164,
            },
            loessBandwidth,
          }"/>
    </div>
    <div>
      <input type="range" name="loessBandwidth"
             min="0.01" max="1.00" v-model.number="loessBandwidth" step="0.005" style="width: 600px;">
      <label for="loessBandwidth">Bandwidth: {{loessBandwidth}}</label>
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
import weight from './data/weight.js';

export default {
  name: 'app',
  components: {
    VueLineChart,
    VueScatter,
    Chart2D
  },
  data() {
    return {
      loessBandwidth: 0.4,
      values: [
        [2, 3, 6, 8, 13],
        [2, 5, 7, 10, 25],
        [3, 4, 8, 10],
      ],
      scatterData: [
        {
            // values: [
            //   [1.1, 2], [0.6, 3.2], [10.6, 4.5], [3, 11],
            //   [2.1, 2], [1.6, 3.2], [11.6, 4.5], [4, 11],
            // ],
          values: [
            // [1, 1],
            // [2, 2],
            // [3, 3],
            // [4, 5],
            // [5, 6],
            // [6, 7],
          ],
          // values: [[1,2], [3, 4]],
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
    // addValue() {
    //   let i = -50;
    //   setInterval(() => {
    //     this.scatterData2[0].values.push([i, Math.sin(i/100)*20 + 5]);
    //     this.scatterData2[1].values.push([i, Math.sin(i/100 + 1)*20 + Math.random()*10]);
    //     // const rand = Math.random()*20;
    //     // this.scatterData2[0].push([Math.sin(rand)*50, Math.cos(rand)*50]);
    //     // this.scatterData2[0].push([
    //     //   (Math.random() + Math.random() + Math.random() + Math.random())*50,
    //     //   (Math.random() + Math.random() + Math.random() + Math.random())*50,
    //     // ]);
    //     i += 1;
    //   }, 50);
    // },
    addValue() {
      // setInterval(() => {
      //   const x = Math.random()*200;
      //   this.scatterData2[0].values.push([x, 20*Math.sin(x/10) + Math.random()*100]);
      // }, 80);
      for (let j = 0; j < 1000; j++) {
        const x = Math.random()*200;
        let y;
        y = 20*Math.sin(x/10) + 20 + Math.random()*100;
        // y = Math.pow(x + Math.random(), 2);
        this.scatterData2[0].values.push([x, y]);
      }
    },
  },
  computed: {
    weightData() {
      return [
        {
          values: weight
            .split('\n')
            .filter(row => row.includes('Bodyweight'))
            .map(row => {
              const cells = row.split('\t');

              return [
                Date.parse(`${cells[0]} ${cells[1]}`)/86400000,
                parseFloat(cells[3])
              ];
            }),
          options: {
            type: 'scatter'
          }
        }
      ];
    }
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
