<template>
  <div class="medical-service-chart-container" :style="{ width: computedWidth }">
    <el-card class="chart-card futuristic-card">

<!--        <div class="card-header">-->
<!--          <h3 class="futuristic-title">{{ title }}</h3>-->
<!--          <div class="chart-controls">-->
<!--          </div>-->
<!--        </div>-->
      <div class="chart-title">{{ title }}</div>


      <div class="chart-wrapper">
        <div
            ref="chartRef"
            class="chart futuristic-chart"
            :style="{ height: height }"
        ></div>
        <canvas ref="particleCanvas" class="particle-canvas"></canvas>
        <div v-if="isLoading" class="loading-overlay">
          <div class="loading-spinner"></div>
        </div>
        <div v-if="!hasData" class="empty-placeholder">
          <i class="el-icon-data-analysis"></i>
          <span>暂无数据</span>
        </div>
      </div>
    </el-card>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, nextTick, onUnmounted, computed } from 'vue'
import * as echarts from 'echarts'
import { ElCard, ElSelect, ElOption } from 'element-plus'

const props = defineProps({
  chartData: {
    type: Array,
    default: () => []
  },
  title: {
    type: String,
    default: '医疗服务统计'
  },
  height: {
    type: String,
    default: '350px'
  },
  width: {
    type: [String, Number],
    default: '100%'
  },
  loading: {
    type: Boolean,
    default: false
  },
  colorPalette: {
    type: Array,
    default: () => ['#7DF9FF', '#00FFE0', '#00B4D8', '#48CAE4', '#90E0EF']
    // 修改了第二个颜色为蓝绿色(#00FFE0)
  }
})

const chartRef = ref(null)
const particleCanvas = ref(null)
let chartInstance = null
let particleCtx = null
let animationFrameId = null
const availableFields = ref([])
const valueFields = ref([])
const selectedXField = ref('year')
const selectedYFields = ref(['outpatientVisits', 'inpatientAdmissions'])
const isLoading = ref(props.loading)
// 添加计算属性处理宽度
const computedWidth = computed(() => {
  return typeof props.width === 'number' ? `${props.width}px` : props.width
})
// 计算属性
const hasData = computed(() => {
  return props.chartData && props.chartData.length > 0 && selectedXField.value && selectedYFields.value.length > 0
})
const globalParticles = ref({
  active: false,
  animationFrameId: null,
  particles: [],
  lastCleanup: 0,
  lastFrameTime: 0
})
// 辅助函数：将hex颜色转换为rgba
const hexToRgba = (hex, alpha) => {
  const r = parseInt(hex.slice(1, 3), 16)
  const g = parseInt(hex.slice(3, 5), 16)
  const b = parseInt(hex.slice(5, 7), 16)
  return `rgba(${r}, ${g}, ${b}, ${alpha})`
}
// 初始化粒子系统
const initParticles = () => {
  if (!particleCanvas.value || !hasData.value) return

  // 如果粒子系统已经在运行，先停止它
  if (globalParticles.value.active) {
    stopParticles()
  }

  const canvas = particleCanvas.value
  canvas.width = canvas.offsetWidth
  canvas.height = canvas.offsetHeight

  const ctx = canvas.getContext('2d')

  // 粒子配置 - 使用MultipleLineChart风格的参数
  const config = {
    MAX_PARTICLES: 40,          // 粒子数量
    PARTICLE_SIZE: 1.5,         // 粒子基础大小
    SPEED: 0.2,                // 基础速度
    TRAIL_LENGTH: 15,          // 拖尾长度
    FPS: 30,                   // 目标帧率
    COLORS: ['#7DF9FF', '#00B4D8', '#90E0EF'] // 粒子颜色
  }

  // 粒子生成函数
  const createParticle = () => ({
    x: Math.random() * canvas.width,
    y: Math.random() * canvas.height,
    size: Math.random() * 1.5 + 0.5,
    speed: config.SPEED * (0.8 + Math.random() * 1.4),
    opacity: Math.random() * 0.4 + 0.2,
    direction: Math.random() * Math.PI * 2,
    color: config.COLORS[Math.floor(Math.random() * config.COLORS.length)],
    history: [],
    life: 1 + Math.random() * 0.3
  })

  // 标记为活跃
  globalParticles.value.active = true

  // 初始填充粒子
  globalParticles.value.particles = Array.from(
      { length: config.MAX_PARTICLES / 2 }, // 初始只填充一半粒子
      createParticle
  )

  const animate = (timestamp) => {
    if (!globalParticles.value.active) return

    // 控制帧率
    const deltaTime = timestamp - globalParticles.value.lastFrameTime
    if (deltaTime < 1000 / config.FPS) {
      globalParticles.value.animationFrameId = requestAnimationFrame(animate)
      return
    }
    globalParticles.value.lastFrameTime = timestamp

    // 清除画布 - 使用更透明的背景让拖尾更快消失
    ctx.fillStyle = 'rgba(3, 4, 94, 0.08)'
    ctx.fillRect(0, 0, canvas.width, canvas.height)

    // 定期清理死亡粒子
    const now = Date.now()
    if (now - globalParticles.value.lastCleanup > 2000) {
      globalParticles.value.particles = globalParticles.value.particles.filter(p => p.life > 0)
      globalParticles.value.lastCleanup = now
    }

    // 补充新粒子 (不超过最大值)
    if (globalParticles.value.particles.length < config.MAX_PARTICLES && Math.random() > 0.7) {
      globalParticles.value.particles.push(createParticle())
    }

    // 更新和绘制粒子
    globalParticles.value.particles.forEach(p => {
      // 更新生命周期
      p.life -= 0.002

      // 更新位置
      p.x += Math.cos(p.direction) * p.speed
      p.y += Math.sin(p.direction) * p.speed

      // 边界检查并反弹
      if (p.x < 0 || p.x > canvas.width) p.direction = Math.PI - p.direction
      if (p.y < 0 || p.y > canvas.height) p.direction = -p.direction

      // 创建粒子光晕效果 - MultipleLineChart风格
      const gradient = ctx.createRadialGradient(
          p.x, p.y, 0,
          p.x, p.y, p.size
      )
      gradient.addColorStop(0, `rgba(125, 249, 255, ${p.opacity * 0.7})`)
      gradient.addColorStop(0.7, `rgba(125, 249, 255, ${p.opacity * 0.2})`)
      gradient.addColorStop(1, 'rgba(125, 249, 255, 0)')

      ctx.beginPath()
      ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2)
      ctx.fillStyle = gradient
      ctx.fill()
    })

    globalParticles.value.animationFrameId = requestAnimationFrame(animate)
  }

  globalParticles.value.animationFrameId = requestAnimationFrame(animate)
}

