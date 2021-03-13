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
      currentPage: 1,
      totalPage: 0,
      timerId: null
    }
  },
  mounted() {
    this.initChart()
    this.getData()
    window.addEventListener('resize', this.screenAdapter)
    this.screenAdapter()
  },
  destroyed() {
    clearInterval(this.timerId)
    window.removeEventListener('resize', this.screenAdapter)
  },
  methods: {
    initChart() {
      this.chartInstance = this.$echarts.init(this.$refs.seller_ref, 'chalk')
      const initOption = {
        title: {
          text: '▎ 商家销售统计',
          top: 20,
          left: 20
        },
        grid: {
          top: '20%',
          left: '3%',
          right: '6%',
          bottom: '3%',
          containLabel: true
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'line',
            z: 0,
            lineStyle: {
              color: '#2D3443'
            }
          }
        },
        xAxis: {type: 'value'},
        yAxis: {type: 'category'},
        series: [
          {
            type: 'bar',
            label: {
              show: true,
              position: 'right',
              color: 'white'
            },
            itemStyle: {
              color: {
                type: 'line',
                x: 0,
                y: 0,
                x2: 1,
                y2: 0,
                colorStops: [
                  {offset: 0, color: '#5052EE'},
                  {offset: 1, color: '#AB6EE5'},
                ]
              }
            }
          }
        ]
      }
      this.chartInstance.setOption(initOption)
      this.chartInstance.on('mouseover', () => {clearInterval(this.timerId)})
      this.chartInstance.on('mouseout', () => {this.startInterval()})
    },
    async getData() {
      const {data: ret} = await this.$http.get('seller.json')
      this.allData = ret
      this.allData.sort((a, b) => {return a.value - b.value})
      this.totalPage = this.allData.length % 5 === 0 ? this.allData.length / 5 : this.allData / 5 + 1
      this.updateChart()
      this.startInterval()
    },
    updateChart() {
      const start = (this.currentPage - 1) * 5
      const end = this.currentPage * 5
      const showData = this.allData.slice(start, end)
      const sellerNames = showData.map((item) => {return item.name})
      const sellerValues = showData.map((item) => {return item.value})
      const dataOption = {
        yAxis: {data: sellerNames},
        series: [
          {data: sellerValues}
        ]
      }
      this.chartInstance.setOption(dataOption)
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
    },
    screenAdapter() {
      const titleFontSize = this.$refs.seller_ref.offsetWidth / 100 * 4.6
      const adapterOption = {
        title: {
          textStyle: {
            fontSize: titleFontSize
          },
        },
        tooltip: {
          axisPointer: {
            lineStyle: {
              width: titleFontSize,
            }
          }
        },
        series: [
          {
            barWidth: titleFontSize,
            itemStyle: {
              barBorderRadius: [0, titleFontSize / 2, titleFontSize / 2, 0],
            }
          }
        ]
      }
      this.chartInstance.setOption(adapterOption)
      this.chartInstance.resize()
    }
  }
}
</script>
<style></style>
