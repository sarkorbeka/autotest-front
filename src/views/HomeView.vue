<script setup lang="ts">
import { ref, onMounted, reactive } from 'vue'

const loading = ref<boolean>(false)
const success = ref<boolean>(false)
const error = ref<string>('')

const userData = reactive<{ telegram_id: number | null; full_name: string }>({
  telegram_id: null,
  full_name: ''
})

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
    localStorage.setItem("user", JSON.stringify(tg.initDataUnsafe.user));

    console.log(JSON.parse(localStorage.getItem("user") ?? ""));

    const store = JSON.parse(localStorage.getItem("user") ?? "");

    userData.telegram_id = store.id;
    userData.full_name = store.first_name + ' ' + (store.last_name || '');

    submitForm()
  }
})

const submitForm = async () => {
  loading.value = true
  error.value = ''
  success.value = false
  try {
    const response = await fetch('https://halley-peritectic-unpreclusively.ngrok-free.dev/api/autotest/v1/tg/auth', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        telegram_id: userData.telegram_id,
        full_name: userData.full_name
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
    <h1>Welcome - {{ userData.full_name }}</h1>
    <!-- <h1>Авторизация через Telegram</h1> -->
    <!-- <form @submit.prevent="submitForm">
      <div>
        <label>Telegram ID:</label>
        <p>{{ userData.telegram_id }}</p>
        <input v-model="userData.telegram_id"
          disabled />
      </div>

      <div>
        <label>Полное имя:</label>
        <p class="text-[red]">{{ userData.full_name }}</p>
        <input v-model="userData.full_name"
          disabled />
      </div>

      <button type="submit"
        :disabled="loading || !userData.telegram_id">Отправить</button>
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
      <pre>{{ userData }}</pre>
      <div>telegramID: {{ telegramID }}</div>
      <div>full_name: {{ full_name }}</div>
    </div> -->
  </main>
</template>