// 停止粒子系统
const stopParticles = () => {
  if (globalParticles.value.animationFrameId) {
    cancelAnimationFrame(globalParticles.value.animationFrameId)
    globalParticles.value.animationFrameId = null
  }
  globalParticles.value.active = false
  globalParticles.value.particles = []

  // 清除画布
  if (particleCanvas.value) {
    const ctx = particleCanvas.value.getContext('2d')
    ctx.clearRect(0, 0, particleCanvas.value.width, particleCanvas.value.height)
  }
}
// 初始化图表
const initChart = () => {
  if (!chartRef.value) return

  if (chartInstance) {
    chartInstance.dispose()
  }

  chartInstance = echarts.init(chartRef.value, 'dark')
  updateChart()
}

// 更新图表数据
const updateChart = () => {
  if (!chartInstance || !hasData.value) return

  const xData = props.chartData.map(item => item[selectedXField.value])

  const series = selectedYFields.value.map((field, index) => ({
    name: field === 'outpatientVisits' ? '门诊量' : field === 'inpatientAdmissions' ? '住院量' : field,
    type: 'bar',
    data: props.chartData.map(item => item[field]),
    barWidth: '30%', // 减小柱形宽度
    barGap: '80%',   // 增加柱形组间间距
    barCategoryGap: '80%', // 增加同一类别柱形间间距
    itemStyle: {
      color: props.colorPalette[index % props.colorPalette.length],
      borderColor: '#03045E',
      borderWidth: 1,
      borderRadius: [4, 4, 0, 0],
      shadowBlur: 10,
      shadowColor: props.colorPalette[index % props.colorPalette.length]
    },
    emphasis: {
      focus: 'series',
      itemStyle: {
        // color: '#FF00FF',
        borderWidth: 2,
        shadowBlur: 15,
        shadowColor: 'rgba(255, 0, 255, 0.5)',
        opacity: 1 // 确保完全可见
      },
      // 禁用标签的淡化效果
      label: {
        opacity: 1,
        color: '#CAF0F8'
      }
    },
    blur: {
      itemStyle: {
        opacity: 0.95 // 未被点击的元素几乎不透明
      },
      label: {
        opacity: 0.90 // 标签完全不透明
      }
    },
    label: {
      show: true,
      position: 'top',
      color: '#CAF0F8',
      fontSize: 11,
      formatter: function(params) {
        // 直接根据数据索引决定是否显示标签
        return params.dataIndex % 2 === 0 ?
            (params.value > 1000 ? `${(params.value / 1000).toFixed(1)}k` : params.value) :
            '';
      },
      // 关键配置：确保标签不受高亮影响
      emphasis: {
        show: true,
        color: '#CAF0F8', // 保持与正常状态相同颜色
        fontSize: 11      // 保持相同大小
      }


    },
    animationType: 'scale',
    animationEasing: 'elasticOut',
    animationDelay: function (idx) {
      return idx * 50 + index * 100
    }
  }))

  const option = {
    backgroundColor: 'transparent',
    title: {
      // text: props.title,
      left: 'center',
      textStyle: {
        color: '#7DF9FF',
        fontSize: 16,
        fontWeight: 'bold'
      }
    },
    tooltip: {
      trigger: 'axis',
      axisPointer: {
        type: 'shadow'
      },
      backgroundColor: 'rgba(0, 20, 40, 0.9)',
      borderColor: '#00B4D8',
      borderWidth: 1,
      textStyle: {
        color: '#FFFFFF',
        fontSize: 12
      },
      formatter: (params) => {
        let result = `<div style="margin-bottom:5px;">
          <span style="color:#90E0EF;">年份: </span>
          <span style="color:#7DF9FF;">${params[0].axisValue}</span>
        </div>`

        params.forEach(param => {
          const value = param.value
          const displayValue = value > 1000 ? `${(value / 1000).toFixed(1)}k` : value
          result += `
            <div>
              <span style="color:#90E0EF;">${param.seriesName}: </span>
              <span style="color:#7DF9FF;font-weight:bold;">${displayValue}</span>
            </div>
          `
        })
        return result
      }
    },
    legend: {
      data: series.map(s => s.name),
      bottom: 5,
      textStyle: {
        color: '#CAF0F8',
        fontSize: 12
      }
    },
    grid: {
      left: '10%',
      right: '5%',
      bottom: '15%',
      top: '30%',
      containLabel: true
    },
    xAxis: {
      type: 'category',
      data: xData,
      axisLine: {
        lineStyle: {
          color: '#48CAE4',
          width: 2
        }
      },
      axisLabel: {
        color: '#90E0EF',
        rotate: xData.length > 8 ? 45 : 0,
        fontSize: 11,
        interval: Math.ceil(xData.length / 10)
      },
      axisTick: {
        alignWithLabel: true,
        lineStyle: {
          color: '#48CAE4'
        }
      }
    },
    yAxis: {
      type: 'value',
      name: '数量',
      nameTextStyle: {
        color: '#90E0EF',
        fontSize: 12
      },
      axisLine: {
        lineStyle: {
          color: '#48CAE4',
          width: 2
        }
      },
      axisLabel: {
        color: '#90E0EF',
        fontSize: 11,
        formatter: (value) => {
          return value > 1000 ? `${value / 1000}k` : value
        }
      },
      splitLine: {
        lineStyle: {
          color: 'rgba(72, 202, 228, 0.15)',
          width: 1  // 可以适当增加分割线宽度
        },
        interval: 'auto'  // 确保自动计算间隔
      },
      splitNumber: 4 , // 增加这个属性可以控制分割段数，使y轴刻度更稀疏
      axisTick: {
        lineStyle: {
          color: '#48CAE4'
        }
      }
    },
    series: series,
    animationDuration: 800
  }

  chartInstance.setOption(option, true)
}

