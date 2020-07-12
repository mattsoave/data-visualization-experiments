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

        const binCount = this.opts.binCount;
        const histogram = d3.histogram()
          .value(d => d);



        // Set the x-scale
        const x = d3.scaleLinear()
          .domain(d3.extent(this.data))
          .range([this.config.margin.left, this.geometry.width - this.config.margin.right]);

        histogram.domain(x.domain())
          .thresholds(x.ticks(binCount));

        const bins = histogram(this.data);
        console.log(bins.length);

        // Set the y-scale
        const binLengths = bins.map(bin => bin.length);
        const maxY = Math.max(...binLengths);
        const y = d3.scaleLinear()
          .domain([0, maxY])
          .range([this.maxBarHeight, this.config.margin.top]);

        // Assign the scales to the axes
        d3.axisLeft().scale(this.x);
        d3.axisBottom().scale(this.y);

        this.svg.select("g.x-axis")
          .attr("transform", `translate(0,${this.geometry.height - this.config.margin.bottom})`)
          .call(d3.axisBottom(x));

        this.svg.select("g.y-axis")
          .attr("transform", `translate(${this.config.margin.left},0)`)
          .call(d3.axisLeft(y));



        this.svg.select("g.bars")
          .selectAll("rect")
          .data(bins)
          .join("rect")
          .attr("x", d => x(d.x0))
          .attr("y", d => y(d.length))
          // .attr("transform", d => {
          //   const translateX = this.x(d.x0);
          //   const translateY = this.y(d.length);
          //   return `translate(${translateX},${translateY})`;
          // })
          .attr("width", d => {
            const pxRight = x(d.x1);
            const pxLeft = x(d.x0);
            const gap = 1;
            const barWidth = pxRight - pxLeft;
            const displayWidth = barWidth - gap;
            return displayWidth;
          })
          .attr("height", d => this.maxBarHeight - y(d.length))
          .style("fill", this.colors[0])




      },
    },
    computed: {
      maxBarHeight() {
        return this.geometry.height - this.config.margin.bottom;
      },
      histogram() {
        // const binCount = this.opts.binCount;
        // const histogram = d3.histogram()
        //   .value(d => d)
        //   .domain(this.x.domain())
        //   .thresholds(this.x.ticks(binCount));
        //
        // return histogram;
      },
      bins() {
        // return this.histogram(this.data);
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
