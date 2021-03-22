<template>
  <div class="com-container">
    <div class="com-chart" ref="hot_ref"></div>
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
    // 初始化echartsInstance对象
    initChart() {
      this.chartInstance = this.$echarts.init(this.$refs.hot_ref, 'chalk')
      // 对图表初始化配置的控制
      const initOption = {
        series: [
          {type: 'pie'}
        ]
      }
      this.chartInstance.setOption(initOption)
    },
    // 获取服务器的数据
    async getData() {
      const {data: ret} = await this.$http.get('hotproduct')
      this.allData = ret
      this.updateChart()
    },
    // 更新图表
    updateChart() {
      const legendDate = this.allData[0].children.map(item => {
        return item.name
      })
      const seriesData = this.allData[0].children.map(item => {
        return {
          name: item.name,
          value: item.value
        }
      })
      const dataOption = {
        legend: {
          data: legendDate
        },
        series: [
          {
            data: seriesData
          }
        ]
      }
      this.chartInstance.setOption(dataOption)
    },
    // 自适应屏幕大小
    screenAdapter() {
      // const titleFontSize = this.$refs.rank_ref.offsetWidth / 100 * 3.6
      const adapterOption = {}
      this.chartInstance.setOption(adapterOption)
      // 调用resize方法
      this.chartInstance.resize()
    }
  }
}
</script>

<style scoped>

</style>