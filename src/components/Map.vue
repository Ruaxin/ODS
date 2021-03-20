<template>
  <div class="com-container">
    <div class="com-chart" ref="map_ref"></div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      chartInstance: null,
      allData: null
    }
  },
  mounted() {
    this.initChart()
    this.getData()
    window.addEventListener('resize', this.screenAdapter)
    this.screenAdapter()
  },
  destroyed() {
    window.removeEventListener('resize', this.screenAdapter)
  },
  methods: {
    async initChart() {
      this.chartInstance = this.$echarts.init(this.$refs.map_ref)
      const {data: ret} = await axios.get('http://localhost:8999/static/map/china.json')
      this.$echarts.registerMap('china', ret)
      const initOption = {
        geo: {
          type: 'map',
          map: 'china'
        }
      }
      this.chartInstance.setOption(initOption)
    },
    async getData() {
      this.updateChart()
    },
    updateChart() {
      const dataOption = {}
      this.chartInstance.setOption(dataOption)
    },
    screenAdapter() {
      const adapterOption = {}
      this.chartInstance.setOption(adapterOption)
      this.chartInstance.resize()
    }
  }
}
</script>

<style scoped>

</style>
