<script setup lang='ts'>
import { computed, ref } from 'vue'
import { NButton, NInput, NModal, useMessage } from 'naive-ui'
import { fetchVerify } from '@/api'
import { useAuthStore } from '@/store'
import Icon403 from '@/icons/403.vue'

interface Props {
  visible: boolean
}

defineProps<Props>()

const authStore = useAuthStore()

const ms = useMessage()

const loading = ref(false)
const username = ref('')
const password = ref('')

const disabled = computed(() => !username.value.trim() || !password.value.trim() || loading.value)

async function handelRegister() {
  window.open('https://www.app4gpt.com/#/register', '_blank')
}

async function handleVerify() {
  const usernameStr = username.value.trim()
  const passwordStr = password.value.trim()

  if (!usernameStr || !passwordStr)
    return

  try {
    loading.value = true
    const res: any = await fetchVerify(usernameStr, passwordStr)
    authStore.setToken(res.data.token)
    ms.success('success')
    window.location.reload()
  }
  catch (error: any) {
    ms.error(error.message ?? 'error')
    authStore.removeToken()
    username.value = ''
  }
  finally {
    loading.value = false
  }
}

function handlePress(event: KeyboardEvent) {
  if (event.key === 'Enter' && !event.shiftKey) {
    event.preventDefault()
    handleVerify()
  }
}
</script>

<template>
  <NModal :show="visible" style="width: 90%; max-width: 640px">
    <div class="p-10 bg-white rounded dark:bg-slate-800">
      <div class="space-y-4">
        <header class="space-y-2">
          <h2 class="text-2xl font-bold text-center text-slate-800 dark:text-neutral-200">
            403
          </h2>
          <p class="text-base text-center text-slate-500 dark:text-slate-500">
            {{ $t('common.unauthorizedTips') }}
          </p>
          <Icon403 class="w-[200px] m-auto" />
        </header>
        <NInput v-model:value="username" type="text" placeholder="用户名" />
        <NInput v-model:value="password" type="password" placeholder="密码" @keypress="handlePress" />
        <NButton
          block
          type="primary"
          :disabled="disabled"
          :loading="loading"
          @click="handleVerify"
        >
          登录
        </NButton>
        <NButton
          block
          type="primary"
          :disabled="false"
          :loading="loading"
          @click="handelRegister"
        >
          注册
        </NButton>
      </div>
    </div>
  </NModal>
</template>
