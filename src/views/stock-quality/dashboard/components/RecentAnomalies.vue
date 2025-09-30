<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'

interface Anomaly {
  id: string
  stockCode: string
  stockName: string
  type: string
  severity: 'critical' | 'high' | 'medium' | 'low'
  description: string
  time: string
}

const router = useRouter()

const anomalies = ref<Anomaly[]>([
  {
    id: '1',
    stockCode: 'AAPL',
    stockName: '苹果',
    type: '估值指标异常',
    severity: 'critical',
    description: 'PE指标与竞品差异超过5%',
    time: '5分钟前',
  },
  {
    id: '2',
    stockCode: 'TSLA',
    stockName: '特斯拉',
    type: '价格异常',
    severity: 'high',
    description: '价格波动超过100%',
    time: '10分钟前',
  },
  {
    id: '3',
    stockCode: '00700.HK',
    stockName: '腾讯控股',
    type: '数据缺失',
    severity: 'medium',
    description: '股票简称字段为空',
    time: '15分钟前',
  },
  {
    id: '4',
    stockCode: 'GOOGL',
    stockName: '谷歌',
    type: '财务指标异常',
    severity: 'high',
    description: 'EPS计算口径差异超过2%',
    time: '20分钟前',
  },
  {
    id: '5',
    stockCode: 'MSFT',
    stockName: '微软',
    type: '多源差异',
    severity: 'medium',
    description: '昨收数据与竞品不一致',
    time: '25分钟前',
  },
  {
    id: '6',
    stockCode: '01810.HK',
    stockName: '小米集团',
    type: '事件同步滞后',
    severity: 'low',
    description: '财报发布后TTM数据未及时更新',
    time: '30分钟前',
  },
])

const severityConfig = {
  critical: { text: '极高', type: 'error' as const, color: '#f5222d' },
  high: { text: '高', type: 'error' as const, color: '#ff4d4f' },
  medium: { text: '中', type: 'warning' as const, color: '#faad14' },
  low: { text: '低', type: 'success' as const, color: '#52c41a' },
}

function handleViewDetail(anomaly: Anomaly) {
  router.push({
    path: '/stock-quality/exceptions',
    query: { id: anomaly.id },
  })
}
</script>

<template>
  <n-card title="最近异常快览" :bordered="false">
    <template #header-extra>
      <n-button
        type="primary"
        text
        @click="router.push('/stock-quality/exceptions')"
      >
        查看全部
        <template #icon>
          <nova-icon icon="icon-park-outline:right" />
        </template>
      </n-button>
    </template>

    <n-list hoverable clickable>
      <n-list-item
        v-for="anomaly in anomalies"
        :key="anomaly.id"
        @click="handleViewDetail(anomaly)"
      >
        <template #prefix>
          <n-tag
            :type="severityConfig[anomaly.severity].type"
            :bordered="false"
            size="small"
            round
          >
            {{ severityConfig[anomaly.severity].text }}
          </n-tag>
        </template>

        <n-thing>
          <template #header>
            <n-flex align="center" :size="8">
              <n-text strong>
                {{ anomaly.stockCode }}
              </n-text>
              <n-text depth="3">
                {{ anomaly.stockName }}
              </n-text>
              <n-tag size="tiny" :bordered="false">
                {{ anomaly.type }}
              </n-tag>
            </n-flex>
          </template>
          <template #description>
            <n-text depth="3" style="font-size: 13px">
              {{ anomaly.description }}
            </n-text>
          </template>
          <template #footer>
            <n-flex align="center" :size="4">
              <nova-icon icon="icon-park-outline:time" :size="14" />
              <n-text depth="3" style="font-size: 12px">
                {{ anomaly.time }}
              </n-text>
            </n-flex>
          </template>
        </n-thing>

        <template #suffix>
          <nova-icon icon="icon-park-outline:right" :size="16" />
        </template>
      </n-list-item>
    </n-list>
  </n-card>
</template>

<style scoped></style>
