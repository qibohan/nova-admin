<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { useEcharts } from '@/hooks'

const chartRef = ref<HTMLElement>()
const { update } = useEcharts(chartRef as Ref<HTMLElement>)

// 模拟近7天异常数据
const chartData = ref({
  dates: ['01-20', '01-21', '01-22', '01-23', '01-24', '01-25', '01-26'],
  total: [45, 52, 38, 48, 56, 42, 39],
  highSeverity: [12, 15, 10, 14, 18, 12, 11],
  mediumSeverity: [18, 22, 16, 20, 24, 18, 16],
  lowSeverity: [15, 15, 12, 14, 14, 12, 12],
})

onMounted(() => {
  const option = {
    title: {
      text: '近7天异常趋势',
      left: 'center',
      textStyle: {
        fontSize: 16,
        fontWeight: 'normal',
      },
    },
    tooltip: {
      trigger: 'axis',
      axisPointer: {
        type: 'cross',
      },
    },
    legend: {
      data: ['总异常', '极高/高', '中', '低'],
      bottom: 10,
    },
    grid: {
      left: '3%',
      right: '4%',
      bottom: '15%',
      top: '15%',
      containLabel: true,
    },
    xAxis: {
      type: 'category',
      boundaryGap: false,
      data: chartData.value.dates,
    },
    yAxis: {
      type: 'value',
      name: '异常数量',
    },
    series: [
      {
        name: '总异常',
        type: 'line',
        data: chartData.value.total,
        smooth: true,
        lineStyle: {
          width: 3,
        },
        emphasis: {
          focus: 'series',
        },
      },
      {
        name: '极高/高',
        type: 'line',
        data: chartData.value.highSeverity,
        smooth: true,
        itemStyle: {
          color: '#f5222d',
        },
        areaStyle: {
          opacity: 0.3,
        },
      },
      {
        name: '中',
        type: 'line',
        data: chartData.value.mediumSeverity,
        smooth: true,
        itemStyle: {
          color: '#faad14',
        },
        areaStyle: {
          opacity: 0.3,
        },
      },
      {
        name: '低',
        type: 'line',
        data: chartData.value.lowSeverity,
        smooth: true,
        itemStyle: {
          color: '#52c41a',
        },
        areaStyle: {
          opacity: 0.3,
        },
      },
    ],
  }
  update(option)
})
</script>

<template>
  <n-card title="异常趋势分析" :bordered="false">
    <template #header-extra>
      <n-space>
        <n-tag type="error" size="small" :bordered="false">
          极高/高
        </n-tag>
        <n-tag type="warning" size="small" :bordered="false">
          中
        </n-tag>
        <n-tag type="success" size="small" :bordered="false">
          低
        </n-tag>
      </n-space>
    </template>
    <div ref="chartRef" style="width: 100%; height: 400px" />
  </n-card>
</template>

<style scoped></style>