// 提取可用字段
const extractAvailableFields = () => {
  if (props.chartData.length === 0) {
    availableFields.value = []
    valueFields.value = []
    return
  }

  const allFields = [...new Set(props.chartData.flatMap(Object.keys))]

  // 默认X轴字段为year
  availableFields.value = allFields
  valueFields.value = allFields.filter(f => f !== 'year')

  if (!selectedXField.value) {
    selectedXField.value = 'year'
  }

  if (selectedYFields.value.length === 0 && valueFields.value.length > 0) {
    selectedYFields.value = valueFields.value.slice(0, 2)
  }
}

// 响应式调整
const handleResize = () => {
  if (chartInstance) chartInstance.resize()
  if (particleCanvas.value) {
    particleCanvas.value.width = particleCanvas.value.offsetWidth
    particleCanvas.value.height = particleCanvas.value.offsetHeight
  }
}

onMounted(() => {
  nextTick(() => {
    extractAvailableFields()
    if (props.chartData.length > 0) {
      initChart()
      initParticles()
    }
  })

  // 添加对数据的主动监听
  watch(() => props.chartData, (newData) => {
    extractAvailableFields()
    if (newData.length > 0) {
      if (animationFrameId) {
        cancelAnimationFrame(animationFrameId)
        animationFrameId = null
        stopParticles()
      }
      stopParticles()
      nextTick(initChart)
      nextTick(initParticles)
    }
  }, { immediate: true, deep: true })

  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  if (chartInstance) chartInstance.dispose()
  stopParticles()
  if (animationFrameId) cancelAnimationFrame(animationFrameId)
  window.removeEventListener('resize', handleResize)
})

