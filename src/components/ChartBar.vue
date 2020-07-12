<template>
  <div class="container" ref="container">
    <svg class="svg" ref='svg'>
      <g class="origin-lines">
        <line ref='x-origin' />
        <line ref='y-origin' />
      </g>
      <g class="scatters"></g>
      <g class="lines"></g>
      <g class="loesslines"></g>
      <g class="llslines"></g>
      <g ref="trendlines"></g>
      <g class="x-axis"></g>
      <g class="y-axis"></g>
    </svg>
  </div>

</template>

<script>
  import * as d3 from 'd3';
  import * as science from 'science';
  import * as _ from 'lodash';

  window.d3 = d3;

  export default {
    name: 'ChartHistogram',
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
        type: Object,
        default() {
          return {
            options: {},
            values: [],
          };
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
      data: {
        deep: true,
        handler() {
          this.refreshChart();
        },
      },
      options: {
        deep: true,
        handler() {
          this.refreshChart();
        },
      },
    },
    mounted() {
      this.svg = d3.select(this.$refs.svg);
      this.refreshChart();
      window.addEventListener('resize', this.refreshChart);
    },
    methods: {
      refreshChart() {
        const margin = {
          bottom: 20,
          left: 60,
          right: 20,
          top: 20,
        };

        // Update chart bounds
        // Must get offsetWidth/height on container, not svg
        this.geometry.width = this.$refs.container.offsetWidth;
        this.geometry.height = this.$refs.container.offsetHeight;

        // Set the x-scale
        this.x
          .range([margin.left, this.geometry.width - margin.right])
          .domain(this.axisRanges.x);

        // Set the y-scale
        this.y
          .range([this.geometry.height - margin.bottom, margin.top])
          .domain(this.axisRanges.y);

        // Assign the scales to the axes
        d3.axisLeft().scale(this.x);
        d3.axisBottom().scale(this.y);

        this.svg.select("g.x-axis")
          .attr("transform", `translate(0,${this.geometry.height - margin.bottom})`)
          .call(d3.axisBottom(this.x));

        this.svg.select("g.y-axis")
          .attr("transform", `translate(${margin.left},0)`)
          .call(d3.axisLeft(this.y));

        if (this.axisRanges.x[0] < 0 && this.axisRanges.x[1] > 0) {
          d3.select(this.$refs['x-origin'])
            .attr("x1", this.x(0))
            .attr("x2", this.x(0))
            .attr("y1", this.y(this.axisRanges.y[0]))
            .attr("y2", this.y(this.axisRanges.y[1]))
            .style('display', '');
        } else {
          d3.select(this.$refs['x-origin'])
            .style('display', 'none');
        }
        if (this.axisRanges.y[0] < 0 && this.axisRanges.y[1] > 0) {
          d3.select(this.$refs['y-origin'])
            .attr("y1", this.y(0))
            .attr("y2", this.y(0))
            .attr("x1", this.x(this.axisRanges.x[0]))
            .attr("x2", this.x(this.axisRanges.x[1]))
            .style('display', '');
        } else {
          d3.select(this.$refs['y-origin'])
            .style('display', 'none');
        }




      },
    },
    computed: {
      axisRanges() {
        let xLower;
        let xUpper;
        let yLower;
        let yUpper;

        const xs = this.data.values.map(p => p[0]);
        if (this.opts.x.min === 'auto') {
          // Then find the lowest absolute x value
          const minX = Math.min(...xs);
          // And the lowest positive x value
          const minXPositive = Math.min(...xs.filter(x => x > 0));
          xLower = this.opts.x.type === 'log' ? minXPositive : minX;
        } else {
          xLower = this.opts.x.min;
        }

        if (this.opts.x.max === 'auto') {
          // Find the highest x values
          xUpper = Math.max(...xs);
        } else {
          xUpper = this.opts.x.max;
        }

        const ys = this.data.values.map(p => p[1]);
        if (this.opts.y.min === 'auto') {
          // Then find the lowest absolute y value
          const minY = Math.min(...ys);
          // And the lowest positive y value
          const minYPositive = Math.min(...ys.filter(y => y > 0));
          yLower = this.opts.y.type === 'log' ? minYPositive : minY;
        } else {
          yLower = this.opts.y.min;
        }

        if (this.opts.y.max === 'auto') {
          // Find the highest y values
          yUpper = Math.max(...ys);
        } else {
          yUpper = this.opts.y.max;
        }



        // TODO: handle when xUpper is less than xLower
        return {
          x: [xLower, xUpper],
          y: [yLower, yUpper],
        };
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
          },
          loessBandwidth: 0.3,
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

  .origin-lines {
    stroke-width: 1px;
    stroke: #aaa;
  }

</style>
