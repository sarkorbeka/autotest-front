<script setup lang="ts">
import { ref, onMounted } from 'vue'

const telegramId = ref<number | null>(null)
const fullName = ref<string>('')
const loading = ref<boolean>(false)
const success = ref<boolean>(false)
const error = ref<string>('')

type TelegramUser = {
  id: number;
  first_name: string;
  last_name?: string;
  username?: string;
  language_code?: string;
};
type TelegramWebApp = {
  initDataUnsafe?: {
    user?: TelegramUser;
  };
};
type TelegramGlobal = {
  WebApp?: TelegramWebApp;
};

const telegramGlobal = ref<TelegramGlobal | null>(null)
onMounted(() => {
  telegramGlobal.value = (window as { Telegram?: TelegramGlobal }).Telegram ?? null
  console.log(telegramGlobal.value?.WebApp.initDataUnsafe)
  const tg = telegramGlobal.value?.WebApp
  if (tg && tg.initDataUnsafe?.user) {
    console.log("hello", tg.initDataUnsafe.user);

    telegramId.value = tg.initDataUnsafe.user.id
    fullName.value = tg.initDataUnsafe.user.first_name + ' ' + (tg.initDataUnsafe.user.last_name || '')
    console.log(telegramId.value, fullName.value);
    
  }
})

// onMounted(() => {
//   console.log("Telegram global:", window.Telegram.WebApp.initDataUnsafe.user)
//   console.log("WebApp:", window.Telegram?.WebApp)
// })

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
    <!-- <h1>Авторизация через Telegram</h1> -->
    <form @submit.prevent="submitForm">
      <div>
        <label>Telegram ID:</label>
        <p>{{ telegramId }}</p>
        <input v-model="telegramId"
          disabled />
      </div>

      <div>
        <label>Полное имя:</label>
        <p>{{ fullName }}</p>
        <input v-model="fullName"
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
    <div style="margin-top: 20px;">
      <h2>Глобальный объект Telegram:</h2>
      <pre>{{ telegramGlobal }}</pre>
    </div>
  </main>
</template>