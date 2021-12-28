<template>
  <div class="chart-container" :class="[darkTheme ? 'chart-container-dark': 'chart-container-light']"
       ref="containerRef">
    <vue3-chart-js
        :id="lineChart.id"
        :type="lineChart.type"
        :data="lineChart.data"
        :options="lineChart.options"
        :plugins="lineChart.plugins"
        ref="chartRef"

    />
    <div v-if="isControlPanel" class="control-container"
         :class="[darkTheme ? 'control-container-dark': 'control-container-light']">
      <div class="markers-control">
        <button @click="addMarker" :class="[darkTheme ? 'button-dark': 'button-light']">Добавить маркер</button>
        <button @click="deleteAllMarker" :class="[darkTheme ? 'button-dark': 'button-light']">Удалить маркеры</button>
        <button @click="cancelMarkersSelect" :class="[darkTheme ? 'button-dark': 'button-light']">Отменить выделения
        </button>
      </div>


      <div class="markers-list">
        <div class="markers-list__container">
          <div v-for="marker in markersList"
               :key="marker.id"
               class="markers-list__item"
               :class="{'item-select': marker?.isSelected}"
               @click="markerSelect(marker)"
               @dblclick="setZoomToMarker(marker)"
          >
            <div :style="'background-color:' +  marker?.color"
                 style="width: 15px; height: 15px; border-radius: 3px;"
                 class="markers-list__item--color"
            >
            </div>
            <div class="markers-list__item--text">
              {{ marker.id + 1 }}.
              {{ xAxisName }}: {{ marker.value.x }} {{ xAxisUnit }},
              {{ yAxisName }}: {{ marker.value.y }} {{ yAxisUnit }}
            </div>
            <button class="markers-list__item--button" :class="[darkTheme ? 'button-dark': 'button-light']"
                    @click.stop="deleteMarker(marker)">✖
            </button>
          </div>
        </div>
      </div>

      <div v-if="isDelta" class="delta">
        <div class="delta__item">{{ yAxisName }} (дельта): {{ delta.y }} {{ yAxisUnit }}</div>
        <div class="delta__item">{{ xAxisName }} (дельта): {{ delta.x }} {{ xAxisUnit }}</div>
      </div>

      <div class="set-control">
        <button @click="setToMaximum" :class="[darkTheme ? 'button-dark': 'button-light']" :disabled="!markerEditing">Установить на максимум
        </button>
        <button @click="setToMaximumLeft" :class="[darkTheme ? 'button-dark': 'button-light']" :disabled="!markerEditing">Максимум влево</button>
        <button @click="setToMaximumRight" :class="[darkTheme ? 'button-dark': 'button-light']" :disabled="!markerEditing">Максимум вправо</button>
      </div>
      <div class="chart-control">
        <button @click="resetZoom" :class="[darkTheme ? 'button-dark': 'button-light']">Сбросить масштаб</button>
        <button @click="changeInterpolation" :class="[darkTheme ? 'button-dark': 'button-light']">Интерполяция: {{isInterpolation ? 'вкл.' : 'выкл.'}}</button>
        <button @click="exportChartImage" :class="[darkTheme ? 'button-dark': 'button-light']">Сохранить график</button>
      </div>

    </div>
  </div>
</template>

<script>
import Vue3ChartJs from '@j-t-mcc/vue3-chartjs'
import zoomPlugin from 'chartjs-plugin-zoom'
import ChartDataLabels from 'chartjs-plugin-datalabels'
import annotationPlugin from 'chartjs-plugin-annotation';
import {onMounted, onUnmounted, ref, toRefs, watch} from "vue";

Vue3ChartJs.registerGlobalPlugins([zoomPlugin, ChartDataLabels, annotationPlugin])

