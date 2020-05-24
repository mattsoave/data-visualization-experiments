<template>
    <div class="container" ref="container">
        <svg class="svg" ref='svg'>
            <g>
                <path v-for="(line, i) in lines" :d="line" :stroke='colors[i%10]' />
            </g>
            <g class="x-axis"></g>
            <g class="y-axis"></g>
        </svg>
    </div>

</template>

<script>
  import * as d3 from 'd3';

  window.d2 = d3;

  export default {
    name: 'VueLineChart',
    data() {
      return {
        data: [],

        svg: null,
        x: d3.scaleLinear(),
        y: d3.scaleLinear(),
        lines: [],

        colors: d3.schemeTableau10,

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
    props: {
      vals: {
        type: Array,
        default() {
          return [];
        },
      }
    },
    created() {
      for (let i = 0; i < 50; i++) {
        this.data.push(Math.pow(i, 2 + Math.random() * .1));
      }
    },
    watch: {
      vals() {
        this.refreshChart();
      },
    },
    mounted() {
      this.svg = d3.select(this.$refs.svg);
      this.refreshChart();
      window.addEventListener('resize', this.refreshChart);
    },
    methods: {
      refreshChart() {
        // Update chart bounds
        // Must get offsetWidth/height on container, not svg
        this.geometry.width = this.$refs.container.offsetWidth;
        this.geometry.height = this.$refs.container.offsetHeight;

        // Set the x-scale
        this.x
          .range([60, this.geometry.width - 10])
          .domain([this.processed.extremes.xMin, this.processed.extremes.xMax]);
        // Set the y-scale
        this.y
          .range([this.geometry.height - 20, 10])
          .domain([this.processed.extremes.yMin, this.processed.extremes.yMax]);

        // Assign the scales to the axes
        d3.axisLeft().scale(this.x);
        d3.axisBottom().scale(this.y);

        d3.select("g.x-axis")
          .attr("transform", "translate(0," + (this.geometry.height - 20) + ")")
          .call(d3.axisBottom(this.x));

        d3.select("g.y-axis")
          .attr("transform", "translate(60,0)")
          .call(d3.axisLeft(this.y));


        //
        const path = d3.line().curve(d3.curveNatural)
          .x((d, i) => this.x(i))
          .y(d => this.y(d))
        this.lines = this.processed.vals.map(i => path(i));
      },
    },
    computed: {
      processed() {
        const processed = {};

        if (Array.isArray(this.vals[0])) {
          processed.vals = this.vals;
        } else {
          processed.vals = [this.vals];
        }

        const extremes = {
          xMin: 0,
          xMax: 0,
          yMin: 0,
          yMax: null
        };

        for (const valArr of processed.vals) {
          if (valArr.length - 1 > extremes.xMax) extremes.xMax = valArr.length - 1;

          const yMax = Math.max(...valArr);
          if (yMax > extremes.yMax) extremes.yMax = yMax;
        }
        processed.extremes = extremes;

        return processed;
      }
    }
  };
</script>

<style lang="scss" scoped>
    .container, svg {
        width: 100%;
        height: 100%;
    }

    svg {
    }

    path {
        fill: none;
        stroke-width: 3px;
    }

</style>
