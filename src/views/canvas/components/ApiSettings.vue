<template>
  <!-- API Settings Modal | API 设置弹窗 -->
  <n-modal v-model:show="showModal" preset="card" title="绘图提供商设置" style="width: 520px;">
    <n-form ref="formRef" :model="formData" label-placement="left" label-width="100">
      
      <n-form-item label="提供商" path="provider">
        <n-select 
          v-model:value="formData.provider" 
          :options="providerOptions"
          placeholder="选择绘图提供商"
          @update:value="handleProviderChange"
        />
      </n-form-item>

      <n-form-item label="Base URL" path="baseUrl">
        <n-input 
          v-model:value="formData.baseUrl" 
          placeholder="https://api.example.com"
          :disabled="!formData.provider"
        />
      </n-form-item>

      <n-form-item label="API Key" path="apiKey">
        <n-input 
          v-model:value="formData.apiKey" 
          type="password"
          show-password-on="click"
          placeholder="请输入 API Key"
          :disabled="!formData.provider"
        />
      </n-form-item>

      <!-- 端点配置说明 -->
      <n-divider title-placement="left" class="!my-3">
        <span class="text-xs text-[var(--text-secondary)]">API 端点</span>
      </n-divider>
      
      <div class="endpoint-list">
        <div class="endpoint-item">
          <span class="endpoint-label">文生图</span>
          <n-tag size="small" type="success" class="endpoint-tag">/images/generations</n-tag>
        </div>
        <div class="endpoint-item">
          <span class="endpoint-label">图生图</span>
          <n-tag size="small" type="success" class="endpoint-tag">/images/generations</n-tag>
        </div>
        <div class="endpoint-item">
          <span class="endpoint-label">图生视频</span>
          <n-tag size="small" type="warning" class="endpoint-tag">/videos</n-tag>
        </div>
        <div class="endpoint-item">
          <span class="endpoint-label">视频查询</span>
          <n-tag size="small" type="warning" class="endpoint-tag">/videos/{taskId}</n-tag>
        </div>
        <div class="endpoint-item">
          <span class="endpoint-label">AI 润色</span>
          <n-tag size="small" type="info" class="endpoint-tag">/chat/completions</n-tag>
        </div>
      </div>

      <n-alert v-if="!isConfigured" type="warning" title="未配置" class="mb-4">
        请选择提供商并配置 API Key 以使用画布功能
      </n-alert>

      <n-alert v-else type="success" title="已配置" class="mb-4">
        {{ currentProviderName }} 已就绪，可以使用画布功能
      </n-alert>
    </n-form>

    <template #footer>
      <div class="flex justify-end gap-2">
        <n-button @click="handleClear" tertiary>清除配置</n-button>
        <n-button @click="showModal = false">取消</n-button>
        <n-button type="primary" @click="handleSave">保存</n-button>
      </div>
    </template>
  </n-modal>
</template>

<script setup>
/**
 * API Settings Component | API 设置组件
 * Modal for configuring drawing provider
 */
import { ref, reactive, watch, computed } from 'vue'
import { NModal, NForm, NFormItem, NInput, NSelect, NButton, NAlert, NDivider, NTag } from 'naive-ui'
import { gptServerStore } from '@/store'
import { useApiConfig } from '../hooks'

// Props | 属性
const props = defineProps({
  show: {
    type: Boolean,
    default: false
  }
})

// Emits | 事件
const emit = defineEmits(['update:show', 'saved'])

// API Config hook | API 配置 hook
const { apiKey, baseUrl, isConfigured, setApiKey, setBaseUrl, clear: clearConfig } = useApiConfig()

// Modal visibility | 弹窗可见性
const showModal = ref(props.show)

// Provider options | 提供商选项
const providerOptions = [
  { label: 'Midjourney', value: 'midjourney' },
  { label: '豆包 (Doubao)', value: 'doubao' },
  { label: 'Kling', value: 'kling' },
  { label: 'Pika', value: 'pika' },
  { label: 'Pixverse', value: 'pixverse' },
  { label: 'Runway', value: 'runway' },
  { label: 'Luma', value: 'luma' },
  { label: 'Viggle', value: 'viggle' },
  { label: 'Ideo', value: 'ideo' },
]

