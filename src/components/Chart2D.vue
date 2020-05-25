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
    name: 'Chart2D',
    data() {
      return {
        svg: null,
        type: 'line',
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
      data: {
        type: Array,
        default() {
          return [];
        },
      },
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
        const xAxisHeight = 20;
        const xAxisBottomOffset = 10;
        const yAxisWidth = 60;
        const yAxisLeftOffset = 10;
        // Update chart bounds
        // Must get offsetWidth/height on container, not svg
        this.geometry.width = this.$refs.container.offsetWidth;
        this.geometry.height = this.$refs.container.offsetHeight;

        // Set the x-scale
        this.x
          .range([60, this.geometry.width - 10])
          .domain([
            this.opts.x.type === 'log' ? Math.max(this.extremes.xMin, this.extremes.xMinPositive) : this.extremes.xMin,
            this.opts.x.type === 'log' ? Math.max(this.extremes.xMax, this.extremes.xMinPositive + 1) : this.extremes.xMax,
          ]);
        // Set the y-scale
        this.y
          .range([this.geometry.height - 20, 10])
          .domain([
            this.opts.y.type === 'log' ? Math.max(this.extremes.yMin, this.extremes.yMinPositive) : this.extremes.yMin,
            this.opts.y.type === 'log' ? Math.max(this.extremes.yMax, this.extremes.yMinPositive + 1) : this.extremes.yMax,
          ]);

        // Assign the scales to the axes
        d3.axisLeft().scale(this.x);
        d3.axisBottom().scale(this.y);

        this.svg.select("g.x-axis")
          .attr("transform", `translate(0,${this.geometry.height - xAxisHeight})`)
          .call(d3.axisBottom(this.x));

        this.svg.select("g.y-axis")
          .attr("transform", `translate(${yAxisWidth},0)`)
          .call(d3.axisLeft(this.y));

        this.svg.select("g.plot")
          .selectAll('g').
          data(this.data)
          .join('g')
          .style("fill", (d, i) => this.colors[i%this.colors.length])
              .selectAll("circle")
              .data(d => d.filter(d => {
                if (Number.isNaN(d[0])) return false;
                if (Number.isNaN(d[1])) return false;
                // Filter out negative values if log scales
                if (this.opts.x.type === 'log' && d[0] <= 0) return false;
                if (this.opts.y.type === 'log' && d[1] <= 0) return false;
                return true;
              }))
              .join("circle")
              .attr("cx", d => this.x(d[0]))
              .attr("cy", d => this.y(d[1]))
              .attr("r", 5.5);
      },
    },
    computed: {
      extremes() {
        const extremes = {
          xMin: Infinity,
          xMinPositive: Infinity,
          xMax: 0,
          yMin: Infinity,
          yMinPositive: Infinity,
          yMax: 0
        };

        for (const series of this.data) {
          const xs = series.map(p => p[0]);
          const xMax = Math.max(...xs);
          const xMin = Math.min(...xs);
          const xMinPositive = Math.min(...xs.filter(x => x > 0));
          if (xMax > extremes.xMax) extremes.xMax = xMax;
          if (xMin < extremes.xMin) extremes.xMin = xMin;
          if (xMinPositive < extremes.xMinPositive) extremes.xMinPositive = xMinPositive;


          const ys = series.map(p => p[1]);
          const yMax = Math.max(...ys);
          const yMin = Math.min(...ys);
          const yMinPositive = Math.min(...ys.filter(y => y > 0));
          if (yMax > extremes.yMax) extremes.yMax = yMax;
          if (yMin < extremes.yMin) extremes.yMin = yMin;
          if (yMinPositive < extremes.yMinPositive) extremes.yMinPositive = yMinPositive;
        }

        return extremes;
      },
      opts() {
        const defaultOptions = {
          x: {
            type: 'linear',
            min: 'auto',
            max: 'auto',
          },
          y: {
            type: 'linear',
            min: 'auto',
            max: 'auto',
          }
        };
        return _.merge({}, defaultOptions, this.options);
      },
      x() {
        let x;
        if (this.opts.y && this.opts.x.type === "log") {
          x = d3.scaleLog();
        } else {
          x = d3.scaleLinear();
        }
        return x;
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
