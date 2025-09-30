<script setup lang="ts">
import { ref } from 'vue'
import type { DataTableColumns } from 'naive-ui'
import { NButton, NSpace, NTag } from 'naive-ui'

interface Event {
  id: string
  stockCode: string
  stockName: string
  eventType: string
  status: 'completed' | 'processing' | 'failed' | 'pending'
  impact: number
  triggerTime: string
  description: string
}

const stats = ref({
  total: 89,
  completed: 76,
  processing: 8,
  failed: 3,
  pending: 2,
})

const events = ref<Event[]>([
  {
    id: '1',
    stockCode: 'AAPL',
    stockName: '苹果',
    eventType: '财报发布',
    status: 'completed',
    impact: 125,
    triggerTime: '2024-01-20 10:00:00',
    description: 'Q4财报发布，TTM数据已更新',
  },
  {
    id: '2',
    stockCode: 'TSLA',
    stockName: '特斯拉',
    eventType: '股票拆分',
    status: 'processing',
    impact: 856,
    triggerTime: '2024-01-20 09:30:00',
    description: '1拆5股票拆分，历史价格调整中',
  },
  {
    id: '3',
    stockCode: '00700.HK',
    stockName: '腾讯控股',
    eventType: '分红派息',
    status: 'completed',
    impact: 45,
    triggerTime: '2024-01-20 09:00:00',
    description: '现金分红，复权数据已更新',
  },
  {
    id: '4',
    stockCode: 'GOOGL',
    stockName: '谷歌',
    eventType: '代码变更',
    status: 'failed',
    impact: 12,
    triggerTime: '2024-01-20 08:30:00',
    description: 'Ticker变更，相关字段同步失败',
  },
  {
    id: '5',
    stockCode: '01810.HK',
    stockName: '小米集团',
    eventType: '股本变更',
    status: 'completed',
    impact: 38,
    triggerTime: '2024-01-19 16:00:00',
    description: '增发新股，股本数据已更新',
  },
])

const statusConfig = {
  completed: { text: '已完成', type: 'success' as const },
  processing: { text: '处理中', type: 'info' as const },
  failed: { text: '失败', type: 'error' as const },
  pending: { text: '等待中', type: 'warning' as const },
}

const columns: DataTableColumns<Event> = [
  {
    title: '股票代码',
    key: 'stockCode',
    width: 120,
  },
  {
    title: '股票名称',
    key: 'stockName',
    width: 120,
  },
  {
    title: '事件类型',
    key: 'eventType',
    width: 120,
    render: (row) => {
      return h(NTag, { size: 'small', bordered: false }, { default: () => row.eventType })
    },
  },
  {
    title: '处理状态',
    key: 'status',
    width: 120,
    render: (row) => {
      return h(
        NTag,
        {
          type: statusConfig[row.status].type,
          bordered: false,
          round: true,
        },
        { default: () => statusConfig[row.status].text },
      )
    },
  },
  {
    title: '影响记录数',
    key: 'impact',
    width: 120,
  },
  {
    title: '触发时间',
    key: 'triggerTime',
    width: 180,
  },
  {
    title: '事件描述',
    key: 'description',
    ellipsis: {
      tooltip: true,
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
              onClick: () => window.$message?.info(`查看: ${row.stockCode}`),
            },
            { default: () => '查看详情' },
          ),
          h(
            NButton,
            {
              size: 'small',
              type: 'warning',
              disabled: row.status !== 'failed',
              onClick: () => window.$message?.info(`重试: ${row.stockCode}`),
            },
            { default: () => '重试' },
          ),
        ],
      })
    },
  },
]

const pagination = ref({
  page: 1,
  pageSize: 10,
  showSizePicker: true,
  pageSizes: [10, 20, 50],
})
</script>

<template>
  <div class="events-page">
    <!-- 统计概览 -->
    <n-card :bordered="false" class="mb-4">
      <n-grid :x-gap="16" :y-gap="16" :cols="5" item-responsive responsive="screen">
        <n-gi span="5 m:1">
          <n-statistic label="事件总数" tabular-nums>
            <n-number-animation :from="0" :to="stats.total" :duration="1000" />
          </n-statistic>
        </n-gi>
        <n-gi span="5 m:1">
          <n-statistic label="已完成" tabular-nums>
            <n-number-animation :from="0" :to="stats.completed" :duration="1000" />
          </n-statistic>
        </n-gi>
        <n-gi span="5 m:1">
          <n-statistic label="处理中" tabular-nums>
            <n-number-animation :from="0" :to="stats.processing" :duration="1000" />
          </n-statistic>
        </n-gi>
        <n-gi span="5 m:1">
          <n-statistic label="失败" tabular-nums>
            <n-number-animation :from="0" :to="stats.failed" :duration="1000" />
          </n-statistic>
        </n-gi>
        <n-gi span="5 m:1">
          <n-statistic label="等待中" tabular-nums>
            <n-number-animation :from="0" :to="stats.pending" :duration="1000" />
          </n-statistic>
        </n-gi>
      </n-grid>
    </n-card>

    <!-- 事件列表 -->
    <n-card :bordered="false" title="事件追溯">
      <n-data-table
        :columns="columns"
        :data="events"
        :pagination="pagination"
        :scroll-x="1400"
        striped
      />
    </n-card>
  </div>
</template>

<style scoped>
.events-page {
  padding: 16px;
}
</style>
