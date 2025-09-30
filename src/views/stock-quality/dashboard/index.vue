<script setup lang="ts">
import { ref } from 'vue'
import AnomalyTrendChart from './components/AnomalyTrendChart.vue'
import DataSourceStatus from './components/DataSourceStatus.vue'
import RecentAnomalies from './components/RecentAnomalies.vue'

// 数据概览统计
const stats = ref({
  totalStocks: 8567,
  totalRecords: 2458903,
  lastUpdate: new Date().toLocaleString('zh-CN'),
  systemStatus: 'normal', // normal | warning | error
})

// 异常统计
const anomalyStats = ref({
  total: 156,
  pending: 42,
  processing: 18,
  resolved: 96,
})

// 系统状态文案和颜色
const statusConfig: Record<string, { text: string, type: 'success' | 'warning' | 'error' }> = {
  normal: { text: '正常', type: 'success' },
  warning: { text: '警告', type: 'warning' },
  error: { text: '异常', type: 'error' },
}
</script>

<template>
  <div class="stock-quality-dashboard">
    <!-- 顶部统计卡片 -->
    <n-grid :x-gap="16" :y-gap="16" :cols="4" item-responsive responsive="screen">
      <!-- 股票总数 -->
      <n-gi span="4 m:2 l:1">
        <n-card :bordered="false">
          <n-statistic label="股票总数" tabular-nums>
            <n-number-animation
              :from="0"
              :to="stats.totalStocks"
              :duration="1000"
              show-separator
            />
            <template #suffix>
              <n-text depth="3" style="font-size: 18px">
                只
              </n-text>
            </template>
          </n-statistic>
          <template #footer>
            <n-flex align="center">
              <nova-icon icon="icon-park-outline:stock-market" :size="20" color="var(--primary-color)" />
              <n-text depth="3" :style="{ fontSize: '13px' }">
                US & HK
              </n-text>
            </n-flex>
          </template>
        </n-card>
      </n-gi>

      <!-- 数据记录总数 -->
      <n-gi span="4 m:2 l:1">
        <n-card :bordered="false">
          <n-statistic label="数据记录总数" tabular-nums>
            <n-number-animation
              :from="0"
              :to="stats.totalRecords"
              :duration="1000"
              show-separator
            />
            <template #suffix>
              <n-text depth="3" style="font-size: 18px">
                条
              </n-text>
            </template>
          </n-statistic>
          <template #footer>
            <n-flex align="center">
              <nova-icon icon="icon-park-outline:data" :size="20" color="var(--success-color)" />
              <n-text depth="3" :style="{ fontSize: '13px' }">
                累计数据
              </n-text>
            </n-flex>
          </template>
        </n-card>
      </n-gi>

      <!-- 系统状态 -->
      <n-gi span="4 m:2 l:1">
        <n-card :bordered="false">
          <n-statistic label="系统状态">
            <n-flex align="center" :size="12">
              <n-tag
                :type="statusConfig[stats.systemStatus].type"
                size="large"
                :bordered="false"
                round
              >
                {{ statusConfig[stats.systemStatus].text }}
              </n-tag>
            </n-flex>
          </n-statistic>
          <template #footer>
            <n-flex align="center">
              <nova-icon icon="icon-park-outline:check-one" :size="20" color="var(--success-color)" />
              <n-text depth="3" :style="{ fontSize: '13px' }">
                运行稳定
              </n-text>
            </n-flex>
          </template>
        </n-card>
      </n-gi>

      <!-- 最后更新时间 -->
      <n-gi span="4 m:2 l:1">
        <n-card :bordered="false">
          <n-statistic label="最后更新">
            <n-text style="font-size: 16px">
              {{ stats.lastUpdate }}
            </n-text>
          </n-statistic>
          <template #footer>
            <n-flex align="center">
              <nova-icon icon="icon-park-outline:time" :size="20" color="var(--info-color)" />
              <n-text depth="3" :style="{ fontSize: '13px' }">
                实时同步
              </n-text>
            </n-flex>
          </template>
        </n-card>
      </n-gi>
    </n-grid>

    <!-- 异常统计概览 -->
    <n-card title="异常统计概览" class="mt-4" :bordered="false">
      <n-grid :x-gap="16" :y-gap="16" :cols="4" item-responsive responsive="screen">
        <n-gi span="4 m:2 l:1">
          <n-flex vertical align="center" :size="8">
            <n-text depth="3">
              异常总数
            </n-text>
            <n-text strong style="font-size: 32px" type="default">
              {{ anomalyStats.total }}
            </n-text>
            <n-progress
              type="line"
              status="default"
              :percentage="100"
              :show-indicator="false"
              :height="4"
            />
          </n-flex>
        </n-gi>
        <n-gi span="4 m:2 l:1">
          <n-flex vertical align="center" :size="8">
            <n-text depth="3">
              待处理
            </n-text>
            <n-text strong style="font-size: 32px" type="warning">
              {{ anomalyStats.pending }}
            </n-text>
            <n-progress
              type="line"
              status="warning"
              :percentage="(anomalyStats.pending / anomalyStats.total) * 100"
              :show-indicator="false"
              :height="4"
            />
          </n-flex>
        </n-gi>
        <n-gi span="4 m:2 l:1">
          <n-flex vertical align="center" :size="8">
            <n-text depth="3">
              处理中
            </n-text>
            <n-text strong style="font-size: 32px" type="info">
              {{ anomalyStats.processing }}
            </n-text>
            <n-progress
              type="line"
              status="info"
              :percentage="(anomalyStats.processing / anomalyStats.total) * 100"
              :show-indicator="false"
              :height="4"
            />
          </n-flex>
        </n-gi>
        <n-gi span="4 m:2 l:1">
          <n-flex vertical align="center" :size="8">
            <n-text depth="3">
              已解决
            </n-text>
            <n-text strong style="font-size: 32px" type="success">
              {{ anomalyStats.resolved }}
            </n-text>
            <n-progress
              type="line"
              status="success"
              :percentage="(anomalyStats.resolved / anomalyStats.total) * 100"
              :show-indicator="false"
              :height="4"
            />
          </n-flex>
        </n-gi>
      </n-grid>
    </n-card>

    <!-- 主内容区 -->
    <n-grid :x-gap="16" :y-gap="16" :cols="3" item-responsive responsive="screen" class="mt-4">
      <!-- 左侧内容 -->
      <n-gi span="3 l:2">
        <n-space vertical :size="16">
          <!-- 异常趋势分析 -->
          <AnomalyTrendChart />

          <!-- 最近异常快览 -->
          <RecentAnomalies />
        </n-space>
      </n-gi>

      <!-- 右侧边栏 -->
      <n-gi span="3 l:1">
        <!-- 数据源状态监控 -->
        <DataSourceStatus />
      </n-gi>
    </n-grid>
  </div>
</template>

<style scoped>
.stock-quality-dashboard {
  padding: 16px;
}
</style>
