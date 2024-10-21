<script setup>
import { paragon } from '@useparagon/connect';
import { ref } from 'vue';

/*
API References:
  - Gmail: https://developers.google.com/gmail/api/reference/rest
  - Outlook: https://learn.microsoft.com/en-us/graph/api/resources/calendar?view=graph-rest-1.0
 */

const userInfo = ref(null);
const userToken = ref(null);
const threads = ref(null);

async function connect(service) {
  await paragon.authenticate(import.meta.env.VITE_PARAGON_PROJECT_ID, userToken.value);
  await paragon.connect(service, {
    onSuccess: () => {console.log('success!')},
    onError: (error) => {console.error(error);}
  });
}

async function disconnect(service) {
  await paragon.authenticate(import.meta.env.VITE_PARAGON_PROJECT_ID, userToken.value);
  await paragon.uninstallIntegration(service, {
    onSuccess: () => {console.log('success!')},
    onError: (error) => {console.error(error);}
  });
}

async function getUserInfo() {
  await paragon.authenticate(import.meta.env.VITE_PARAGON_PROJECT_ID, userToken.value);
  userInfo.value = await paragon.getUser();
}

async function getThreadsByAddress(address) {
  await paragon.authenticate(import.meta.env.VITE_PARAGON_PROJECT_ID, userToken.value);
  threads.value = await paragon.request('gmail', `gmail/v1/users/me/threads?maxResults=15&q=${encodeURIComponent(address)}`, {
    method: "GET"
  });
}
</script>

<template>
  <div>
    User Token:
    <input type="text" v-model="userToken" />
  </div>
  <hr>
  <div>
    <button @click="connect('gmail')" :disabled="!userToken">Connect Gmail</button>
    <button @click="connect('outlook')" :disabled="!userToken">Connect Outlook</button>
  </div>
  <hr>
  <div>
    <button @click="getUserInfo" :disabled="!userToken">Get User Info</button>
    {{ userInfo }}
  </div>
  <hr>
  <div>
    <button @click="getThreadsByAddress('rmessenger@gmail.com')" :disabled="!userToken">Search threads</button>
    {{ threads }}
  </div>
  <hr>
  <div>
    <button @click="disconnect('gmail')" :disabled="!userToken">Disconnect Gmail</button>
    <button @click="disconnect('outlook')" :disabled="!userToken">Disconnect Outlook</button>
  </div>
</template>

<style scoped>
</style>
