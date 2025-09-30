<script setup lang="ts">
import { computed, ref } from 'vue'
import type { DataTableColumns } from 'naive-ui'
import { NButton, NSpace, NTag } from 'naive-ui'
import AnomalyDetailModal from './components/AnomalyDetailModal.vue'

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

// 异常统计
const stats = ref({
  total: 156,
  pending: 42,
  processing: 18,
  resolved: 96,
})

// 筛选条件
const filters = ref({
  type: null as string | null,
  severity: null as string | null,
  status: null as string | null,
  stockCode: '',
  market: null as string | null,
})

// 异常类型选项
const typeOptions = [
  { label: '价格异常', value: 'price' },
  { label: '数据缺失', value: 'missing' },
  { label: '财务异常', value: 'financial' },
  { label: '多源差异', value: 'comparison' },
  { label: '规则违反', value: 'rule' },
  { label: '事件同步', value: 'event' },
]

// 严重程度选项
const severityOptions = [
  { label: '极高', value: 'critical' },
  { label: '高', value: 'high' },
  { label: '中', value: 'medium' },
  { label: '低', value: 'low' },
]

// 状态选项
const statusOptions = [
  { label: '待处理', value: 'pending' },
  { label: '处理中', value: 'processing' },
  { label: '已解决', value: 'resolved' },
]

// 市场选项
const marketOptions = [
  { label: '美股', value: 'US' },
  { label: '港股', value: 'HK' },
]

// 模拟数据
const anomalies = ref<Anomaly[]>([
  {
    id: '1',
    stockCode: 'AAPL',
    stockName: '苹果',
    market: 'US',
    type: '估值指标异常',
    severity: 'critical',
    status: 'pending',
    description: 'PE指标与竞品差异超过5%',
    discoveredAt: '2024-01-20 10:30:00',
  },
  {
    id: '2',
    stockCode: 'TSLA',
    stockName: '特斯拉',
    market: 'US',
    type: '价格异常',
    severity: 'high',
    status: 'processing',
    description: '价格波动超过100%',
    discoveredAt: '2024-01-20 10:25:00',
    handler: '张三',
  },
  {
    id: '3',
    stockCode: '00700.HK',
    stockName: '腾讯控股',
    market: 'HK',
    type: '数据缺失',
    severity: 'medium',
    status: 'pending',
    description: '股票简称字段为空',
    discoveredAt: '2024-01-20 10:20:00',
  },
  {
    id: '4',
    stockCode: 'GOOGL',
    stockName: '谷歌',
    market: 'US',
    type: '财务指标异常',
    severity: 'high',
    status: 'resolved',
    description: 'EPS计算口径差异超过2%',
    discoveredAt: '2024-01-20 10:15:00',
    handler: '李四',
  },
  {
    id: '5',
    stockCode: 'MSFT',
    stockName: '微软',
    market: 'US',
    type: '多源差异',
    severity: 'medium',
    status: 'pending',
    description: '昨收数据与竞品不一致',
    discoveredAt: '2024-01-20 10:10:00',
  },
])

// 选中的行
const checkedRowKeys = ref<string[]>([])

// 详情弹窗
const showDetailModal = ref(false)
const currentAnomaly = ref<Anomaly | null>(null)

// 严重程度配置
const severityConfig = {
  critical: { text: '极高', type: 'error' as const },
  high: { text: '高', type: 'error' as const },
  medium: { text: '中', type: 'warning' as const },
  low: { text: '低', type: 'success' as const },
}

// 状态配置
const statusConfig = {
  pending: { text: '待处理', type: 'warning' as const },
  processing: { text: '处理中', type: 'info' as const },
  resolved: { text: '已解决', type: 'success' as const },
}

