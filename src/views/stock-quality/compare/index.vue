<script setup lang="ts">
import { ref } from 'vue'
import type { DataTableColumns } from 'naive-ui'
import { NButton, NProgress, NSpace, NTag } from 'naive-ui'

interface Comparison {
  id: string
  stockCode: string
  stockName: string
  compareType: string
  dataSources: string[]
  consistency: number
  differenceCount: number
  compareTime: string
  isConsistent: boolean
}

const stats = ref({
  total: 245,
  inconsistent: 28,
  consistencyRate: 88.6,
  lastUpdate: new Date().toLocaleString('zh-CN'),
})

const comparisons = ref<Comparison[]>([
  {
    id: '1',
    stockCode: 'AAPL',
    stockName: '苹果',
    compareType: '价格数据',
    dataSources: ['Bloomberg', 'Reuters', 'Yahoo Finance'],
    consistency: 98.5,
    differenceCount: 1,
    compareTime: '2024-01-20 10:30:00',
    isConsistent: true,
  },
  {
    id: '2',
    stockCode: 'TSLA',
    stockName: '特斯拉',
    compareType: '财务数据',
    dataSources: ['Bloomberg', 'FactSet'],
    consistency: 85.2,
    differenceCount: 3,
    compareTime: '2024-01-20 10:25:00',
    isConsistent: false,
  },
  {
    id: '3',
    stockCode: '00700.HK',
    stockName: '腾讯控股',
    compareType: '基础信息',
    dataSources: ['Bloomberg', 'Reuters', 'HKEx'],
    consistency: 100,
    differenceCount: 0,
    compareTime: '2024-01-20 10:20:00',
    isConsistent: true,
  },
  {
    id: '4',
    stockCode: 'GOOGL',
    stockName: '谷歌',
    compareType: '价格数据',
    dataSources: ['Bloomberg', 'Reuters'],
    consistency: 92.3,
    differenceCount: 2,
    compareTime: '2024-01-20 10:15:00',
    isConsistent: false,
  },
])

const columns: DataTableColumns<Comparison> = [
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
    title: '对比类型',
    key: 'compareType',
    width: 120,
    render: (row) => {
      return h(NTag, { size: 'small', bordered: false }, { default: () => row.compareType })
    },
  },
  {
    title: '数据源',
    key: 'dataSources',
    width: 250,
    render: (row) => {
      return h(
        NSpace,
        { size: 4 },
        {
          default: () => row.dataSources.map(source =>
            h(NTag, { size: 'tiny', bordered: false }, { default: () => source }),
          ),
        },
      )
    },
  },
  {
    title: '一致性',
    key: 'consistency',
    width: 180,
    render: (row) => {
      return h(
        NSpace,
        { size: 8, align: 'center' },
        {
          default: () => [
            h(NProgress, {
              type: 'line',
              percentage: row.consistency,
              status: row.consistency >= 95 ? 'success' : row.consistency >= 85 ? 'warning' : 'error',
              height: 8,
              borderRadius: 4,
              showIndicator: false,
              style: { width: '80px' },
            }),
            h('span', { style: 'font-size: 13px' }, `${row.consistency}%`),
          ],
        },
      )
    },
  },
  {
    title: '差异字段数',
    key: 'differenceCount',
    width: 120,
  },
  {
    title: '对比时间',
    key: 'compareTime',
    width: 180,
  },
  {
    title: '操作',
    key: 'actions',
    width: 180,
    fixed: 'right',
    render: (row) => {
      return h(NSpace, null, {
        default: () => [
          h(
            NButton,
            {
              size: 'small',
              onClick: () => window.$message?.info(`查看差异: ${row.stockCode}`),
            },
            { default: () => '查看差异' },
          ),
          h(
            NButton,
            {
              size: 'small',
              type: 'info',
              onClick: () => window.$message?.info(`重新对比: ${row.stockCode}`),
            },
            { default: () => '重新对比' },
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

function handleCreateCompare() {
  window.$message?.info('发起新对比')
}
</script>

<template>
  <div class="compare-page">
    <!-- 统计概览 -->
    <n-card :bordered="false" class="mb-4">
      <n-grid :x-gap="16" :y-gap="16" :cols="4" item-responsive responsive="screen">
        <n-gi span="4 m:2 l:1">
          <n-statistic label="对比总数" tabular-nums>
            <n-number-animation :from="0" :to="stats.total" :duration="1000" />
          </n-statistic>
        </n-gi>
        <n-gi span="4 m:2 l:1">
          <n-statistic label="不一致记录" tabular-nums>
            <n-number-animation :from="0" :to="stats.inconsistent" :duration="1000" />
            <template #suffix>
              <NTag type="warning" size="small" :bordered="false" round>
                需关注
              </NTag>
            </template>
          </n-statistic>
        </n-gi>
        <n-gi span="4 m:2 l:1">
          <n-statistic label="一致性率" tabular-nums>
            <n-number-animation :from="0" :to="stats.consistencyRate" :duration="1000" :precision="1" />
            <template #suffix>
              <n-text style="font-size: 18px">
                %
              </n-text>
            </template>
          </n-statistic>
        </n-gi>
        <n-gi span="4 m:2 l:1">
          <n-statistic label="最后更新">
            <n-text style="font-size: 14px">
              {{ stats.lastUpdate }}
            </n-text>
          </n-statistic>
        </n-gi>
      </n-grid>
    </n-card>

    <!-- 操作栏 -->
    <n-card :bordered="false" class="mb-4">
      <NSpace>
        <NButton type="primary" @click="handleCreateCompare">
          <template #icon>
            <nova-icon icon="icon-park-outline:plus" />
          </template>
          发起新对比
        </NButton>
        <NButton>
          <template #icon>
            <nova-icon icon="icon-park-outline:setting" />
          </template>
          对比配置
        </NButton>
      </NSpace>
    </n-card>

    <!-- 对比结果列表 -->
    <n-card :bordered="false" title="对比结果">
      <n-data-table
        :columns="columns"
        :data="comparisons"
        :pagination="pagination"
        :scroll-x="1400"
        striped
      />
    </n-card>
  </div>
</template>

<style scoped>
.compare-page {
  padding: 16px;
}
</style>
