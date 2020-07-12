<template>
  <div class="container" ref="container">
    <svg class="svg" ref='svg'>
      <g class="bars"></g>
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
          width: 0,
          height: 0
        },

        unique: null,
        config: {
          margin: {
            bottom: 20,
            left: 60,
            right: 20,
            top: 20,
          },
        }
      };
    },
    props: {
      data: {
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
        console.log('refresh');

        // Update chart bounds
        // Must get offsetWidth/height on container, not svg
        this.geometry.width = this.$refs.container.offsetWidth;
        this.geometry.height = this.$refs.container.offsetHeight;

        // // Set the x-scale
        // this.x
        //   .range([margin.left, this.geometry.width - margin.right])
        //   .domain(this.axisRanges.x);
        //
        // // Set the y-scale
        // this.y
        //   .range([this.geometry.height - margin.bottom, margin.top])
        //   .domain(this.axisRanges.y);

        // Assign the scales to the axes
        d3.axisLeft().scale(this.x);
        d3.axisBottom().scale(this.y);

        this.svg.select("g.x-axis")
          .attr("transform", `translate(0,${this.geometry.height - this.config.margin.bottom})`)
          .call(d3.axisBottom(this.x));

        this.svg.select("g.y-axis")
          .attr("transform", `translate(${this.config.margin.left},0)`)
          .call(d3.axisLeft(this.y));

        this.svg.select("g.bars")
          .selectAll("rect")
          .data(this.bins)
          .join("rect")
          .attr("x", d => this.x(d.x0))
          .attr("y", d => this.y(d.length))
          // .attr("transform", d => {
          //   const translateX = this.x(d.x0);
          //   const translateY = this.y(d.length);
          //   return `translate(${translateX},${translateY})`;
          // })
          .attr("width", d => {
            const pxRight = this.x(d.x1);
            const pxLeft = this.x(d.x0);
            const gap = 1;
            const barWidth = pxRight - pxLeft;
            const displayWidth = barWidth - gap;
            return displayWidth;
          })
          .attr("height", d => this.maxBarHeight - this.y(d.length))
          .style("fill", this.colors[0])




      },
    },
    computed: {
      x() {

        const extent = d3.extent(this.data);
        const left = this.opts.x.min === 'auto' ? extent[0] : this.opts.x.min;
        const right = this.opts.x.max === 'auto' ? extent[1] : this.opts.x.max;

        const x = d3.scaleLinear()
        // .domain(d3.extent(this.data)).nice(this.opts.binCount)
        .domain([left, right])
        .range([this.config.margin.left, this.geometry.width - this.config.margin.right]);

        return x;
      },
      maxBarHeight() {
        return this.geometry.height - this.config.margin.bottom;
      },
      y() {
        const binLengths = this.bins.map(bin => bin.length);
        const maxY = Math.max(...binLengths);
        const y = d3.scaleLinear()
          .domain([0, maxY])
          .range([this.maxBarHeight, this.config.margin.top]);

        return y;
      },
      histogram() {
        const binCount = this.opts.binCount;
        // https://github.com/d3/d3-array/issues/46

        const extent = d3.extent(this.data);
        const left = this.opts.x.min === 'auto' ? extent[0] : this.opts.x.min;
        const right = this.opts.x.max === 'auto' ? extent[1] : this.opts.x.max;
        const diff = right - left;

        const thresholds = d3.range(left, right, diff / binCount);
        const histogram = d3.histogram()
          .value(d => d)
          .domain(this.x.domain())
          .thresholds(thresholds);

        return histogram;
      },
      bins() {
        const bins = this.histogram(this.data);
        return bins;
      },
      // axisRanges() {
      //   let xLower;
      //   let xUpper;
      //   let yLower;
      //   let yUpper;
      //
      //   const xs = this.data.values.map(p => p[0]);
      //   if (this.opts.x.min === 'auto') {
      //     // Then find the lowest absolute x value
      //     const minX = Math.min(...xs);
      //     // And the lowest positive x value
      //     const minXPositive = Math.min(...xs.filter(x => x > 0));
      //     xLower = this.opts.x.type === 'log' ? minXPositive : minX;
      //   } else {
      //     xLower = this.opts.x.min;
      //   }
      //
      //   if (this.opts.x.max === 'auto') {
      //     // Find the highest x values
      //     xUpper = Math.max(...xs);
      //   } else {
      //     xUpper = this.opts.x.max;
      //   }
      //
      //   const ys = this.data.values.map(p => p[1]);
      //   if (this.opts.y.min === 'auto') {
      //     // Then find the lowest absolute y value
      //     const minY = Math.min(...ys);
      //     // And the lowest positive y value
      //     const minYPositive = Math.min(...ys.filter(y => y > 0));
      //     yLower = this.opts.y.type === 'log' ? minYPositive : minY;
      //   } else {
      //     yLower = this.opts.y.min;
      //   }
      //
      //   if (this.opts.y.max === 'auto') {
      //     // Find the highest y values
      //     yUpper = Math.max(...ys);
      //   } else {
      //     yUpper = this.opts.y.max;
      //   }
      //
      //
      //
      //   // TODO: handle when xUpper is less than xLower
      //   return {
      //     x: [xLower, xUpper],
      //     y: [yLower, yUpper],
      //   };
      // },
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
          binCount: 25,
          loessBandwidth: 0.3,
        };
        return _.merge({}, defaultOptions, this.options);
      },
      // x() {
      //   let x;
      //   if (this.opts.y && this.opts.x.type === "log") {
      //     x = d3.scaleLog();
      //   } else {
      //     x = d3.scaleLinear();
      //   }
      //   return x;
      // },
      // y() {
      //   let y;
      //   if (this.opts.y && this.opts.y.type === "log") {
      //     y = d3.scaleLog();
      //   } else {
      //     y = d3.scaleLinear();
      //   }
      //   return y;
      // }
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
