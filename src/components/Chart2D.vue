<template>
  <div class="container" ref="container">
    <svg class="svg" ref='svg'>
      <g class="origin-lines">
        <line ref='x-origin' />
        <line ref='y-origin' />
      </g>
      <g class="scatters"></g>
      <g class="lines"></g>
      <g class="llslines"></g>
      <g ref="trendlines"></g>
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
      data: {
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


        // Draw the 'scatter' charts
        this.svg.select("g.scatters")
          // State that a list of g elements
          .selectAll('g')
          // will represent each series within this.data
          .data(this.data.filter(series => series.options.type === 'scatter'))
          // and that by default:
          //   existing elements will be updated,
          //   new elements will be added,
          //   and removed elements will be removed.
          .join('g')
          .style("fill", series => {
            if (series.options && series.options.color) {
              return series.options.color;
            }
            const i = this.data.indexOf(series);
            return this.colors[i % this.colors.length];
          })
          // State that a list of circle elements
          .selectAll("circle")
          // will represent each data point within this.data[n]
          .data(series => series.values.filter(d => {
            // For scatter plot, remove invalid points
            if (isNaN(d[0] + d[1])) return false;
            // Filter out negative values if log scales
            if (this.opts.x.type === 'log' && d[0] <= 0) return false;
            if (this.opts.y.type === 'log' && d[1] <= 0) return false;
            return true;
          }))
          // default (see above)
          .join("circle")
          .attr("cx", d => this.x(d[0]))
          .attr("cy", d => this.y(d[1]))
          .attr("r", 5);

        // Draw the 'line' charts
        this.svg.select("g.lines")
          // State that a list of path elements
          .selectAll('path')
          // will represent each series within this.data
          .data(this.data.filter(series => series.options.type === 'line'))
          // and that by default:
          //   existing elements will be updated,
          //   new elements will be added,
          //   and removed elements will be removed.
          .join('path')
          // State that a list of circle elements
          .attr("stroke", series => {
            if (series.options && series.options.color) {
              return series.options.color;
            }
            const i = this.data.indexOf(series);
            return this.colors[i % this.colors.length];
          })
          .datum(series => {
            if (series.options.sort) {
              return series.values.slice().sort((a,b) => a[0] - b[0]);
            }
            return series.values;
          })
          .attr("stroke-width", 1.5)
          .attr('fill', 'none')
          .attr('d', d3.line()
            .defined(d => {
              if (isNaN(d[0] + d[1])) return false;
              // Filter out negative values if log scales
              if (this.opts.x.type === 'log' && d[0] <= 0) return false;
              if (this.opts.y.type === 'log' && d[1] <= 0) return false;
              return true;
            })
            .x(point => this.x(point[0]))
            .y(point => this.y(point[1]))
          );

        const llsData = this.data.map(series => {
          const endpoints = [];

          if (series.values.length > 1) {
            const xs = series.values.map(p => p[0]);
            const minX = Math.min(...xs);
            const maxX = Math.max(...xs);

            let sumX = 0;
            let sumY = 0;
            let sumXX = 0;
            let sumXY = 0;

            for (const p of series.values) {
              const x = p[0];
              const y = p[1];
              sumX += x;
              sumY += y;
              sumXY += x * y;
              sumXX += x * x;
            }

            const n = series.values.length;
            const m = (n * sumXY - sumX * sumY) / (n * sumXX - Math.pow(sumX, 2));
            const b = (sumY - m * sumX) / n;
            const y = x => m * x + b;

            endpoints.push([minX, y(minX)]);
            endpoints.push([maxX, y(maxX)]);
          }

          return {
            options: {},
            values: endpoints,
          }
        });

        // Draw the lls lines
        this.svg.select("g.llslines")
          // State that a list of path elements
          .selectAll('path')
          // will represent each series within this.data
          .data(llsData)
          // and that by default:
          //   existing elements will be updated,
          //   new elements will be added,
          //   and removed elements will be removed.
          .join('path')
          // State that a list of circle elements
          .attr("stroke", series => {
            if (series.options && series.options.color) {
              return series.options.color;
            }
            const i = llsData.indexOf(series);
            return this.colors[i % this.colors.length];
          })
          .datum(series => {
            if (series.options.sort) {
              return series.values.slice().sort((a,b) => a[0] - b[0]);
            }
            return series.values;
          })
          .attr("stroke-width", 0.5)
          .attr('fill', 'none')
          .attr('d', d3.line()
            .defined(d => {
              if (isNaN(d[0] + d[1])) return false;
              // Filter out negative values if log scales
              if (this.opts.x.type === 'log' && d[0] <= 0) return false;
              if (this.opts.y.type === 'log' && d[1] <= 0) return false;
              return true;
            })
            .x(point => this.x(point[0]))
            .y(point => this.y(point[1]))
          );


        // const trendlinesData = this.data.map(series => {
        //   const leftMost = [...series.values].sort((a,b) => a[0] - b[0])[0];
        //   const rightMost = [...series.values].sort((a,b) => b[0] - a[0])[0];
        //   return [
        //     leftMost,
        //     rightMost,
        //   ];
        // });
        //
        // d3.select(this.$refs.trendlines)
        //   .selectAll('path').data(trendlinesData)
        //   .join('path')
        //   .attr("stroke", "steelblue")
        //   .attr("stroke-width", 1.5)
        //   .attr('d', d3.line()
        //     .x(point => this.x(point[0]))
        //     .y(point => this.y(point[1]))
        //   );

        // const path = d3.line().curve(d3.curveNatural)
        //   .x((d, i) => this.x(i))
        //   .y(d => this.y(d));
        // this.lines = this.processed.vals.map(i => path(i));
      },
    },
    computed: {
      axisRanges() {
        let xLower;
        let xUpper;
        let yLower;
        let yUpper;

        const xs = [].concat(...this.data.map(s => s.values.map(p=>p[0])));
        if (this.opts.x.min === 'auto') {
          // Then find the lowest absolute x value
          const minX = Math.min(...xs);
          // And the lowest positive x value
          const minXPositive = Math.min(...xs.filter(p => p > 0));
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

        const ys = [].concat(...this.data.map(s => s.values.map(p=>p[1])));
        if (this.opts.y.min === 'auto') {
          // Then find the lowest absolute y value
          const minY = Math.min(...ys);
          // And the lowest positive y value
          const minYPositive = Math.min(...ys.filter(p => p > 0));
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

  .origin-lines {
    stroke-width: 1px;
    stroke: #aaa;
  }

</style>
