<template>
  <div class="com-container" @dblclick="revertMap">
    <div class="com-chart" ref="map_ref"></div>
  </div>
</template>

<script>
import axios from 'axios'
import {getProvinceMapInfo} from '@/utils/map_utils.js'

export default {
  data() {
    return {
      chartInstance: null,
      allData: null,
      mapData: {}
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
      this.chartInstance = this.$echarts.init(this.$refs.map_ref, 'chalk')
      const {data: ret} = await axios.get('http://localhost:8999/static/map/china.json')
      this.$echarts.registerMap('china', ret)
      const initOption = {
        title: {
          text: '▎ 商家分布',
          left: 20,
          top: 20
        },
        geo: {
          type: 'map',
          map: 'china',
          top: '5%',
          bottom: '5%',
          itemStyle: {
            areaColor: '#1f4955',
            borderColor: '#333'
          }
        },
        legend: {
          left: '5%',
          bottom: '5%',
          orient: 'vertical'
        }
      }
      this.chartInstance.setOption(initOption)
      this.chartInstance.on('click', async arg => {
        const provinceInfo = getProvinceMapInfo(arg.name)
        if (!(provinceInfo.path === '/static/map/province/undefined.json')) {
          if (!this.mapData[provinceInfo.key]) {
            const {data: ret} = await axios.get('http://localhost:8999' + provinceInfo.path)
            this.mapData[provinceInfo.key] = ret
            this.$echarts.registerMap(provinceInfo.key, ret)
          }
          const changeOption = {
            geo: {
              map: provinceInfo.key
            }
          }
          this.chartInstance.setOption(changeOption)
        }
      })
    },
    async getData() {
      const {data: ret} = await this.$http.get('map.json')
      this.allData = ret
      this.updateChart()
    },
    updateChart() {
      const legendArr = this.allData.map(item => {
        return item.name
      })
      const seriesArr = this.allData.map(item => {
        return {
          type: 'effectScatter',
          rippleEffect: {
            scale: 5,
            brushType: 'stroke'
          },
          name: item.name,
          data: item.children,
          coordinateSystem: 'geo'
        }
      })
      const dataOption = {
        series: seriesArr,
        legend: {
          data: legendArr
        }
      }
      this.chartInstance.setOption(dataOption)
    },
    screenAdapter() {
      const titleFontSize = this.$refs.map_ref.offsetWidth / 100 * 3.6
      const adapterOption = {
        title: {
          textStyle: {
            fontSize: titleFontSize
          }
        },
        legend: {
          itemWidth: titleFontSize / 2,
          itemHeight: titleFontSize / 2,
          itemGap: titleFontSize / 2,
          textStyle: {
            fontSize: titleFontSize / 2
          }
        }
      }
      this.chartInstance.setOption(adapterOption)
      this.chartInstance.resize()
    },
    revertMap() {
      const revertOption = {
        geo: {
          map: 'china'
        }
      }
      this.chartInstance.setOption(revertOption)
    }
  }
}
</script>

<style scoped>

</style>
