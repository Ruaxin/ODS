<template>
  <div class="com-container">
    <div class="com-chart" ref="stock_ref"></div>
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
      this.chartInstance = this.$echarts.init(this.$refs.stock_ref, 'chalk')
      // 对图表初始化配置的控制
      const initOption = {}
      this.chartInstance.setOption(initOption)
    },
    // 获取服务器的数据
    async getData() {
      const {data: ret} = await this.$http.get('stock')
      this.allData = ret
      this.updateChart()
    },
    // 更新图表
    updateChart() {
      const centerArr = [
        ['18%', '40%'],
        ['50%', '40%'],
        ['82%', '40%'],
        ['34%', '75%'],
        ['66%', '75%'],
      ]
      const showData = this.allData.slice(0, 5)
      const seriesArr = showData.map((item, index) => {
        return {
          type: 'pie',
          radius: [110, 100],
          center: centerArr[index],
          data: [
            {
              value: item.sales
            },
            {
              value: item.stock
            }
          ]
        }
      })
      const dataOption = {
        series: seriesArr
      }
      this.chartInstance.setOption(dataOption)
    },
    // 自适应屏幕大小
    screenAdapter() {
      // const titleFontSize = this.$refs.stock_ref.offsetWidth / 100 * 3.6
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