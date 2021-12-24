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
          <button @click="setToMaximum">Установить на максимум</button>
        </div>
        <button v-else @click="addMarker">Добавить маркер</button>

        <div class="markers__markers-list">
          <div v-for="(marker, index) in markersList"
               :key="index"
               class="markers__markers-list--item"
               :class="{'item-select': marker?.isSelected}"
               @click="markerSelect(marker)">
            <div :style="'background-color:' +  marker?.color" style="width: 15px; height: 15px;"></div>
            <div>
              {{ index + 1 }}.
              {{ xAxisName }}: {{ marker.value.x }} {{ xAxisUnit }},
              {{ yAxisName }}: {{ marker.value.y }} {{ yAxisUnit }}
            </div>
            <button @click.stop="resetZoom">✎</button>
            <button @click.stop="deleteMarker(marker)">✖</button>
          </div>
        </div>
      </div>
      <div>
        <button @click="resetZoom">Сбросить масштаб</button>
        <button @click="exportChartImage">Сохранить график</button>
      </div>

    </div>
  </div>
</template>

<script>
import Vue3ChartJs from '@j-t-mcc/vue3-chartjs'
import zoomPlugin from 'chartjs-plugin-zoom'
import ChartDataLabels from 'chartjs-plugin-datalabels'
import annotationPlugin from 'chartjs-plugin-annotation';
import {onMounted, onUnmounted, ref, toRefs} from "vue";

Vue3ChartJs.registerGlobalPlugins([zoomPlugin, ChartDataLabels, annotationPlugin])