export default {
  name: 'Home',
  components: {
    Vue3ChartJs,
  },
  props: {
    chartData1: {
      type: Array,
      default: () => {
        let arr = []
        for(let i = 1; i < 500; i++) {
          arr.push({x: i, y: Math.random() * 200})
        }
        return arr
      }
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
      default: 'дБ'
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
    },
    darkTheme: {
      type: Boolean,
      default: false
    }
  },
  methods: {},
  setup(props) {
    const chartRef = ref(null)
    const containerRef = ref(null)

    let markerEditing = ref(false)
    let isDelta = ref(false)
    let isInterpolation = ref(false)

    const markersColors = [
      'rgb(255,211,41)',
      'rgb(38,90,255)',
      'rgb(255,93,51)',
      'rgb(227,41,160)',
      'rgb(132,0,255)',
      'rgb(89,255,203)'
    ]

    const darkColors = {
      chartBackground: 'rgb(60,63,64)',
      legend: 'rgb(204,203,203)',
      axis: 'rgb(204,203,203)',
      grid: 'rgb(100,100,97)',
      line: [
        'rgb(43,171,0)',
        'rgb(194,19,48)',
      ]
    }

    const lightColors = {
      chartBackground: 'rgb(255,255,255)',
      legend: 'rgb(0,0,0)',
      axis: 'rgb(0,0,0)',
      grid: '#8d9091',
      line: [
        'rgb(88,149,44)',
        'rgb(206,0,40)',
      ]
    }

    const {
      chartData1,
      chartData2,
      xAxisName,
      yAxisName,
      xAxisUnit,
      yAxisUnit,
      legendName1,
      legendName2,
      darkTheme
    } = toRefs(props)

    watch(() => darkTheme.value, () => updateChart())

    const updateMaximumList = (data) => {
      maximumList = []
      maximumList.push(mainMaximum)

      if (mainMaximum > 1) {
        let earlySymbol = '-'
        for (let i = mainMaximum - 1; i >= 0; i--) {
          let symbol
          if (data[i].y < data[i + 1].y) {
            symbol = '-'
          } else {
            symbol = '+'
          }
          if (earlySymbol === '+' && symbol === '-') {
            maximumList.unshift(i + 1)
          }
          earlySymbol = symbol
        }
      }
      if (data.length - mainMaximum + 1 > 1) {
        let earlySymbol = '-'
        for (let i = mainMaximum + 1; i < data.length; i++) {
          let symbol
          if (data[i].y < data[i - 1].y) {
            symbol = '-'
          } else {
            symbol = '+'
          }
          if (earlySymbol === '+' && symbol === '-') {
            maximumList.push(i - 1)
          }
          earlySymbol = symbol
        }
      }
    }

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
    let maximumList = []
    let mainMaximum = getMaxIndex(chartData1.value)
    updateMaximumList(chartData1.value)

    let currentIndex = mainMaximum
    let markersList = ref([])
    let changeMarkerIndex = null
    let markersSelectedCount = 0

    let delta = ref({
      x: 0,
      y: 0
    })

    let datasets = []
    if (chartData1.value.length) datasets.push({
      label: legendName1.value,
      data: chartData1.value,
      cubicInterpolationMode: () => isInterpolation.value ? 'monotone' : '',
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
          duration: 0
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
          darkTheme.value ? darkColors.line[0] : lightColors.line[0],
          darkTheme.value ? darkColors.line[1] : lightColors.line[1],
        ],
        pointBorderWidth: 3,
        borderWidth: 2,

        showLine: true,
        layout: {
          padding: 0
        },
        scales: {
          x: {
            title: () => {
              return {
                display: true,
                text: xAxisName.value + ',' + xAxisUnit.value,
                color: darkTheme.value ? darkColors.axis : lightColors.axis
              }
            },
            min: chartData1.value[0]?.x - 1,
            max: chartData1.value[chartData1.value.length - 1]?.x + 1,
            grid: () => {
              return {
                color: darkTheme.value ? darkColors.grid : lightColors.grid,
                z: 0,
                tickColor: darkTheme.value ? darkColors.axis : lightColors.axis
              }
            },
            ticks: {
              color: () => {
                return darkTheme.value ? darkColors.axis : lightColors.axis
              },
              callback: (value) => value.toFixed(1)
            }
          },
          y: {
            title: () => {
              return {
                display: true,
                text: yAxisName.value + ',' + yAxisUnit.value,
                color: darkTheme.value ? darkColors.axis : lightColors.axis,
              }
            },
            ticks: {
              color: () => {
                return darkTheme.value ? darkColors.axis : lightColors.axis
              },
              callback: (value) => value.toFixed(1)
            },
            max: chartData1.value[getMaxIndex(chartData1.value)]?.y + 7,
            grid: () => {
              return {
                color: darkTheme.value ? darkColors.grid : lightColors.grid,
                z: 0,
                tickColor: darkTheme.value ? darkColors.axis : lightColors.axis
              }
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
            labels: () => {
              return {
                color: darkTheme.value ? darkColors.legend : lightColors.legend,
                boxHeight: 0
              }
            }
            // onClick: () => console.log('kek')
          },
          tooltip: {
            intersect: false,
            displayColors: false,
            position: 'nearest',
            callbacks: {
              label: (ctx) => {
                return `${xAxisName.value}: ${ctx.label} ${xAxisUnit.value}\n`
              },
              title: (ctx) => `${yAxisName.value}: ${ctx[0].formattedValue} ${yAxisUnit.value}`
            }
          },
          annotation: {
            annotations: {
              annotation2: {
                display: () => markerEditing.value ? 1 : 0,
                type: 'point',
                backgroundColor: 'rgb(215,37,31)',
                borderColor: 'transparent',
                pointStyle: 'triangle',
                radius: () => {
                  return chartRef.value.chartRef.width / 160
                },
                scaleID: 'y',
                yAdjust: () => {
                  return chartRef.value.chartRef.width / 80
                },
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
                textStrokeWidth: 2,
                textStrokeColor: 'rgba(0,0,0,0.7)',
                align: 'end',
                textAlign: 'center',
                padding: {
                  bottom: 0
                },
                borderColor: ctx => {
                  if (getMarker(ctx.dataIndex)?.isSelected) {
                    return markersColors[getMarkerId(ctx.dataset.data[ctx.dataIndex])]
                  }
                  return 'rgba(0,0,0,0)'
                },
                borderWidth: ctx => {
                  if (getMarker(ctx.dataIndex)?.isSelected) {
                    return 3
                  }
                  return 0
                },
                display: (ctx) => {
                  if (isMarkerToIndex(ctx.dataIndex) || (ctx.dataIndex === currentIndex && markerEditing.value)) {
                    return 1
                  } else {
                    return 0
                  }
                },
                formatter: (value) => {
                  const testId = getMarkerId(value)
                  let ind = testId !== -1 ? testId + 1 : ''
                  return `${ind}`
                },
                color: (ctx) => {
                  if (isMarkerToIndex(ctx.dataIndex)) {
                    return markersColors[getMarkerId(ctx.dataset.data[ctx.dataIndex])]
                  }
                },
                font: () => {
                  return {
                    size: getFontSize(),
                    weight: 'bold'
                  }
                },
                offset: () => {
                  let width = chartRef.value.chartRef.width
                  return width / 79
                }
              },
              mark: {
                backgroundColor: 'rgba(0,0,0,0)',
                borderRadius: 30,
                textStrokeWidth: 2,
                textStrokeColor: 'rgba(0,0,0,0.7)',
                align: 'end',
                textAlign: 'center',
                display: (ctx) => {
                  if (isMarkerToIndex(ctx.dataIndex)) {
                    return 1
                  } else {
                    return 0
                  }
                },
                formatter: () => {
                  return `▼`
                },
                color: (ctx) => {
                  if (isMarkerToIndex(ctx.dataIndex)) {
                    return markersColors[getMarkerId(ctx.dataset.data[ctx.dataIndex])]
                  }
                },
                font: () => {
                  return {
                    size: getFontSize(),
                    weight: 'bold'
                  }
                },
                offset: 0
              }
            }
          },
        },
      },

      plugins: [{
        id: 'custom_canvas_background_color',
        beforeDraw: (chart) => {
          const ctx = chart.canvas.getContext('2d')
          ctx.save()
          ctx.globalCompositeOperation = 'destination-over'
          ctx.fillStyle = darkTheme.value ? darkColors.chartBackground : lightColors.chartBackground
          ctx.fillRect(0, 0, chart.width, chart.height)
          ctx.restore()
        }
      }]
    }

    function value(ctx, datasetIndex, index, prop) {
      const meta = ctx.chart.getDatasetMeta(0)
      const parsed = meta.controller.getParsed(currentIndex)
      return parsed ? parsed[prop] : NaN
    }

    // const isMarkerToValue = (testValue) => {
    //   let flag = false
    //   markersList.value.forEach(item => {
    //     if (item.value.x === testValue.x && item.value.y === testValue.y) flag = true
    //   })
    //   return flag
    // }

    const calcDelta = () => {
      let markers = markersList.value.filter(item => item.isSelected)
      if (markers.length === 2) {
        delta.value.x = Math.abs(markers[1].value.x - markers[0].value.x)
        delta.value.y = Math.abs(markers[1].value.y - markers[0].value.y)
      }
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
        cancelMarkersSelect()
        saveMarkerValue()
        updateChart()
      } else {
        markerEditing.value = false
        alert('Максимальное число маркеров: 6')
      }
    }

    const deleteMarker = marker => {
      let delIndex = markersList.value.indexOf(marker)
      markersList.value.splice(delIndex, 1)
      markersList.value.forEach((item, index) => {
        item.color = markersColors[index]
        item.id = index
      })
      updateChart()
    }

    const deleteAllMarker = () => {
      cancelMarkersSelect()
      markersList.value = []
      updateChart()
    }

    const saveMarkerValue = () => {
      if (markersList.value.length < 6) {
        let newMarker = {
          id: markersList.value.length,
          value: chartData1.value[currentIndex],
          isSelected: false,
          dataIndex: currentIndex,
          color: markersColors[markersList.value.length]
        }
        markersList.value.push(newMarker)
        markerEditing.value = false
        updateChart()
      } else {
        markerEditing.value = false
        alert('Максимальное число маркеров: 6')
      }
    }

    const changeMarkerValue = () => {
      markersList.value.forEach((item, index) => {
        if (item.isSelected) {
          changeMarkerIndex = index
          currentIndex = item.dataIndex
        }
      })
    }
    const moveSelectedMarker = () => {
      if (markersSelectedCount === 1) {
        markersList.value[changeMarkerIndex].dataIndex = currentIndex
        markersList.value[changeMarkerIndex].value = chartData1.value[currentIndex]
      }
    }

    const markerSelect = (marker) => {
      let ind = marker.id
      markersList.value[ind].isSelected = !markersList.value[ind].isSelected
      if (markersList.value[ind].isSelected) {
        let chart = chartRef.value.chartJSState.chart
        chart.setActiveElements([{datasetIndex: 0, index: marker.dataIndex}])
        markersSelectedCount++
      } else {
        markersSelectedCount--
      }
      if (markersSelectedCount === 1) {
        changeMarkerValue()
        markerEditing.value = true
      } else {
        markerEditing.value = false
      }
      if (markersSelectedCount === 2) {
        isDelta.value = true
        calcDelta()
      } else {
        isDelta.value = false
      }
      updateChart()
    }

    const cancelMarkersSelect = () => {
      markersList.value.forEach(item => item.isSelected = false)
      markersSelectedCount = 0
      markerEditing.value = false
      updateChart()
    }

    const setToMaximum = () => {
      if (markersSelectedCount === 1) {
        currentIndex = mainMaximum
        moveSelectedMarker()
        updateChart()
      }
    }

    const setToMaximumRight = () => {
      if (markersSelectedCount === 1) {
        let index = -1
        maximumList.some(item => {
          if (currentIndex < item) {
            index = item
            return true
          }
        })
        if (index !== -1) {
          currentIndex = index
          moveSelectedMarker()
          updateChart()
        }
      }
    }

    const setToMaximumLeft = () => {
      if (markersSelectedCount === 1) {
        let index = -1
        maximumList.forEach(item => {
          if (currentIndex > item) {
            index = item
          }
        })
        if (index !== -1 && index - 1 >= 0) {
          currentIndex = index
          moveSelectedMarker()
          updateChart()
        }
      }
    }

    const rightSelectPoint = () => {
      if (currentIndex < chartData1.value.length - 1) {
        currentIndex++
      }
      moveSelectedMarker()
      updateChart()
    }

    const leftSelectPoint = () => {
      if (currentIndex > 0) {
        currentIndex--
      }
      moveSelectedMarker()
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
        case 'Escape':
          cancelMarkersSelect()
      }
    }

    const clickHandler = () => {
      let chart = chartRef.value.chartJSState.chart
      const activeElement = chart.getActiveElements()[0]
      let lastIndex = currentIndex
      if (activeElement?.datasetIndex === 0) {
        currentIndex = activeElement.index
      }
      if (markerEditing.value) {
        if (lastIndex !== currentIndex) {
          moveSelectedMarker()
        } else {
          let marker = getMarker(currentIndex)
          if (!marker.isSelected) cancelMarkersSelect()
          markerSelect(marker)
        }
      } else {
        if (isMarkerToIndex(currentIndex)) {
          markerSelect(getMarker(currentIndex))
        }
      }
      chart.update()
    }

    const dblClickHandler = (e) => {
      if (!markerEditing.value) {
        e.preventDefault()
        let chart = chartRef.value.chartJSState.chart
        const activeElement = chart.getActiveElements()[0]
        if (activeElement?.datasetIndex === 0) {
          currentIndex = activeElement.index
          markerEditing.value = true
          saveMarkerValue()
        }
      }
    }

    const getFontSize = () => {
      let width = chartRef.value.chartRef.width
      return width / 76.8
    }

    const changeInterpolation = () => {
      isInterpolation.value = !isInterpolation.value
      updateChart()
    }

    const setZoomToMarker = (marker) => {
      let chart = chartRef.value.chartJSState.chart
      chart.zoomScale('x',{min: marker.value.x - 50, max: marker.value.x + 50,},'default')
      chart.zoomScale('y',{min: marker.value.y - 50, max: marker.value.y + 50,},'default')
    }

    onMounted(() => {
      document.addEventListener('keydown', keyHandler)
      let chart = chartRef.value.chartJSState.chart
      chart.canvas.addEventListener('click', clickHandler)
      chart.canvas.addEventListener('dblclick', dblClickHandler)
      chart.update()
    })

    onUnmounted(() => {
      document.removeEventListener('wheel', keyHandler)
      chartRef.value.chartJSState.chart.canvas.removeEventListener('click', clickHandler)
      chartRef.value.chartJSState.chart.canvas.removeEventListener('dblclick', dblClickHandler)
    })

    return {
      lineChart,
      chartRef,
      containerRef,
      markerEditing,
      markersList,
      delta,
      isDelta,
      isInterpolation,
      resetZoom,
      exportChartImage,
      rightSelectPoint,
      leftSelectPoint,
      addMarker,
      saveMarkerValue,
      deleteMarker,
      deleteAllMarker,
      changeMarkerValue,
      markerSelect,
      cancelMarkersSelect,
      setToMaximum,
      setToMaximumRight,
      setToMaximumLeft,
      setZoomToMarker,
      changeInterpolation
    }
  },
}
</script>

