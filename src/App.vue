<template>
  <div id="app">
    <div style="height: 400px; background-color: #f7f7f7;">
      <VueLineChart/>
    </div>
  </div>
</template>

<script>
import VueLineChart from './components/VueLineChart.vue'

export default {
  name: 'app',
  components: {
    VueLineChart
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