// Form data | 表单数据
const formData = reactive({
  provider: '',
  apiKey: apiKey.value,
  baseUrl: baseUrl.value
})

// Current provider name | 当前提供商名称
const currentProviderName = computed(() => {
  const option = providerOptions.find(opt => opt.value === formData.provider)
  return option?.label || '提供商'
})

// Get provider config from gptServerStore | 从 gptServerStore 获取提供商配置
const getProviderConfig = (provider) => {
  const configMap = {
    midjourney: {
      server: gptServerStore.myData.MJ_SERVER,
      key: gptServerStore.myData.MJ_API_SECRET
    },
    doubao: {
      server: gptServerStore.myData.DOUBAO_SERVER,
      key: gptServerStore.myData.DOUBAO_KEY
    },
    kling: {
      server: gptServerStore.myData.KLING_SERVER,
      key: gptServerStore.myData.KLING_KEY
    },
    pika: {
      server: gptServerStore.myData.PIKA_SERVER,
      key: gptServerStore.myData.PIKA_KEY
    },
    pixverse: {
      server: gptServerStore.myData.PIXVERSE_SERVER,
      key: gptServerStore.myData.PIXVERSE_KEY
    },
    runway: {
      server: gptServerStore.myData.RUNWAY_SERVER,
      key: gptServerStore.myData.RUNWAY_KEY
    },
    luma: {
      server: gptServerStore.myData.LUMA_SERVER,
      key: gptServerStore.myData.LUMA_KEY
    },
    viggle: {
      server: gptServerStore.myData.VIGGLE_SERVER,
      key: gptServerStore.myData.VIGGLE_KEY
    },
    ideo: {
      server: gptServerStore.myData.IDEO_SERVER,
      key: gptServerStore.myData.IDEO_KEY
    }
  }
  return configMap[provider] || { server: '', key: '' }
}

// Handle provider change | 处理提供商变更
const handleProviderChange = (provider) => {
  const config = getProviderConfig(provider)
  formData.baseUrl = config.server || ''
  formData.apiKey = config.key || ''
}

// Initialize with current config | 使用当前配置初始化
const initFormData = () => {
  // Try to detect current provider from baseUrl
  const currentUrl = baseUrl.value
  let detectedProvider = ''
  
  for (const option of providerOptions) {
    const config = getProviderConfig(option.value)
    if (config.server === currentUrl) {
      detectedProvider = option.value
      break
    }
  }
  
  formData.provider = detectedProvider
  formData.apiKey = apiKey.value
  formData.baseUrl = baseUrl.value
}

// Watch prop changes | 监听属性变化
watch(() => props.show, (val) => {
  showModal.value = val
  if (val) {
    initFormData()
  }
})

// Watch modal changes | 监听弹窗变化
watch(showModal, (val) => {
  emit('update:show', val)
})

// Handle save | 处理保存
const handleSave = () => {
  if (!formData.provider) {
    window.$message?.warning('请选择提供商')
    return
  }
  
  if (formData.apiKey) {
    setApiKey(formData.apiKey)
  }
  if (formData.baseUrl) {
    setBaseUrl(formData.baseUrl)
  }
  
  // Save to localStorage for persistence
  localStorage.setItem('canvas_provider', formData.provider)
  
  showModal.value = false
  emit('saved')
  window.$message?.success('配置已保存')
}

// Handle clear | 处理清除
const handleClear = () => {
  clearConfig()
  formData.provider = ''
  formData.apiKey = ''
  formData.baseUrl = ''
  localStorage.removeItem('canvas_provider')
  window.$message?.info('配置已清除')
}
</script>

<style scoped>
.endpoint-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 16px;
  padding: 12px;
  background: var(--bg-secondary, #f5f5f5);
  border-radius: 6px;
}

.endpoint-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.endpoint-label {
  font-size: 13px;
  color: var(--text-secondary, #666);
  min-width: 80px;
}

.endpoint-tag {
  font-family: monospace;
  font-size: 12px;
}
</style>
