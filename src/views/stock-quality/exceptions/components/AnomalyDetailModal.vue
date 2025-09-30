<script setup lang="ts">
import { computed } from 'vue'

interface Anomaly {
  id: string
  stockCode: string
  stockName: string
  market: 'US' | 'HK'
  type: string
  severity: 'critical' | 'high' | 'medium' | 'low'
  status: 'pending' | 'processing' | 'resolved'
  description: string
  discoveredAt: string
  handler?: string
}

interface Props {
  show: boolean
  anomaly: Anomaly | null
}

interface Emits {
  (e: 'update:show', value: boolean): void
}

const props = defineProps<Props>()
const emit = defineEmits<Emits>()

const showModal = computed({
  get: () => props.show,
  set: value => emit('update:show', value),
})

// 模拟详细信息
const detailInfo = computed(() => {
  if (!props.anomaly)
    return null

  return {
    基础信息: [
      { label: '异常ID', value: props.anomaly.id },
      { label: '股票代码', value: props.anomaly.stockCode },
      { label: '股票名称', value: props.anomaly.stockName },
      { label: '市场', value: props.anomaly.market === 'US' ? '美股' : '港股' },
      { label: '异常类型', value: props.anomaly.type },
      { label: '发现时间', value: props.anomaly.discoveredAt },
    ],
    异常详情: [
      { label: '异常描述', value: props.anomaly.description },
      { label: '数据源', value: 'Bloomberg, Reuters' },
      { label: '影响字段', value: 'PE, PB, PS' },
      { label: '差异程度', value: '8.5%' },
    ],
    处理信息: [
      { label: '当前状态', value: props.anomaly.status },
      { label: '处理人', value: props.anomaly.handler || '未分配' },
      { label: '创建时间', value: props.anomaly.discoveredAt },
      { label: '更新时间', value: props.anomaly.discoveredAt },
    ],
  }
})

// 技术详情（JSON格式）
const technicalDetails = {
  anomaly_id: props.anomaly?.id,
  stock_info: {
    code: props.anomaly?.stockCode,
    name: props.anomaly?.stockName,
    market: props.anomaly?.market,
  },
  data_source: ['Bloomberg', 'Reuters'],
  affected_fields: ['PE', 'PB', 'PS'],
  expected_value: {
    PE: 25.5,
    PB: 4.2,
    PS: 6.8,
  },
  actual_value: {
    PE: 27.7,
    PB: 4.2,
    PS: 6.8,
  },
  difference: {
    PE: '8.5%',
    PB: '0%',
    PS: '0%',
  },
  rule_triggered: 'PE_COMPARISON_THRESHOLD',
  threshold: '5%',
}
</script>

<template>
  <n-modal
    v-model:show="showModal"
    preset="card"
    title="异常详情"
    style="width: 800px"
    :bordered="false"
    :segmented="{
      content: true,
      footer: 'soft',
    }"
  >
    <div v-if="anomaly">
      <n-tabs type="line" animated>
        <n-tab-pane name="basic" tab="基础信息">
          <n-descriptions
            label-placement="left"
            :column="2"
            bordered
          >
            <n-descriptions-item
              v-for="item in detailInfo?.基础信息"
              :key="item.label"
              :label="item.label"
            >
              {{ item.value }}
            </n-descriptions-item>
          </n-descriptions>

          <n-divider />

          <n-descriptions
            label-placement="left"
            :column="1"
            bordered
            title="异常详情"
          >
            <n-descriptions-item
              v-for="item in detailInfo?.异常详情"
              :key="item.label"
              :label="item.label"
            >
              {{ item.value }}
            </n-descriptions-item>
          </n-descriptions>

          <n-divider />

          <n-descriptions
            label-placement="left"
            :column="2"
            bordered
            title="处理信息"
          >
            <n-descriptions-item
              v-for="item in detailInfo?.处理信息"
              :key="item.label"
              :label="item.label"
            >
              {{ item.value }}
            </n-descriptions-item>
          </n-descriptions>
        </n-tab-pane>

        <n-tab-pane name="technical" tab="技术详情">
          <n-code
            :code="JSON.stringify(technicalDetails, null, 2)"
            language="json"
            :word-wrap="true"
          />
        </n-tab-pane>

        <n-tab-pane name="history" tab="处理历史">
          <n-timeline>
            <n-timeline-item
              type="success"
              title="异常发现"
              :time="anomaly.discoveredAt"
              content="系统自动检测发现异常"
            />
            <n-timeline-item
              v-if="anomaly.status === 'processing' || anomaly.status === 'resolved'"
              type="info"
              title="开始处理"
              :time="anomaly.discoveredAt"
              :content="`${anomaly.handler} 开始处理该异常`"
            />
            <n-timeline-item
              v-if="anomaly.status === 'resolved'"
              type="success"
              title="处理完成"
              :time="anomaly.discoveredAt"
              :content="`${anomaly.handler} 已解决该异常`"
            />
          </n-timeline>
        </n-tab-pane>
      </n-tabs>
    </div>

    <template #footer>
      <n-space justify="end">
        <n-button @click="showModal = false">
          关闭
        </n-button>
        <n-button
          v-if="anomaly && anomaly.status !== 'resolved'"
          type="primary"
        >
          开始处理
        </n-button>
        <n-button
          v-if="anomaly && anomaly.status === 'processing'"
          type="success"
        >
          标记为已解决
        </n-button>
      </n-space>
    </template>
  </n-modal>
</template>

<style scoped></style>
