<script setup lang="ts">
import { ref } from 'vue'
import type { DataTableColumns } from 'naive-ui'
import { NButton, NProgress, NSpace, NSwitch, NTag } from 'naive-ui'
import RuleConfigModal from './components/RuleConfigModal.vue'

interface Rule {
  id: string
  name: string
  type: string
  description: string
  status: 'active' | 'paused'
  executeCount: number
  triggerCount: number
  accuracy: number
  lastExecute: string
}

// 规则统计
const stats = ref({
  total: 24,
  active: 18,
  paused: 6,
  avgAccuracy: 96.2,
})

// 模拟规则数据
const rules = ref<Rule[]>([
  {
    id: '1',
    name: '估值指标与竞品差异检查',
    type: '三方对比',
    description: '与4家三方数据源对比时，与至少1家的差值小于5%',
    status: 'active',
    executeCount: 1250,
    triggerCount: 48,
    accuracy: 98.5,
    lastExecute: '2分钟前',
  },
  {
    id: '2',
    name: '估值指标历史数据对比',
    type: '自校验',
    description: '估值指标与自身前一交易日数据差值小于100%',
    status: 'active',
    executeCount: 1250,
    triggerCount: 12,
    accuracy: 99.2,
    lastExecute: '2分钟前',
  },
  {
    id: '3',
    name: '价格为零或负数检查',
    type: '自校验',
    description: '交易日价格大于0',
    status: 'active',
    executeCount: 1250,
    triggerCount: 3,
    accuracy: 99.8,
    lastExecute: '2分钟前',
  },
  {
    id: '4',
    name: 'EPS计算口径检查',
    type: '自校验',
    description: '自算EPS（净利润/总股本）与标普EPS差异小于2%',
    status: 'active',
    executeCount: 856,
    triggerCount: 15,
    accuracy: 97.5,
    lastExecute: '30分钟前',
  },
  {
    id: '5',
    name: 'ROE计算钩稽关系检查',
    type: '自校验',
    description: '自算ROE（净利润/净资产）与标普ROE差异小于3%',
    status: 'paused',
    executeCount: 856,
    triggerCount: 18,
    accuracy: 96.8,
    lastExecute: '1小时前',
  },
  {
    id: '6',
    name: '股票简称完整性检查',
    type: '完整性',
    description: '必填字段有值',
    status: 'active',
    executeCount: 8567,
    triggerCount: 5,
    accuracy: 99.9,
    lastExecute: '2分钟前',
  },
  {
    id: '7',
    name: 'Ticker变更后字段同步检查',
    type: '事件溯源',
    description: '依赖ICE的Ticker变更事件，相关字段及时同步变更',
    status: 'active',
    executeCount: 42,
    triggerCount: 2,
    accuracy: 95.2,
    lastExecute: '1天前',
  },
  {
    id: '8',
    name: '财报发布后TTM更新检查',
    type: '事件溯源',
    description: '财报发布2个工作日后TTM数据更新到最新期',
    status: 'active',
    executeCount: 128,
    triggerCount: 8,
    accuracy: 93.8,
    lastExecute: '3小时前',
  },
])

// 配置弹窗
const showConfigModal = ref(false)
const currentRule = ref<Rule | null>(null)
const isCreating = ref(false)

// 表格列配置
const columns: DataTableColumns<Rule> = [
  {
    title: '规则名称',
    key: 'name',
    width: 200,
    ellipsis: {
      tooltip: true,
    },
  },
  {
    title: '规则类型',
    key: 'type',
    width: 120,
    render: (row) => {
      const typeMap: Record<string, string> = {
        三方对比: 'primary',
        自校验: 'info',
        完整性: 'success',
        事件溯源: 'warning',
      }
      return h(
        NTag,
        {
          type: typeMap[row.type] as any || 'default',
          size: 'small',
          bordered: false,
        },
        { default: () => row.type },
      )
    },
  },
  {
    title: '规则描述',
    key: 'description',
    ellipsis: {
      tooltip: true,
    },
  },
  {
    title: '状态',
    key: 'status',
    width: 100,
    render: (row) => {
      return h(
        NSwitch,
        {
          value: row.status === 'active',
          onUpdateValue: (value: boolean) => handleToggleStatus(row, value),
        },
      )
    },
  },
  {
    title: '执行次数',
    key: 'executeCount',
    width: 120,
  },
  {
    title: '触发次数',
    key: 'triggerCount',
    width: 120,
  },
  {
    title: '准确率',
    key: 'accuracy',
    width: 180,
    render: (row) => {
      return h(
        NSpace,
        { size: 8, align: 'center' },
        {
          default: () => [
            h(NProgress, {
              type: 'line',
              percentage: row.accuracy,
              status: row.accuracy >= 95 ? 'success' : row.accuracy >= 90 ? 'warning' : 'error',
              height: 8,
              borderRadius: 4,
              showIndicator: false,
              style: { width: '80px' },
            }),
            h('span', { style: 'font-size: 13px; white-space: nowrap' }, `${row.accuracy}%`),
          ],
        },
      )
    },
  },
  {
    title: '最后执行',
    key: 'lastExecute',
    width: 120,
  },
  {
    title: '操作',
    key: 'actions',
    width: 240,
    fixed: 'right',
    render: (row) => {
      return h(NSpace, null, {
        default: () => [
          h(
            NButton,
            {
              size: 'small',
              onClick: () => handleEdit(row),
            },
            { default: () => '编辑' },
          ),
          h(
            NButton,
            {
              size: 'small',
              type: 'info',
              onClick: () => handleTest(row),
            },
            { default: () => '测试' },
          ),
          h(
            NButton,
            {
              size: 'small',
              type: 'error',
              onClick: () => handleDelete(row),
            },
            { default: () => '删除' },
          ),
        ],
      })
    },
  },
]

