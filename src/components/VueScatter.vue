<template>
    <div class="container" ref="container">
        <svg class="svg" ref='svg'>
            <g class="plot"></g>
            <g class="x-axis"></g>
            <g class="y-axis"></g>
        </svg>
    </div>

</template>

<script>
  import * as d3 from 'd3';
  import * as _ from 'lodash';

  window.d2 = d3;

  export default {
    name: 'VueLineChart',
    data() {
      return {
        svg: null,
        type: 'line',
        x: d3.scaleLinear(),
        // y: d3.scaleLog(),
        lines: [],

        colors: d3.schemeTableau10,

        geometry: {
          width: null,
          height: null
        },

        unique: null,
      };
    },
    props: {
      vals: {
        type: Array,
        default() {
          return [];
        },
      },
      options: {
        type: Object,
        default() {
          return {};
        }
      }
    },
    created() {
      this.unique = Math.random();
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
          .domain([
            // Temp: for log scales, ensure we don't start at 0
            // Math.max(this.processed.extremes.yMin, this.opts.y.type === 'log' ? 1 : null),
            0,
            this.processed.extremes.yMax
          ]);

        // Assign the scales to the axes
        d3.axisLeft().scale(this.x);
        d3.axisBottom().scale(this.y);

        this.svg.select("g.x-axis")
          .attr("transform", "translate(0," + (this.geometry.height - 20) + ")")
          .call(d3.axisBottom(this.x));

        this.svg.select("g.y-axis")
          .attr("transform", "translate(60,0)")
          .call(d3.axisLeft(this.y));

        this.svg.select("g.plot")
          .selectAll('g').
          data(
            [
              [[1, 1], [2, 2], [3, 4]],
              [[0.1, 2], [0.2, 2.2], [0.3, 2.5]],
              [[1.1, 2], [0.6, 3.2], [0.6, 4.5], [3, 6]],
            ]
            )
          .join('g')
          .style("fill", (d, i) => this.colors[i])
              .selectAll("circle")
              .data(d => d)
              .join("circle")
              .attr("cx", d => this.x(d[0]))
              .attr("cy", d => this.y(d[1]))
              .attr("r", 5.5);

        //
        const path = d3.line().curve(d3.curveNatural)
          .x((d, i) => this.x(i))
          .y(d => this.y(d));
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
          yMin: Infinity,
          yMax: 0
        };

        for (const valArr of processed.vals) {
          if (valArr.length - 1 > extremes.xMax) extremes.xMax = valArr.length - 1;

          const yMax = Math.max(...valArr);
          const yMin = Math.min(...valArr);
          if (yMax > extremes.yMax) extremes.yMax = yMax;
          if (yMin < extremes.yMin) extremes.yMin = yMin;
        }
        processed.extremes = extremes;

        return processed;
      },
      opts() {
        const defaultOptions = {
          y: {
            type: 'linear'
          }
        };
        return _.merge({}, defaultOptions, this.options);
      },
      y() {
        let y;
        if (this.opts.y && this.opts.y.type === "log") {
          y = d3.scaleLog();
        } else {
          y = d3.scaleLinear();
        }
        return y;
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