// 表格列配置
const columns: DataTableColumns<Anomaly> = [
  {
    type: 'selection',
  },
  {
    title: '股票代码',
    key: 'stockCode',
    width: 120,
    render: (row) => {
      return h(NSpace, { size: 4, align: 'center' }, {
        default: () => [
          h('span', { style: 'font-weight: 600' }, row.stockCode),
          h(NTag, { size: 'tiny', bordered: false }, { default: () => row.market }),
        ],
      })
    },
  },
  {
    title: '股票名称',
    key: 'stockName',
    width: 120,
  },
  {
    title: '异常类型',
    key: 'type',
    width: 140,
  },
  {
    title: '严重程度',
    key: 'severity',
    width: 100,
    render: (row) => {
      return h(
        NTag,
        {
          type: severityConfig[row.severity].type,
          bordered: false,
          round: true,
          size: 'small',
        },
        { default: () => severityConfig[row.severity].text },
      )
    },
  },
  {
    title: '状态',
    key: 'status',
    width: 100,
    render: (row) => {
      return h(
        NTag,
        {
          type: statusConfig[row.status].type,
          bordered: false,
          round: true,
          size: 'small',
        },
        { default: () => statusConfig[row.status].text },
      )
    },
  },
  {
    title: '异常描述',
    key: 'description',
    ellipsis: {
      tooltip: true,
    },
  },
  {
    title: '发现时间',
    key: 'discoveredAt',
    width: 180,
  },
  {
    title: '处理人',
    key: 'handler',
    width: 100,
    render: (row) => {
      return row.handler || '-'
    },
  },
  {
    title: '操作',
    key: 'actions',
    width: 200,
    fixed: 'right',
    render: (row) => {
      return h(NSpace, null, {
        default: () => [
          h(
            NButton,
            {
              size: 'small',
              onClick: () => handleViewDetail(row),
            },
            { default: () => '查看' },
          ),
          h(
            NButton,
            {
              size: 'small',
              type: 'primary',
              disabled: row.status !== 'pending',
              onClick: () => handleProcess(row),
            },
            { default: () => '处理' },
          ),
          h(
            NButton,
            {
              size: 'small',
              type: 'success',
              disabled: row.status === 'resolved',
              onClick: () => handleResolve(row),
            },
            { default: () => '解决' },
          ),
        ],
      })
    },
  },
]

// 过滤后的数据
const filteredAnomalies = computed(() => {
  return anomalies.value.filter((anomaly) => {
    if (filters.value.type && anomaly.type !== filters.value.type)
      return false
    if (filters.value.severity && anomaly.severity !== filters.value.severity)
      return false
    if (filters.value.status && anomaly.status !== filters.value.status)
      return false
    if (filters.value.market && anomaly.market !== filters.value.market)
      return false
    if (filters.value.stockCode && !anomaly.stockCode.includes(filters.value.stockCode))
      return false
    return true
  })
})

// 分页
const pagination = ref({
  page: 1,
  pageSize: 10,
  showSizePicker: true,
  pageSizes: [10, 20, 50, 100],
  onChange: (page: number) => {
    pagination.value.page = page
  },
  onUpdatePageSize: (pageSize: number) => {
    pagination.value.pageSize = pageSize
    pagination.value.page = 1
  },
})

// 查看详情
function handleViewDetail(anomaly: Anomaly) {
  currentAnomaly.value = anomaly
  showDetailModal.value = true
}

// 处理异常
function handleProcess(anomaly: Anomaly) {
  window.$message?.info(`开始处理: ${anomaly.stockCode}`)
  anomaly.status = 'processing'
  anomaly.handler = '当前用户'
}

// 解决异常
function handleResolve(anomaly: Anomaly) {
  window.$message?.success(`已解决: ${anomaly.stockCode}`)
  anomaly.status = 'resolved'
}

// 批量处理
function handleBatchProcess() {
  if (checkedRowKeys.value.length === 0) {
    window.$message?.warning('请选择要处理的异常')
    return
  }
  window.$message?.info(`批量处理 ${checkedRowKeys.value.length} 条异常`)
}

// 批量解决
function handleBatchResolve() {
  if (checkedRowKeys.value.length === 0) {
    window.$message?.warning('请选择要解决的异常')
    return
  }
  window.$message?.success(`批量解决 ${checkedRowKeys.value.length} 条异常`)
}

// 重置筛选
function handleResetFilters() {
  filters.value = {
    type: null,
    severity: null,
    status: null,
    stockCode: '',
    market: null,
  }
}
</script>

