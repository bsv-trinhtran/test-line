<script setup lang="ts">
import TheWelcome from '../components/TheWelcome.vue'
import { onMounted } from 'vue'
import liff from '@line/liff'

function login() {
  if (!liff.isLoggedIn()) {
    liff.login()
  }
}
const sendMessage = () => {
  liff
    .sendMessages([
      {
        type: 'text',
        text: 'Hello, World!'
      }
    ])
    .then(() => {
      console.log('message sent')
    })
    .catch((err) => {
      console.log('error', err)
    })
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
  const accessToken = liff.getAccessToken()
  const profile = await liff.getProfile()
  console.log({ accessToken, profile })
  console.log(123)
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
