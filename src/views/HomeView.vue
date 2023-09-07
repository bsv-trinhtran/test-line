<script setup lang="ts">
import TheWelcome from '../components/TheWelcome.vue'
import { onMounted, ref } from 'vue'
import liff from '@line/liff'
import axios from 'axios'

const accessToken = ref()
const profile = ref()
const serviceNotificationToken = ref()
const liffAccessToken = ref(
  'eyJhbGciOiJIUzI1NiJ9.tjnDJ4dtiuctZyVJ6B59dFKN92YUBlNoKixb2mbA8AYY1i09PIU6-LWcob4_yDFQJ5-B6UDnFhZmW2P2jEXIs6QE3U96vUObqB1-PHPeDB1LhexzA5xPAmDlLvifP67Pzy5iT7enGTFFEjrdngSQemhpfnplw6knopG6dZcYyio.ebWkxiEMFlpCIPMKmOBix8bkX375TWlTG3j'
)

const notificationToken = async () => {
  const { data } = await axios.post(
    'https://api.line.me/message/v3/notifier/token',
    {
      liffAccessToken: accessToken.value
    },
    {
      headers: {
        Authorization: `Bearer ${accessToken.value}`,
        'Content-Type': 'application/json'
      }
    }
  )
  console.log(data)
  console.log('notificationToken')
}

const sendMessage = async () => {
  //
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
  // eyJhbGciOiJIUzI1NiJ9.tjnDJ4dtiuctZyVJ6B59dFKN92YUBlNoKixb2mbA8AYY1i09PIU6-LWcob4_yDFQJ5-B6UDnFhZmW2P2jEXIs6QE3U96vUObqB1-PHPeDB1LhexzA5xPAmDlLvifP67Pzy5iT7enGTFFEjrdngSQemhpfnplw6knopG6dZcYyio.ebWkxiEMFlpCIPMKmOBix8bkX375TWlTG3j
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
  console.log(1234567)
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