// 监听loading状态
watch(() => props.loading, (val) => {
  isLoading.value = val
})
</script>

<style scoped lang="scss">
$tech-blue: #03045E;
$tech-cyan: #7DF9FF;
$tech-lightblue: #90E0EF;
$tech-darkblue: #023E8A;
$tech-text: #CAF0F8;
.medical-service-chart-container {
  min-width: 300px;
  display: flex;
  flex-direction: column;
  flex-shrink: 0;
  background: linear-gradient(135deg, $tech-blue 0%, $tech-darkblue 100%);
  box-shadow: 0 2px 12px rgba(0, 119, 182, 0.3);
  border-radius: 4px;
  padding: 1px;
}
.futuristic-card {
  margin: 0;
  border: none;
  background: transparent;
  box-shadow: none;
  display: flex;
  flex-direction: column;
  flex-grow: 1;
  position: relative;

  :deep(.el-card__header) {
    padding: 0;
    border-bottom: none;
  }

  :deep(.el-card__body) {
    padding: 0 !important;
  }
}

.chart-title {
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  position: absolute;
  top: 35px;
  left: 0;
  right: 0;
  text-align: center;
  color: $tech-cyan;
  font-size: 14px;
  margin: 0;
  font-weight: 500;
  text-shadow: 0 0 8px rgba($tech-cyan, 0.3);
  z-index: 10;
  cursor: pointer; /* 添加指针样式 */
  text-decoration: none; /* 移除下划线 */

  &:hover {
    color: #0059ff; /* 悬停时改变颜色 */
    text-shadow: 0 0 10px rgba(255, 0, 255, 0.5); /* 悬停时增强阴影 */
  }
}
.futuristic-title {
  color: $tech-cyan;
  font-size: 18px;
  margin: 0 0 12px 0;
  font-weight: 500;
  text-shadow: 0 0 8px rgba($tech-cyan, 0.5);
}

.chart-controls {
  display: flex;
  gap: 20px;
  flex-wrap: wrap;
}

.control-group {
  display: flex;
  align-items: center;
  gap: 10px;
  min-width: 220px;

  .control-label {
    color: $tech-cyan;
    font-size: 14px;
    white-space: nowrap;
    text-shadow: 0 0 4px rgba($tech-cyan, 0.3);
  }
}

.futuristic-select {
  width: 180px;

  :deep(.el-input__wrapper) {
    background: rgba(0, 119, 182, 0.3) !important;
    border: 1px solid rgba(72, 202, 228, 0.5) !important;
    box-shadow: 0 0 8px rgba(125, 249, 255, 0.2) !important;
    height: 32px;

    .el-input__inner {
      color: $tech-text !important;
      font-size: 13px;

      &::placeholder {
        color: rgba($tech-lightblue, 0.7) !important;
      }
    }
  }

  :deep(.el-select__caret) {
    color: $tech-cyan !important;
  }
}

.chart-wrapper {
  position: relative;
  width: 100%;
  height: v-bind(height);
  min-height: 200px;
  flex-grow: 1;
  padding-top: 10px; /* 为标题留出空间 */
  //top: 10px;
}

.chart {
  width: 100%;
  height: 100%;
  position: relative;
  z-index: 2;
}

.particle-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
  pointer-events: none;

}

.loading-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(2, 62, 138, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 3;

  .loading-spinner {
    width: 30px;
    height: 30px;
    border: 3px solid rgba(125, 249, 255, 0.2);
    border-radius: 50%;
    border-top-color: #7DF9FF;
    animation: spin 1s linear infinite;
  }
}

.empty-placeholder {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
  color: rgba($tech-lightblue, 0.7);
  z-index: 3;

  .el-icon-data-analysis {
    font-size: 48px;
    color: rgba($tech-cyan, 0.3);
    margin-bottom: 12px;
    display: block;
  }

  span {
    display: block;
    font-size: 14px;
  }
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

:deep(.el-select-dropdown) {
  background: $tech-darkblue !important;
  border: 1px solid $tech-cyan !important;
  box-shadow: 0 0 15px rgba($tech-cyan, 0.3) !important;

  .el-select-dropdown__item {
    color: $tech-text;

    &:hover {
      background: rgba($tech-cyan, 0.1) !important;
    }

    &.selected {
      color: $tech-cyan;
      font-weight: normal;
    }
  }
}
:deep(.el-card .el-card__body) {
  padding: 0 !important;
}
</style>