export default {
  name: 'Home',
  components: {
    Vue3ChartJs,
  },
  props: {
    chartData1: {
      type: Array,
      default: () => [
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
    },
    chartData2: {
      type: Array,
      default: () => []
    },
    xAxisName: {
      type: String,
      default: 'Частота'
    },
    yAxisName: {
      type: String,
      default: 'Амплитуда'
    },
    xAxisUnit: {
      type: String,
      default: 'МГц'
    },
    yAxisUnit: {
      type: String,
      default: 'В'
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

    const colors = [
      'rgb(255,211,41)',
      'rgb(38,90,255)',
      'rgb(255,93,51)',
      'rgb(227,41,160)',
      'rgb(125,93,218)',
      'rgb(89,255,203)'
    ]

    const {
      chartData1,
      chartData2,
      xAxisName,
      yAxisName,
      xAxisUnit,
      yAxisUnit,
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
        animation: {
          duration: 500
        },
        pointBackgroundColor: (ctx) => {
          if (ctx.index === currentIndex) {
            return 'rgb(215,37,31)'
          } else {
            return 'rgba(255, 159, 64, 1)'
          }
        },
        // pointBorderColor: 'rgba(69,111,170, 0.4)',
        pointBorderColor: 'transparent',
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
              text: xAxisName.value + ',' + xAxisUnit.value,
              color: 'rgb(204,203,203)'
            },
            min: chartData1.value[0]?.x - 1,
            max: chartData1.value[chartData1.value.length - 1]?.x + 1,
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
              text: yAxisName.value + ',' + yAxisUnit.value,
              color: 'rgb(204,203,203)',
            },
            ticks: {
              color: 'rgb(222,222,222)',
            },
            max: chartData1.value[getMaxIndex(chartData1.value)]?.y + 7,
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
              drag: {
                enabled: true,
                modifierKey: 'ctrl'
              },
              animation: {
                duration: 1000,
                easing: 'easeOutCubic'
              }
            },
            drag: {
              enabled: true,
              threshold: 1
            }
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
          annotation: {
            annotations: {
              annotation2: {
                type: 'point',
                backgroundColor: 'rgb(215,37,31)',
                borderColor: 'transparent',
                pointStyle: 'triangle',
                radius: 12,
                rotation: 180,
                scaleID: 'y',
                yAdjust: -22,
                xValue: (ctx) => value(ctx, 1, 4, 'x'),
                yValue: (ctx) => value(ctx, 1, 4, 'y')
              }
            }
          },
          datalabels: {
            labels: {
              index: {
                backgroundColor: 'rgba(0,0,0,0)',
                borderRadius: 30,
                textShadowBlur: 2,
                padding:{
                  bottom: 0
                },
                borderColor: ctx => {
                  if (getMarker(ctx.dataIndex).isSelected) {
                    return colors[getMarkerId(ctx.dataset.data[ctx.dataIndex])]
                  }
                  return 'rgba(0,0,0,1)'
                },
                borderWidth: ctx => {
                  if (getMarker(ctx.dataIndex).isSelected) {
                    return 2
                  }
                  return 0
                },
                textShadowColor: 'rgba(0,0,0,1)',
                align: 'end',
                textAlign: 'center',
                display: (ctx) => {
                  if (isMarkerToIndex(ctx.dataIndex) || (ctx.dataIndex === currentIndex && markerEditing.value)) {
                    return 1
                  } else {
                    return 0
                  }
                },
                formatter: (value, ctx) => {
                  const testId = getMarkerId(value)
                  let ind = testId !== -1 ? testId + 1 : ''
                  if (ctx.dataIndex === currentIndex) {
                    return `${ind}`
                  } else {
                    return `${ind}`
                  }
                },
                color: (ctx) => {
                  if (isMarkerToIndex(ctx.dataIndex)) {
                    return colors[getMarkerId(ctx.dataset.data[ctx.dataIndex])]
                  }
                },
                font: {
                  size: 25,
                  weight: 'bold'
                },
                // offset: ctx => {
                //   if (isMarkerToIndex(ctx.dataIndex) && ctx.dataIndex === currentIndex) {
                //     return 30
                //   } else {
                //     return 0
                //   }
                // },
                offset: 22
              },
              mark: {
                backgroundColor: 'rgba(0,0,0,0)',
                borderRadius: 30,
                textShadowBlur: 2,
                padding:{
                  bottom: 0
                },
                textShadowColor: 'rgba(0,0,0,1)',
                align: 'end',
                textAlign: 'center',
                display: (ctx) => {
                  if (isMarkerToIndex(ctx.dataIndex) || (ctx.dataIndex === currentIndex && markerEditing.value)) {
                    return 1
                  } else {
                    return 0
                  }
                },
                formatter: (value, ctx) => {
                  if (ctx.dataIndex === currentIndex) {
                    return ``
                  } else {
                    return `▼`
                  }
                },
                color: (ctx) => {
                  if (isMarkerToIndex(ctx.dataIndex)) {
                    return colors[getMarkerId(ctx.dataset.data[ctx.dataIndex])]
                  }
                },
                font: {
                  size: 25,
                  weight: 'bold'
                },
                offset: ctx => {
                  if (isMarkerToIndex(ctx.dataIndex) && ctx.dataIndex === currentIndex) {
                    return 30
                  } else {
                    return 0
                  }
                }
              }
            }


          },
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

    function value(ctx, datasetIndex, index, prop) {
      const meta = ctx.chart.getDatasetMeta(0)
      const parsed = meta.controller.getParsed(currentIndex)
      return parsed ? parsed[prop] : NaN
    }

    const isMarkerToValue = (testValue) => {
      let flag = false
      markersList.value.forEach(item => {
        if (item.value.x === testValue.x && item.value.y === testValue.y) flag = true
      })
      return flag
    }

    const isMarkerToIndex = index => {
      let flag = false
      markersList.value.forEach(item => {
        if (item.dataIndex === index) flag = true
      })
      return flag
    }

    const getMarkerId = (value) => {
      let id = -1
      if (markersList.value) {
        markersList.value.forEach((item, index) => {
          if (item.value.x === value.x && item.value.y === value.y) id = index
        })
      }
      return id
    }
    const getMarker = dataIndex => {
      let marker = null
      markersList.value.forEach(item => {
        if (item.dataIndex === dataIndex) {
          marker = item
        }
      })
      return marker
    }

    const exportChartImage = () => {
      let a = document.createElement('a')
      a.href = chartRef.value.chartJSState.chart.toBase64Image()
      let date = new Date()
      a.download = `Спектр_${date.toLocaleString()}.png`
      a.click()
      a = null
    }
    const resetZoom = () => {
      chartRef.value.chartJSState.chart.resetZoom('none')
    }

    const addMarker = () => {
      if (markersList.value.length < 6) {
        markerEditing.value = true
        updateChart()
      } else {
        markerEditing.value = false
        alert('Максимальное число маркеров: 6')
      }
    }

    const deleteMarker = marker => {
      let delIndex = markersList.value.indexOf(marker)
      markersList.value.splice(delIndex, 1)
      markersList.value.forEach((item, index) => item.color = colors[index])
      updateChart()

    }

    const saveMarkerValue = () => {
      if (markersList.value.length < 6) {
        let newMarker = {
          value: chartData1.value[currentIndex],
          isSelected: false,
          dataIndex: currentIndex,
          color: colors[markersList.value.length]
        }
        if (markerEditing.value && !isMarkerToValue(newMarker.value)) {
          markersList.value.push(newMarker)
          markerEditing.value = false
        } else {
          alert('Данный маркер уже установлен')
          markerEditing.value = false
        }
        updateChart()
      } else {
        markerEditing.value = false
        alert('Максимальное число маркеров: 6')
      }
    }

    const changeMarkerValue = (marker) => {
      console.log(marker)
    }

    const markerSelect = (marker) => {
      let ind = getMarkerId(marker.value)
      markersList.value[ind].isSelected = !markersList.value[ind].isSelected
      updateChart()
    }

    const setToMaximum = () => {
      currentIndex = maximum
      updateChart()
    }
    const rightSelectPoint = () => {
      if (currentIndex < chartData1.value.length - 1) {
        currentIndex++
      }
      updateChart()
    }
    const leftSelectPoint = () => {
      if (currentIndex > 0) {
        currentIndex--
      }
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
    const dblClickHandler = () => {
      let chart = chartRef.value.chartJSState.chart
      const activeElement = chart.getActiveElements()[0]
      if (activeElement.datasetIndex === 0) {
        currentIndex = activeElement.index
        markerEditing.value = true
        saveMarkerValue()
      }
    }
    onMounted(() => {
          document.addEventListener('keydown', keyHandler)
          let chart = chartRef.value.chartJSState.chart
          chart.canvas.addEventListener('click', clickHandler)
          chart.canvas.addEventListener('dblclick', dblClickHandler)
          chart.update()
        }
    )
    onUnmounted(() => {
          document.removeEventListener('wheel', keyHandler)
          chartRef.value.chartJSState.chart.canvas.removeEventListener('click', clickHandler)
          chartRef.value.chartJSState.chart.canvas.removeEventListener('dblclick', dblClickHandler)
        }
    )

    return {
      lineChart,
      chartRef,
      markerEditing,
      markersList,
      resetZoom,
      exportChartImage,
      rightSelectPoint,
      leftSelectPoint,
      addMarker,
      saveMarkerValue,
      deleteMarker,
      changeMarkerValue,
      markerSelect,
      setToMaximum
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
  display: flex;
  flex-direction: column;
}

.markers__markers-list--item {
  display: flex;
  align-items: center;
  min-width: 30%;
  cursor: pointer;
  width: 100%;
}

.item-select {
  background-color: red;
}
</style>