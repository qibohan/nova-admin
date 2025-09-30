<script setup lang="ts">
import { computed, ref, watch } from 'vue'
import type { FormInst } from 'naive-ui'

interface Rule {
  id: string
  name: string
  type: string
  description: string
  status: 'active' | 'paused'
}

interface Props {
  show: boolean
  rule: Rule | null
  isCreating: boolean
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

const formRef = ref<FormInst | null>(null)
const formData = ref({
  name: '',
  type: '',
  market: '',
  field: '',
  checkMethod: '',
  threshold: '',
  executeFrequency: '',
  executeTime: '',
  alertMethod: '',
  description: '',
})

const rules = {
  name: {
    required: true,
    message: '请输入规则名称',
    trigger: 'blur',
  },
  type: {
    required: true,
    message: '请选择规则类型',
    trigger: 'change',
  },
  market: {
    required: true,
    message: '请选择市场',
    trigger: 'change',
  },
}

const ruleTypeOptions = [
  { label: '三方对比', value: 'comparison' },
  { label: '自校验', value: 'self-check' },
  { label: '事件溯源', value: 'event-trace' },
  { label: '完整性检查', value: 'integrity' },
]

const marketOptions = [
  { label: '美股', value: 'US' },
  { label: '港股', value: 'HK' },
  { label: '全部', value: 'ALL' },
]

const fieldOptions = [
  { label: 'PE（市盈率）', value: 'PE' },
  { label: 'PB（市净率）', value: 'PB' },
  { label: 'PS（市销率）', value: 'PS' },
  { label: 'EPS（每股收益）', value: 'EPS' },
  { label: 'ROE（净资产收益率）', value: 'ROE' },
  { label: '价格', value: 'price' },
  { label: '股本', value: 'shares' },
]

const executeFrequencyOptions = [
  { label: '每天', value: 'daily' },
  { label: '每周', value: 'weekly' },
  { label: '事件触发', value: 'event' },
]

const alertMethodOptions = [
  { label: 'Slack 机器人', value: 'slack' },
  { label: '后台通知', value: 'admin' },
  { label: '多维表格', value: 'table' },
  { label: '邮件', value: 'email' },
]

watch(() => props.rule, (newRule) => {
  if (newRule && !props.isCreating) {
    formData.value = {
      name: newRule.name,
      type: newRule.type,
      market: 'US',
      field: 'PE',
      checkMethod: 'comparison',
      threshold: '5',
      executeFrequency: 'daily',
      executeTime: '09:00',
      alertMethod: 'slack',
      description: newRule.description,
    }
  }
  else if (props.isCreating) {
    formData.value = {
      name: '',
      type: '',
      market: '',
      field: '',
      checkMethod: '',
      threshold: '',
      executeFrequency: '',
      executeTime: '',
      alertMethod: '',
      description: '',
    }
  }
}, { immediate: true })

function handleSubmit() {
  formRef.value?.validate((errors) => {
    if (!errors) {
      window.$message?.success(props.isCreating ? '规则创建成功' : '规则更新成功')
      showModal.value = false
    }
  })
}
</script>

<template>
  <n-modal
    v-model:show="showModal"
    preset="card"
    :title="isCreating ? '创建规则' : '编辑规则'"
    style="width: 700px"
    :bordered="false"
  >
    <n-form
      ref="formRef"
      :model="formData"
      :rules="rules"
      label-placement="left"
      label-width="120px"
    >
      <n-form-item label="规则名称" path="name">
        <n-input v-model:value="formData.name" placeholder="请输入规则名称" />
      </n-form-item>

      <n-form-item label="规则类型" path="type">
        <n-select
          v-model:value="formData.type"
          placeholder="请选择规则类型"
          :options="ruleTypeOptions"
        />
      </n-form-item>

      <n-form-item label="适用市场" path="market">
        <n-select
          v-model:value="formData.market"
          placeholder="请选择市场"
          :options="marketOptions"
        />
      </n-form-item>

      <n-form-item label="检查字段" path="field">
        <n-select
          v-model:value="formData.field"
          placeholder="请选择检查字段"
          :options="fieldOptions"
          multiple
        />
      </n-form-item>

      <n-form-item label="阈值">
        <n-input-group>
          <n-input v-model:value="formData.threshold" placeholder="请输入阈值" />
          <n-input-group-label>%</n-input-group-label>
        </n-input-group>
      </n-form-item>

      <n-form-item label="执行频率">
        <n-select
          v-model:value="formData.executeFrequency"
          placeholder="请选择执行频率"
          :options="executeFrequencyOptions"
        />
      </n-form-item>

      <n-form-item label="执行时间">
        <n-time-picker
          v-model:formatted-value="formData.executeTime"
          value-format="HH:mm"
          format="HH:mm"
          placeholder="请选择执行时间"
        />
      </n-form-item>

      <n-form-item label="预警方式">
        <n-select
          v-model:value="formData.alertMethod"
          placeholder="请选择预警方式"
          :options="alertMethodOptions"
          multiple
        />
      </n-form-item>

      <n-form-item label="规则描述">
        <n-input
          v-model:value="formData.description"
          type="textarea"
          placeholder="请输入规则描述"
          :rows="3"
        />
      </n-form-item>
    </n-form>

    <template #footer>
      <n-space justify="end">
        <n-button @click="showModal = false">
          取消
        </n-button>
        <n-button type="primary" @click="handleSubmit">
          {{ isCreating ? '创建' : '保存' }}
        </n-button>
      </n-space>
    </template>
  </n-modal>
</template>

<style scoped></style>
