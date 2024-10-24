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
const threadId = ref(null);
const thread = ref(null);
const sendTo = ref(null);
const subject = ref(null);
const body = ref(null);

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

async function getThreadById(id) {
  await paragon.authenticate(import.meta.env.VITE_PARAGON_PROJECT_ID, userToken.value);
  thread.value = await paragon.request('gmail', `gmail/v1/users/me/threads/${id}`, {
    method: "GET"
  });
}

async function sendMessage(to, subject, body) {
  await paragon.authenticate(import.meta.env.VITE_PARAGON_PROJECT_ID, userToken.value);
  await paragon.request('gmail', `gmail/v1/users/me/messages/send`, {
    method: "POST",
    headers: {
      'Content-Type': 'application/json',
    },
    body: {
      raw: btoa(
      `Content-Type: text/plain; charset=UTF-8
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
to: ${to}
from: robin.messenger@dashhudson.com
subject: ${subject}

${body}`
      )
    }
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
    To:
    <input type="text" v-model="sendTo" />
  </div>
  <div>
    Subject:
    <input type="text" v-model="subject" />
  </div>
  <div>
    Body:
    <textarea v-model="body" />
  </div>
  <div>
    <button @click="sendMessage(sendTo, subject, body)" :disabled="!sendTo || !subject || !body">Send</button>
  </div>
  <hr>
  <div>
    Thread ID:
    <input type="text" v-model="threadId" />
    <button @click="getThreadById(threadId)" :disabled="!threadId">Get Single Thread By ID</button>
    {{ thread }}
  </div>
  <hr>
  <div>
    <button @click="disconnect('gmail')" :disabled="!userToken">Disconnect Gmail</button>
    <button @click="disconnect('outlook')" :disabled="!userToken">Disconnect Outlook</button>
  </div>
</template>

<style scoped>
</style>
