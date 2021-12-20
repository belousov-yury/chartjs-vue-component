<template>
  <div class="chart-container">
    <vue3-chart-js
        :id="lineChart.id"
        :type="lineChart.type"
        :data="lineChart.data"
        :options="lineChart.options"
        :plugins="lineChart.plugins"
        ref="chartRef"

    />
    <div v-if="isControlPanel" class="control-container">

      <div class="markers">
        <div v-if="markerEditing">
          <button @click="saveMarkerValue">Сохранить маркер</button>
          <button @click="selectToMaximum">Установить на максимум</button>
        </div>
        <button v-else @click="addMarker">Добавить маркер</button>
        <div v-for="marker in markersList" :key="markersList.indexOf(marker)">
          <div>
            {{ markersList.indexOf(marker) + 1 }}.
            {{ marker }}
            <button @click="deleteMarker(marker)">✖</button>
          </div>
        </div>
        <div class="markers__markers-list">

        </div>
      </div>
      <button @click="exportChartImage">Сохранить график</button>

    </div>
  </div>
</template>

<script>
import Vue3ChartJs from '@j-t-mcc/vue3-chartjs'
import zoomPlugin from 'chartjs-plugin-zoom'
import ChartDataLabels from 'chartjs-plugin-datalabels'
import {onMounted, onUnmounted, ref, toRefs, watch} from "vue";

Vue3ChartJs.registerGlobalPlugins([zoomPlugin, ChartDataLabels])

