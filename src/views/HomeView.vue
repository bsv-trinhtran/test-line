<script setup lang="ts">
import TheWelcome from '../components/TheWelcome.vue'
import { onMounted } from 'vue'
import liff from '@line/liff'

function login() {
  if (!liff.isLoggedIn()) {
    liff.login()
  }
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
    <TheWelcome />
  </main>
</template>