// 切换规则状态
function handleToggleStatus(rule: Rule, value: boolean) {
  rule.status = value ? 'active' : 'paused'
  window.$message?.success(`规则已${value ? '启用' : '暂停'}`)
}

// 创建规则
function handleCreate() {
  isCreating.value = true
  currentRule.value = null
  showConfigModal.value = true
}

// 编辑规则
function handleEdit(rule: Rule) {
  isCreating.value = false
  currentRule.value = rule
  showConfigModal.value = true
}

// 测试规则
function handleTest(rule: Rule) {
  window.$message?.info(`开始测试规则: ${rule.name}`)
}

// 删除规则
function handleDelete(rule: Rule) {
  window.$dialog?.warning({
    title: '删除规则',
    content: `确定要删除规则 "${rule.name}" 吗？`,
    positiveText: '确定',
    negativeText: '取消',
    onPositiveClick: () => {
      const index = rules.value.findIndex(r => r.id === rule.id)
      if (index !== -1) {
        rules.value.splice(index, 1)
        window.$message?.success('规则已删除')
      }
    },
  })
}

// 分页
const pagination = ref({
  page: 1,
  pageSize: 10,
  showSizePicker: true,
  pageSizes: [10, 20, 50],
})
</script>

<template>
  <div class="rules-page">
    <!-- 统计概览 -->
    <n-card :bordered="false" class="mb-4">
      <n-grid :x-gap="16" :y-gap="16" :cols="4" item-responsive responsive="screen">
        <n-gi span="4 m:2 l:1">
          <n-statistic label="规则总数" tabular-nums>
            <n-number-animation :from="0" :to="stats.total" :duration="1000" />
          </n-statistic>
        </n-gi>
        <n-gi span="4 m:2 l:1">
          <n-statistic label="运行中规则" tabular-nums>
            <n-number-animation :from="0" :to="stats.active" :duration="1000" />
            <template #suffix>
              <NTag type="success" size="small" :bordered="false" round>
                活跃
              </NTag>
            </template>
          </n-statistic>
        </n-gi>
        <n-gi span="4 m:2 l:1">
          <n-statistic label="已暂停规则" tabular-nums>
            <n-number-animation :from="0" :to="stats.paused" :duration="1000" />
          </n-statistic>
        </n-gi>
        <n-gi span="4 m:2 l:1">
          <n-statistic label="平均准确率" tabular-nums>
            <n-number-animation :from="0" :to="stats.avgAccuracy" :duration="1000" :precision="1" />
            <template #suffix>
              <n-text style="font-size: 18px">
                %
              </n-text>
            </template>
          </n-statistic>
        </n-gi>
      </n-grid>
    </n-card>

    <!-- 操作栏 -->
    <n-card :bordered="false" class="mb-4">
      <NSpace>
        <NButton type="primary" @click="handleCreate">
          <template #icon>
            <nova-icon icon="icon-park-outline:plus" />
          </template>
          创建规则
        </NButton>

        <NButton>
          <template #icon>
            <nova-icon icon="icon-park-outline:import" />
          </template>
          导入规则
        </NButton>

        <NButton>
          <template #icon>
            <nova-icon icon="icon-park-outline:export" />
          </template>
          导出规则
        </NButton>
      </NSpace>
    </n-card>

    <!-- 规则列表 -->
    <n-card :bordered="false">
      <n-data-table
        :columns="columns"
        :data="rules"
        :pagination="pagination"
        :scroll-x="1600"
        striped
      />
    </n-card>

    <!-- 规则配置弹窗 -->
    <RuleConfigModal
      v-model:show="showConfigModal"
      :rule="currentRule"
      :is-creating="isCreating"
    />
  </div>
</template>

<style scoped>
.rules-page {
  padding: 16px;
}
</style>