export default {
  name: 'Home',
  components: {
    Vue3ChartJs,
  },
  props: {
    chartData1: {
      type: Array,
      default() {
        return [
          {x: 0, y: 50},
          {x: 2, y: 60},
          {x: 4, y: 50},
          {x: 6, y: 30},
          {x: 20, y: 19},
          {x: 22, y: 70},
          {x: 24, y: 13},
          {x: 26, y: 15},
          {x: 42, y: 3},
          {x: 44, y: 60},
          {x: 46, y: 3},
          {x: 48, y: 32},
          {x: 60, y: 44},
          {x: 62, y: 5},
          {x: 64, y: 56},
          {x: 66, y: 5},
          {x: 80, y: 22},
          {x: 82, y: 33},
          {x: 84, y: 2},
          {x: 86, y: 12},
          {x: 102, y: 22},
          {x: 104, y: 27},
          {x: 106, y: 80},
          {x: 108, y: 55},
        ]
      }
    },
    chartData2: {
      type: Array,
      default() {
        return []
      }
    },
    xAxisName: {
      type: String,
      default: 'Частота'
    },
    yAxisName: {
      type: String,
      default: 'Амплитуда'
    },
    legendName1: {
      type: String,
      default: 'Спектр 1'
    },
    legendName2: {
      type: String,
      default: 'Спектр 2'
    },
    isControlPanel: {
      type: Boolean,
      default: true
    }
  },
  methods: {},
  setup(props) {
    const chartRef = ref(null)
    let markerEditing = ref(false)

    const {
      chartData1,
      chartData2,
      xAxisName,
      yAxisName,
      legendName1,
      legendName2
    } = toRefs(props)

    const getMaxIndex = (array) => {
      let max = array[0].y
      let index = 0
      array.forEach((value, ind) => {
        if (value.y > max) {
          index = ind
          max = value.y
        }
      })
      return index
    }
    let maximum = getMaxIndex(chartData1.value)
    let currentIndex = maximum
    let selectedIndexes = ref([])
    let markersList = ref([])

    let datasets = []
    if (chartData1.value.length) datasets.push({
      label: legendName1.value,
      data: chartData1.value,
    })

    if (chartData2.value.length) datasets.push({
      label: legendName2.value,
      data: chartData2.value,
    })

    const lineChart = {
      id: 'line',
      type: 'scatter',
      data: {
        datasets: datasets
      },
      options: {
        interaction: {
          intersect: false,
          mode: 'index',
        },
        pointBackgroundColor: 'rgba(255, 159, 64, 1)',
        pointBorderColor: 'rgba(153, 102, 255, 0.5)',
        pointRadius: 4,
        borderColor: [
          'rgb(88,149,44)',
          'rgb(194,19,48)',
        ],
        pointBorderWidth: 3,
        borderWidth: 2,

        showLine: true,
        layout: {
          padding: 0
        },
        scales: {
          x: {
            title: {
              display: true,
              text: xAxisName.value,
              color: 'rgb(204,203,203)'
            },
            min: 0,
            grid: {
              color: 'rgb(100,100,97)',
              z: 0,
              tickColor: 'rgb(222,222,222)',
            },
            ticks: {
              color: 'rgb(222,222,222)',
            }
          },
          y: {
            title: {
              display: true,
              text: yAxisName.value,
              color: 'rgb(204,203,203)',
            },
            ticks: {
              color: 'rgb(222,222,222)',
            },
            min: 0,
            max: 100
            ,
            grid: {
              color: 'rgb(100,100,97)',
              z: 0,
              tickColor: 'rgb(222,222,222)'
            }
          },
        },
        plugins: {
          zoom: {
            pan: {
              enabled: true,
              threshold: 10,
            },
            zoom: {
              mode: 'xy',
              overScaleMode: 'x',
              wheel: {
                enabled: true,
              },
            },
          },
          legend: {
            labels: {
              color: 'rgb(204,203,203)',
              boxHeight: 0,
            }
            // onClick: () => console.log('kek')
          },
          tooltip: {
            intersect: false,
            displayColors: false,
            position: 'nearest',
          },
          datalabels: {
            backgroundColor: 'rgba(0,0,0,0)',
            borderRadius: 10,
            align: 'end',
            textAlign: 'center',
            display: (ctx) => {
              if (markerEditing.value) {
                if (selectedIndexes.value.includes(ctx.dataIndex)) return 1
                if (ctx.datasetIndex === 0) return ctx.dataIndex === currentIndex ? 1 : 0
                else return 0
              } else {
                if (ctx.datasetIndex === 0) {
                  let flag = false
                  selectedIndexes.value.forEach(value => {
                    if (ctx.dataIndex === value) {
                      flag = true
                    }
                  })
                  return flag ? 1 : 0
                }
              }
            },
            formatter: (value) => {
              let ind = markersList.value.indexOf(value) !== -1 ? markersList.value.indexOf(value) + 1 : ''
              return `${ind}\n▼`
            },
            color: (ctx) => {
              if (markerEditing.value && currentIndex === ctx.dataIndex) {
                return 'red'
              } else {
                if (selectedIndexes.value.includes(ctx.dataIndex)) return 'rgb(235,190,107)'
              }
            },
            font: {
              size: 25,
              weight: 'bold'
            },
          }
        },
      },

      plugins: [{
        id: 'custom_canvas_background_color',
        beforeDraw: (chart) => {
          const ctx = chart.canvas.getContext('2d');
          ctx.save();
          ctx.globalCompositeOperation = 'destination-over';
          ctx.fillStyle = 'rgb(60,63,64)';
          ctx.fillRect(0, 0, chart.width, chart.height);
          ctx.restore();
        }
      }]
    }

    const exportChartImage = () => {
      let a = document.createElement('a')
      a.href = chartRef.value.chartJSState.chart.toBase64Image()
      let date = new Date()
      a.download = `Спектр_${date.toLocaleString()}.png`
      a.click()
      a = null
    }

    const addMarker = () => {
      markerEditing.value = true
      updateChart()
    }
    const deleteMarker = marker => {
      let delIndex = markersList.value.indexOf(marker)
      markersList.value.splice(delIndex, 1)
      selectedIndexes.value.splice(delIndex, 1)
      updateChart()
    }
    const saveMarkerValue = () => {
      if(markerEditing.value) {
        selectedIndexes.value.push(currentIndex)
        markersList.value = selectedIndexes.value.map(value => chartData1.value[value])
        markerEditing.value = false
        updateChart()
      }
    }
    const selectToMaximum = () => {
      currentIndex = maximum
      updateChart()
    }
    const rightSelectPoint = () => {
      currentIndex++
      updateChart()
    }
    const leftSelectPoint = () => {
      currentIndex--
      updateChart()
    }

    const updateChart = () => {
      chartRef.value.chartJSState.chart.update()
    }

    const keyHandler = e => {
      switch (e.code) {
        case 'ArrowRight':
          rightSelectPoint()
          break
        case 'ArrowLeft':
          leftSelectPoint()
          break
        case 'Enter':
          saveMarkerValue()
          break
      }
    }

    const clickHandler = () => {
      let chart = chartRef.value.chartJSState.chart
      const activeElement = chart.getActiveElements()[0]
      if (activeElement.datasetIndex === 0) {
        currentIndex = activeElement.index
        chart.update()
      }
    }
    watch(selectedIndexes, () => {
      console.log(selectedIndexes)

      console.log(markersList)
    })
    onMounted(() => {
          document.addEventListener('keydown', keyHandler)
          let chart = chartRef.value.chartJSState.chart
          chart.canvas.addEventListener('click', () => {
            clickHandler()
          })

          //
          // chart.tooltip.setActiveElements([{datasetIndex: 0, index: 5}, {datasetIndex: 0, index: 7}, {
          //   datasetIndex: 0,
          //   index: 10
          // }], {x: 10, y: 10})

          chart.update()
        }
    )
    onUnmounted(() => {
          document.removeEventListener('keydown', keyHandler)
          chartRef.value.chartJSState.chart.canvas.removeEventListener('click', () => {
            clickHandler()
          })
        }
    )

    return {
      lineChart,
      chartRef,
      markerEditing,
      markersList,
      exportChartImage,
      rightSelectPoint,
      leftSelectPoint,
      addMarker,
      saveMarkerValue,
      deleteMarker,
      selectToMaximum
    }
  }
  ,
}
</script>

<style scoped>
button {
  border: 1px solid #3c3f40;
  border-radius: 3px;
  background-color: #585a5c;
  color: #eaeaea;
  padding: 4px 5px 5px 5px;
  transition: all 70ms;
  font-size: 0.9em;
  height: 25px;
}

button:hover {
  box-shadow: 0 0 2px 1px rgb(100, 100, 100);
}

button:active {
  box-shadow: inset 0 0 15px 1px #2b2b2b;
  transform: translateY(1px);
}

button:disabled {
  background-color: #2b2b2b;
  opacity: 0.7;
}

button:disabled:hover {
  box-shadow: none;
}

.chart-container {
  color: #eaeaea;
  display: flex;
  flex-direction: column;
}

.control-container {
  background-color: rgb(60, 63, 64);
  border-top: 1px solid rgb(100, 100, 97);
  padding: 10px;
  display: grid;
  justify-items: center;
  align-items: center;
  grid-template-columns: 80% 20%;
}

.markers {
  width: 100%;
  height: 100%;
  display: flex;
}

.markers__markers-list {

}
</style>