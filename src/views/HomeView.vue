<script setup lang="ts">
import TheWelcome from '../components/TheWelcome.vue'
import { onMounted, ref } from 'vue'
import liff from '@line/liff'
import axios from 'axios'

const accessToken = ref()
const profile = ref()
const error = ref()
const serviceNotificationToken = ref()
const context = ref()

const notificationToken = async () => {
  try {
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
    error.value = true
  } catch (e) {
    error.value = false
  }
}

const sendMessage = async () => {
  //
  //   const message = {
  //     templateName: 'hello',
  //     params: {
  //       date: '2020-04-23',
  //       username: 'Brown & Cony'
  //     },
  //     notificationToken: 'YOUR_NOTIFICATION_TOKEN' // Thay thế bằng Service Notification Token
  //   }
  //   const { data } = await axios.post('https://api.line.me/message/v3/notifier/send', message, {
  //     headers: {
  //       Authorization: 'Bearer ' + accessToken.value,
  //       'Content-Type': 'application/json'
  //     }
  //   })
  //   console.log(data)

  //
  //   liff
  //     .sendMessages([
  //       {
  //         type: 'text',
  //         text: 'Hello, World!'
  //       }
  //     ])
  //     .then(() => {
  //       window.alert('message sent')
  //     })
  //     .catch((err) => {
  //       error.value = err
  //       window.alert('error')
  //     })

  liff
    .shareTargetPicker(
      [
        {
          type: 'text',
          text: 'Hello, World!'
        }
      ],
      {
        isMultiple: true
      }
    )
    .then(function (res) {
      if (res) {
        // succeeded in sending a message through TargetPicker
        console.log(`[${res.status}] Message sent!`)
      } else {
        // sending message canceled
        console.log('TargetPicker was closed!')
      }
    })
    .catch(function (error) {
      // something went wrong before sending a message
      console.log('something wrong happen: ' + error)
    })
}
onMounted(async () => {
  // 1. LIFFの初期化
  //   2000661938-g8YDPLkl
  // 2000661940-rVB8DdeY

  //   api login:
  // 2000599010-Gbbpqw9E

  await liff.init({ liffId: '2000661940-rVB8DdeY' }).catch((err) => {
    console.error(err)
    window.alert('LIFFの初期化失敗。\n' + err)
  })
  if (!liff.isLoggedIn()) {
    await liff.login()
  }
  accessToken.value = liff.getAccessToken()
  profile.value = await liff.getProfile()
  console.log(accessToken.value, profile.value, profile.value.statusMessage)
  console.log(12345678)
  liff.permission.query('chat_message.write').then((permissionStatus) => {
    console.log(permissionStatus)
    // permissionStatus = { state: 'granted' }
  })
  context.value = liff.getContext()
  console.log(context.value)

  //   await notificationToken()
})
</script>

<template>
  <main>
    <!-- <TheWelcome /> -->
    <div class="info">
      <div class="logo">
        <img :src="profile.pictureUrl" alt="" v-if="profile?.pictureUrl" />
      </div>
      <div style="display: flex; flex-direction: column; gap: 10px; margin: 20px 0px">
        <div>
          <p>{{ 'name: ' + profile?.displayName }}</p>
          <hr />
        </div>
        <div>
          <p>{{ 'userId: ' + profile?.userId }}</p>
          <hr />
        </div>
      </div>
    </div>
    <div @click="sendMessage" class="send">send messages ({{ error }})</div>
    <div>{{ context }}</div>
  </main>
</template>
<style>
main {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 24px;
}
.info {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.logo img {
  border-radius: 50%;
  width: 200px;
  aspect-ratio: 1;
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
