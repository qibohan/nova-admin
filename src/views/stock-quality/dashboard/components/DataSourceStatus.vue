<script setup lang="ts">
import { ref } from 'vue'

interface DataSource {
  name: string
  status: 'online' | 'offline' | 'warning'
  reliability: number
  lastSync: string
  icon: string
}

const dataSources = ref<DataSource[]>([
  {
    name: 'Bloomberg',
    status: 'online',
    reliability: 98.5,
    lastSync: '2分钟前',
    icon: 'icon-park-outline:blockchain',
  },
  {
    name: 'Reuters',
    status: 'online',
    reliability: 97.2,
    lastSync: '5分钟前',
    icon: 'icon-park-outline:data-server',
  },
  {
    name: 'Yahoo Finance',
    status: 'warning',
    reliability: 89.6,
    lastSync: '15分钟前',
    icon: 'icon-park-outline:chart-stock',
  },
  {
    name: 'FactSet',
    status: 'online',
    reliability: 96.8,
    lastSync: '3分钟前',
    icon: 'icon-park-outline:data-all',
  },
  {
    name: 'ICE Data',
    status: 'online',
    reliability: 99.1,
    lastSync: '1分钟前',
    icon: 'icon-park-outline:data-switching',
  },
])

const statusConfig = {
  online: { text: '在线', type: 'success' as const },
  offline: { text: '离线', type: 'error' as const },
  warning: { text: '警告', type: 'warning' as const },
}

function getReliabilityType(reliability: number) {
  if (reliability >= 95)
    return 'success'
  if (reliability >= 90)
    return 'warning'
  return 'error'
}
</script>

<template>
  <n-card title="数据源状态监控" :bordered="false">
    <template #header-extra>
      <n-tag :bordered="false" type="success" size="small">
        {{ dataSources.filter(d => d.status === 'online').length }}/{{ dataSources.length }} 在线
      </n-tag>
    </template>

    <n-space vertical :size="16">
      <n-card
        v-for="source in dataSources"
        :key="source.name"
        size="small"
        :bordered="false"
        embedded
      >
        <n-flex justify="space-between" align="center">
          <n-flex align="center" :size="12">
            <nova-icon :icon="source.icon" :size="24" />
            <n-text strong>
              {{ source.name }}
            </n-text>
          </n-flex>
          <n-tag
            :type="statusConfig[source.status].type"
            size="small"
            :bordered="false"
            round
          >
            {{ statusConfig[source.status].text }}
          </n-tag>
        </n-flex>

        <n-divider style="margin: 12px 0" />

        <n-space vertical :size="8">
          <!-- 可靠性 -->
          <n-flex justify="space-between" align="center">
            <n-text depth="3" style="font-size: 13px">
              可靠性
            </n-text>
            <n-flex align="center" :size="8">
              <n-progress
                type="line"
                :percentage="source.reliability"
                :height="6"
                :border-radius="4"
                :status="getReliabilityType(source.reliability)"
                style="width: 100px"
              />
              <n-text :type="getReliabilityType(source.reliability)" style="font-size: 13px">
                {{ source.reliability }}%
              </n-text>
            </n-flex>
          </n-flex>

          <!-- 最后同步 -->
          <n-flex justify="space-between" align="center">
            <n-text depth="3" style="font-size: 13px">
              最后同步
            </n-text>
            <n-text style="font-size: 13px">
              {{ source.lastSync }}
            </n-text>
          </n-flex>
        </n-space>
      </n-card>
    </n-space>
  </n-card>
</template>

<style scoped></style>
