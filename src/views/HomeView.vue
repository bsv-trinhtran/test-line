<template>
  <div style="display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100vh; gap: 20px;">
    <Button @click="loginWithLine">Login with Line</Button>

    <div style="display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 10px;">
      <label for="lineId">Line ID</label>
      <input type="text" id="lineId" v-model="lineId" style="height: 50px; width: 300px; padding: 10px; border-radius: 5px; border: 1px solid #ccc;">
    </div>
  </div>
</template>

<script setup>
import liff from '@line/liff';
import { ref } from 'vue';


const lineId = ref('');

const loginWithLine = async () => {
  await liff.init({
    liffId: '2000635253-WADl2ApA',
  });
    if (!liff.isLoggedIn()) {
        const currentUrl = 'https://test-line-wheat.vercel.app/';
        await liff.login({
            redirectUri: currentUrl,
        });
        return;
     }
   await liff.logout();
};
if(liff.isLoggedIn()) {
const profile = await liff.getProfile();
   lineId.value = profile.userId;
}
</script>

<style>
</style>