<style scoped>
button {
  border-radius: 3px;
  padding: 4px 5px 5px 5px;
  transition: all 70ms;
  font-size: 0.9em;
  height: 25px;
}

.button-dark {
  border: 1px solid #3c3f40;
  background-color: #585a5c;
  color: #eaeaea;
}

.button-light {
  border: 1px solid #3c3f40;
  background-color: #f3f3f3;
  color: #0c0c0c;
}

.button-dark:hover {
  box-shadow: 0 0 2px 1px rgb(100, 100, 100);
}

.button-light:hover {
  box-shadow: 0 0 2px 1px rgb(100, 100, 100);
}

button:active {
  transform: translateY(1px);
}

.button-dark:active {
  box-shadow: inset 0 0 15px 1px #2b2b2b;
}

.button-light:active {
  box-shadow: inset 0 0 15px 1px #2b2b2b;
}

button:disabled {
  opacity: 0.7;
}

.button-dark:disabled {
  background-color: #2b2b2b;
}

.button-light:disabled {
  background-color: #2b2b2b;
}

button:disabled:hover {
  box-shadow: none;
}

.chart-container {
  font-size: 14px;
  display: flex;
  flex-direction: column;
}

.chart-container-dark {
  color: #eaeaea;
}

.chart-container-light {
  color: #0c0c0c;
}

