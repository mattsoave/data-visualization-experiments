<template>
  <div class="container" ref="container">
    <svg class="svg" width="100%" height="100%">
      <g>
        <path :d="line" />
      </g>
      <g class="x-axis"></g>
      <g class="y-axis"></g>
    </svg>
  </div>

</template>

<script>
  import * as d3 from 'd3';
  export default {
    name: 'VueLineChart',
    data() {
      return {
        data: [],
        line: '',
        geometry: {
          width: null,
          height: null
        },
        d3Parameters: {
          xScale: d3.scaleLinear(),
          yScale: d3.scaleLinear(),
        }
      };
    },
    created() {
      for (let i = 0; i < 50; i++) {
        this.data.push(Math.pow(i, 2 + Math.random()*.1) );
      }
    },
    mounted() {
      this.refreshChart();
      window.addEventListener('resize', this.refreshChart);
    },
    methods: {
      refreshChart() {
        // Update chart bounds
        this.geometry.width = this.$refs.container.offsetWidth;
        this.geometry.height = this.$refs.container.offsetHeight;

        // Set the x-scale
        this.d3Parameters.xScale
                .range([60, this.geometry.width - 10])
                .domain(d3.extent(this.data, (d, i) => i));
        // Set the y-scale
        this.d3Parameters.yScale
                .range([this.geometry.height - 20, 10])
                .domain([0, d3.max(this.data, d => d)]);

        // Assign the scales to the axes
        d3.axisLeft().scale(this.d3Parameters.xScale);
        d3.axisBottom().scale(this.d3Parameters.yScale);

        d3.select("g.x-axis")
                .attr("transform", "translate(0," + (this.geometry.height - 20) + ")")
                .call(d3.axisBottom(this.d3Parameters.xScale));

        d3.select("g.y-axis")
                .attr("transform", "translate(60,0)")
                .call(d3.axisLeft(this.d3Parameters.yScale));


        //
        const path = d3.line().curve(d3.curveNatural)
                .x((d, i) => this.d3Parameters.xScale(i))
                .y(d => this.d3Parameters.yScale(d))
        this.line = path(this.data);
      },
      getScales() {
        const x = d3.scaleTime().range([0, this.geometry.width]);
        const y = d3.scaleLinear().range([this.geometry.height, 0]);
        d3.axisLeft().scale(x);
        d3.axisBottom().scale(y);
        x.domain(d3.extent(this.data, (d, i) => i));
        y.domain([0, d3.max(this.data, d => d)]);
        console.log(x(4));
        return { x, y };
      },
      calculatePath() {
        const scale = this.getScales();
        const path = d3.line()
                .x((d, i) => scale.x(i))
                .y(d => scale.y(d));
        this.line = path(this.data);
      },
    },
  };
</script>

<style lang="scss" scoped>
  div.container {
    background-color: #f3f3f3;
  }
  svg {}
  path {
    fill: none;
    stroke: #76BF8A;
    stroke-width: 3px;
  }

</style>
