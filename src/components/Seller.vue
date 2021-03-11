<!--商家销量统计的横向柱状图-->
<template>
  <div class="com-container">
    <div class="com-chart" ref="seller_ref"></div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      chartInstance: null,
      allData: null,
      currentPage: 1, // 当前显示的页数
      totalPage: 0, // 一共多少页
      timerId: null // 定时器标识
    }
  },
  mounted() {
    this.initChart()
    this.getData()
  },
  destroyed() {
    clearInterval(this.timerId)
  },
  methods: {
    // 初始化echartInstance对象
    initChart() {
      this.chartInstance = this.$echarts.init(this.$refs.seller_ref)
      // 对图表对象进行鼠标事件的监听
      this.chartInstance.on('mouseover', () => {
        clearInterval(this.timerId)
      })
      this.chartInstance.on('mouseout', () => {
        this.startInterval()
      })
    },
    // 获取服务器的数据
    async getData() {
      const {data: ret} = await this.$http.get('seller')
      this.allData = ret
      // 对数据进行排序
      this.allData.sort((a, b) => {
        return a.value - b.value
      })
      // 每五个显示一页
      this.totalPage = this.allData.length % 5 === 0 ? this.allData.length / 5 : this.allData.length / 5 + 1
      this.updateChart()
      // 启动定时器
      this.startInterval()
    },
    // 更新图表
    updateChart() {
      const start = (this.currentPage - 1) * 5 //0
      const end = this.currentPage * 5 //5
      const showData = this.allData.slice(start, end) //展示数据
      const sellerNames = showData.map((item) => {
        return item.name
      })
      const sellerValues = showData.map((item) => {
        return item.value
      })
      const option = {
        xAxis: {
          type: 'value'
        },
        yAxis: {
          type: 'category',
          data: sellerNames
        },
        series: [
          {
            type: 'bar',
            data: sellerValues
          }
        ]
      }
      this.chartInstance.setOption(option)
    },
    startInterval() {
      if (this.timerId) {
        clearInterval(this.timerId)
      }
      this.timerId = setInterval(() => {
        this.currentPage++
        if (this.currentPage > this.totalPage) {
          this.currentPage = 1
        }
        this.updateChart()
      }, 3000)
    }
  }
}
</script>

<style lang="scss" scoped>

</style>
