<script setup lang="ts">
import { ref, onMounted } from 'vue'

const telegramId = ref<number | null>(null)
const fullName = ref('')
const loading = ref(false)
const success = ref(false)
const error = ref('')

onMounted(() => {
  const tg = (window as { Telegram?: { WebApp?: { initDataUnsafe?: { user?: { id: number, first_name: string, last_name?: string } } } } }).Telegram?.WebApp
  if (tg && tg.initDataUnsafe?.user) {
    telegramId.value = tg.initDataUnsafe.user.id
    fullName.value = tg.initDataUnsafe.user.first_name + ' ' + (tg.initDataUnsafe.user.last_name || '')
  }
})

const submitForm = async () => {
  loading.value = true
  error.value = ''
  success.value = false
  try {
    const response = await fetch('https://7580d4a0132f.ngrok-free.app/api/autotest/v1/tg/auth', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        telegram_id: telegramId.value,
        full_name: fullName.value
      })
    })
    if (!response.ok) {
      throw new Error('Ошибка при отправке данных')
    }
    success.value = true
  } catch (e: unknown) {
    if (e instanceof Error) {
      error.value = e.message
    } else {
      error.value = String(e)
    }
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <main>
    <h1>Авторизация через Telegram</h1>
    <form @submit.prevent="submitForm">
      <div>
        <label>Telegram ID:</label>
        <input :value="telegramId ?? ''"
          disabled />
      </div>
      <div>
        <label>Полное имя:</label>
        <input :value="fullName"
          disabled />
      </div>
      <button type="submit"
        :disabled="loading || !telegramId">Отправить</button>
    </form>
    <div v-if="success"
      style="color: green; margin-top: 10px;">
      Данные успешно отправлены!
    </div>
    <div v-if="error"
      style="color: red; margin-top: 10px;">
      {{ error }}
    </div>
  </main>
</template>