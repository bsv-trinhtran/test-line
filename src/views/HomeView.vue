<script setup lang="ts">
import TheWelcome from '../components/TheWelcome.vue'
import { onMounted, ref } from 'vue'
import liff from '@line/liff'
import axios from 'axios'

const accessToken = ref()
const profile = ref()
const serviceNotificationToken = ref()

const notificationToken = async () => {
  const { data } = await axios.post(
    'https://api.line.me/message/v3/notifier/token',
    {},
    {
      headers: {
        Authorization: `Bearer ${accessToken.value}`,
        'Content-Type': 'application/json'
      }
    }
  )
  console.log(data)
}

const sendMessage = async () => {
  const message = {
    templateName: 'hello',
    params: {
      date: '2020-04-23',
      username: 'Brown & Cony'
    },
    notificationToken: 'YOUR_NOTIFICATION_TOKEN' // Thay thế bằng Service Notification Token
  }
  const { data } = await axios.post('https://api.line.me/message/v3/notifier/send', message, {
    headers: {
      Authorization: 'Bearer ' + accessToken.value,
      'Content-Type': 'application/json'
    }
  })
  console.log(data)
}
onMounted(async () => {
  // 1. LIFFの初期化
  await liff.init({ liffId: '2000661940-rVB8DdeY' }).catch((err) => {
    console.error(err)
    window.alert('LIFFの初期化失敗。\n' + err)
  })
  if (!liff.isLoggedIn()) {
    await liff.login()
    // return
  }
  accessToken.value = liff.getAccessToken()
  profile.value = await liff.getProfile()
  console.log(accessToken.value, profile.value)
  console.log(123)
  await notificationToken()
})
</script>

<template>
  <main>
    <!-- <TheWelcome /> -->
    <div @click="sendMessage" class="send">send messages</div>
  </main>
</template>
<style>
main {
  display: flex;
  justify-content: center;
}
.send {
  width: 200px;
  background: red;
  border-radius: 8px;
  text-align: center;
  padding: 12px 0px;
  cursor: pointer;
}
</style>