<template>
  <div class="exceptions-page">
    <!-- 统计概览 -->
    <n-card :bordered="false" class="mb-4">
      <n-grid :x-gap="16" :y-gap="16" :cols="4" item-responsive responsive="screen">
        <n-gi span="4 m:2 l:1">
          <n-statistic label="总异常数" tabular-nums>
            <n-number-animation :from="0" :to="stats.total" :duration="1000" show-separator />
          </n-statistic>
        </n-gi>
        <n-gi span="4 m:2 l:1">
          <n-statistic label="待处理" tabular-nums>
            <n-number-animation :from="0" :to="stats.pending" :duration="1000" show-separator />
            <template #suffix>
              <NTag type="warning" size="small" :bordered="false" round>
                需关注
              </NTag>
            </template>
          </n-statistic>
        </n-gi>
        <n-gi span="4 m:2 l:1">
          <n-statistic label="处理中" tabular-nums>
            <n-number-animation :from="0" :to="stats.processing" :duration="1000" show-separator />
          </n-statistic>
        </n-gi>
        <n-gi span="4 m:2 l:1">
          <n-statistic label="已解决" tabular-nums>
            <n-number-animation :from="0" :to="stats.resolved" :duration="1000" show-separator />
            <template #suffix>
              <n-text type="success" style="font-size: 14px">
                {{ ((stats.resolved / stats.total) * 100).toFixed(1) }}%
              </n-text>
            </template>
          </n-statistic>
        </n-gi>
      </n-grid>
    </n-card>

    <!-- 筛选区域 -->
    <n-card :bordered="false" class="mb-4">
      <NSpace vertical :size="16">
        <n-grid :x-gap="16" :y-gap="16" :cols="4" item-responsive responsive="screen">
          <n-gi span="4 m:2 l:1">
            <n-select
              v-model:value="filters.type"
              placeholder="异常类型"
              :options="typeOptions"
              clearable
            />
          </n-gi>
          <n-gi span="4 m:2 l:1">
            <n-select
              v-model:value="filters.severity"
              placeholder="严重程度"
              :options="severityOptions"
              clearable
            />
          </n-gi>
          <n-gi span="4 m:2 l:1">
            <n-select
              v-model:value="filters.status"
              placeholder="处理状态"
              :options="statusOptions"
              clearable
            />
          </n-gi>
          <n-gi span="4 m:2 l:1">
            <n-select
              v-model:value="filters.market"
              placeholder="市场"
              :options="marketOptions"
              clearable
            />
          </n-gi>
        </n-grid>

        <NSpace align="center">
          <n-input
            v-model:value="filters.stockCode"
            placeholder="搜索股票代码"
            clearable
            style="width: 200px"
          >
            <template #prefix>
              <nova-icon icon="icon-park-outline:search" />
            </template>
          </n-input>

          <NButton @click="handleResetFilters">
            <template #icon>
              <nova-icon icon="icon-park-outline:reload" />
            </template>
            重置
          </NButton>

          <n-divider vertical />

          <NButton type="primary" :disabled="checkedRowKeys.length === 0" @click="handleBatchProcess">
            <template #icon>
              <nova-icon icon="icon-park-outline:play" />
            </template>
            批量处理
          </NButton>

          <NButton type="success" :disabled="checkedRowKeys.length === 0" @click="handleBatchResolve">
            <template #icon>
              <nova-icon icon="icon-park-outline:check-one" />
            </template>
            批量解决
          </NButton>
        </NSpace>
      </NSpace>
    </n-card>

    <!-- 异常列表 -->
    <n-card :bordered="false">
      <n-data-table
        v-model:checked-row-keys="checkedRowKeys"
        :columns="columns"
        :data="filteredAnomalies"
        :pagination="pagination"
        :row-key="(row: Anomaly) => row.id"
        :scroll-x="1400"
        striped
      />
    </n-card>

    <!-- 详情弹窗 -->
    <AnomalyDetailModal
      v-model:show="showDetailModal"
      :anomaly="currentAnomaly"
    />
  </div>
</template>

<style scoped>
.exceptions-page {
  padding: 16px;
}
</style>