.control-container {
  padding: 10px;
  display: flex;
  justify-content: space-around;
  justify-items: center;
  align-items: center;
}

.control-container-dark {
  background-color: rgb(60, 63, 64);
  border-top: 1px solid rgb(100, 100, 97);
}

.control-container-light {
  border-top: 1px solid rgb(100, 100, 97);
}

.markers-control {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.markers-control button {
  margin: 5px 0;
}

.markers-list {
  font-size: 0.9em;
  overflow: hidden;
  width: 25%;
}

.markers-list__container {
  height: 80px;
  user-select: none;
  overflow-y: auto;
  overflow-x: hidden;
  padding: 5px 0;
  box-shadow: inset 0 0 2px 0.3px #585a5c;
  border-radius: 4px;
}

.markers-list__container::-webkit-scrollbar {
  width: 8px;
}

.markers-list__container::-webkit-scrollbar-track {
  border-radius: 4px;

  box-shadow: inset 0 0 2px 0.3px #585a5c;
}

.markers-list__container::-webkit-scrollbar-thumb {
  background-color: #585a5c;
  border-radius: 4px;
}

.markers-list__item {
  display: grid;
  align-items: center;
  cursor: pointer;
  width: 100%;
  grid-template-columns: 10% 75% 15%;
  padding: 3px 5px;
  border-radius: 4px;
}

.markers-list__item--color {
  margin: 5px;
}

.markers-list__item--text {
  margin-left: 7px;
}

.markers-list__item--button {
  width: 25px;
  height: 25px;
  margin-left: 5px;
  justify-self: center;
}

.item-select {
  background-color: #bbbbbb;
  color: #2b2b2b;
}

.delta {
  font-size: 0.9em;
  height: 100%;
  display: flex;
  justify-content: space-between;
  flex-direction: column;
  align-items: center;
}

.delta__item {
  margin: 5px 0;
}

.set-control {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.set-control button {
  margin: 5px 0;
}

.chart-control {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.chart-control button {
  margin: 5px 0;
}


</style>