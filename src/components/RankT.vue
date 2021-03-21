<template>
  <div class="com-container">
    <div class="com-chart" ref="rank_ref"></div>
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
      this.chartInstance = this.$echarts.init(this.$refs.rank_ref, 'chalk')
      // 对图表初始化配置的控制
      const initOption = {
        title: {
          text: '▎ 地区销售排行',
          left: 20,
          top: 20
        },
        grid: {
          top: '40%',
          left: '5%',
          right: '5%',
          bottom: '5%',
          containLabel: true
        },
        tooltip: {
          show: true
        },
        xAxis: {
          type: 'category'
        },
        yAxis: {
          type: 'value'
        },
        series: [{
          type: 'bar'
        }]
      }
      this.chartInstance.setOption(initOption)
    },
    // 获取服务器的数据
    async getData() {
      const {data: ret} = await this.$http.get('rank.json')
      this.allData = ret
      this.allData.sort((a, b) => {
        return b.value - a.value
      })
      this.updateChart()
    },
    // 更新图表
    updateChart() {
      const colorArr = [
        ['#0BA82C', '#4FF778'],
        ['#2E72BF', '#23E5E5'],
        ['#5052EE', '#AB6EE5'],
      ]
      const provinceArr = this.allData.map(item => {
        return item.name
      })
      const valueArr = this.allData.map(item => {
        return item.value
      })
      const dataOption = {
        xAxis: {
          data: provinceArr
        },
        series: [{
          data: valueArr,
          itemStyle: {
            color: arg => {
              let targetColorArr
              if (arg.value > 300) {
                targetColorArr = colorArr[0]
                return {
                  type: 'linear',
                  x: 0,
                  y: 0,
                  x2: 0,
                  y2: 1,
                  colorStops: [{
                    offset: 0, color: targetColorArr[0] // 0% 处的颜色
                  }, {
                    offset: 1, color: targetColorArr[1] // 100% 处的颜色
                  }],
                }
              } else if (arg.value > 200) {
                targetColorArr = colorArr[1]
                return {
                  type: 'linear',
                  x: 0,
                  y: 0,
                  x2: 0,
                  y2: 1,
                  colorStops: [{
                    offset: 0, color: targetColorArr[0] // 0% 处的颜色
                  }, {
                    offset: 1, color: targetColorArr[1] // 100% 处的颜色
                  }],
                }
              } else {
                targetColorArr = colorArr[2]
                return {
                  type: 'linear',
                  x: 0,
                  y: 0,
                  x2: 0,
                  y2: 1,
                  colorStops: [{
                    offset: 0, color: targetColorArr[0] // 0% 处的颜色
                  }, {
                    offset: 1, color: targetColorArr[1] // 100% 处的颜色
                  }],
                }
              }
            }
          }
        }]
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