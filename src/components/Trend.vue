<template>
  <div class="com-container">
    <div class="com-chart" ref="trend_ref"></div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      chartInstance: null,
      allData: null,
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
    initChart() {
      this.chartInstance = this.$echarts.init(this.$refs.trend_ref)
      const initOption = {
        xAxis: {
          type: 'category'
        },
        yAxis: {
          type: 'value'
        }
      }
      this.chartInstance.setOption(initOption)
    },
    async getData() {
      const {data: ret} = await this.$http.get('trend')
      this.allData = ret
      this.updateChart()
    },
    updateChart() {
      // 类目轴的数据
      const timeArr = this.allData.common.month
      // y轴的数据 series下的数据
      const valueArr = this.allData.map.data
      const seriesArr = valueArr.map(item => {
        return {
          type: 'line',
          data: item.data
        }
      })
      const dataOption = {
        xAxis: {
          data: timeArr
        },
        series: seriesArr
      }
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
