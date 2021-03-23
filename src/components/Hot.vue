<template>
  <div class="com-container">
    <div class="com-chart" ref="hot_ref"></div>
    <span class="iconfont arr-left" @click="toLeft">&#xe6ef;</span>
    <span class="iconfont arr-right" @click="toRight">&#xe6ed;</span>
    <span class="cat-name">{{ catName }}</span>
  </div>
</template>

<script>
export default {
  data() {
    return {
      chartInstance: null,
      allData: null,
      currentIndex: 0 // 当前所展示出的一级分类数据
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
  computed: {
    catName() {
      if (!this.allData) {
        return ''
      } else {
        return this.allData[this.currentIndex].name
      }
    }
  },
  methods: {
    // 初始化echartsInstance对象
    initChart() {
      this.chartInstance = this.$echarts.init(this.$refs.hot_ref, 'chalk')
      // 对图表初始化配置的控制
      const initOption = {
        title: {
          text: '▎ 热销商品占比',
          left: 20,
          top: 20
        },
        legend: {
          top: '5%',
          icon: 'circle'
        },
        tooltip: {
          show: true,
          formatter: arg => {
            const thirdCategory = arg.data.children
            // 计算出所有三级分类的数值总和
            let total = 0
            thirdCategory.forEach(item => {
              total += item.value
            })
            let retStr = ''
            thirdCategory.forEach(item => {
              retStr += `${item.name}:${parseInt(item.value / total * 100) + '%'}<br/>`
            })
            return retStr
          }
        },
        series: [
          {
            type: 'pie',
            label: {
              show: false
            },
            // 高亮模式
            emphasis: {
              label: {
                show: true
              },
              labelLine: {
                show: false
              }
            }
          }
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
      const legendDate = this.allData[this.currentIndex].children.map(item => {
        return item.name
      })
      const seriesData = this.allData[this.currentIndex].children.map(item => {
        return {
          name: item.name,
          value: item.value,
          children: item.children
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
    },
    toLeft() {
      this.currentIndex--
      if (this.currentIndex < 0) {
        this.currentIndex = this.allData.length - 1
      }
      this.updateChart()
    },
    toRight() {
      this.currentIndex++
      if (this.currentIndex > this.allData.length - 1) {
        this.currentIndex = 0
      }
      this.updateChart()
    }
  }
}
</script>

<style lang="scss" scoped>
.arr-left {
  position: absolute;
  left: 10%;
  top: 50%;
  transform: translateY(-50%);
  cursor: pointer;
  color: white;
}

.arr-right {
  position: absolute;
  right: 10%;
  top: 50%;
  transform: translateY(-50%);
  cursor: pointer;
  color: white;
}

.cat-name {
  position: absolute;
  left: 80%;
  bottom: 20px;
  color: white;
}
</style>