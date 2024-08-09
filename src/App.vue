<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import SockJS from 'sockjs-client/dist/sockjs';
import Stomp, { type Message } from 'stompjs'

const BASE_URL = "http://localhost:8080"
const SOCKET_ENPOINT = "/ws"
const TOPIC = "/topic/chat"

interface MessageResponse {
  message: string,
  created: Date
}

const sock = new SockJS(`${BASE_URL}${SOCKET_ENPOINT}`)
const stomClient = Stomp.over(sock)

const connected = ref(false)
const messages = ref<MessageResponse[]>([])
const message = ref('')

const connect = () => {
  stomClient.connect({}, () => {
    connected.value = true
    stomClient.subscribe(TOPIC, (msg: Message) => {
      const payload = (JSON.parse(msg.body) as MessageResponse)
      messages.value = [...messages.value, payload]
    })
  })
}

const send = async () => {
  const msg = {
    message: message.value
  }
  const res = await fetch(`${BASE_URL}/api/chat`, {
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/json'
    },
    method: "POST",
    body: JSON.stringify(msg)
  })
  console.log(await res.text());
  message.value = ''
}

onMounted(() => {
  connect()
})

onUnmounted(() => {
  stomClient.disconnect(() => {
    connected.value = false
  })
})

</script>

<template>
  <div>
    <h1>Connection: {{ connected }}</h1>
    <hr />
    <ul>
      <li v-for="(msg, index) in messages" :key="index">{{ msg.message }}</li>
    </ul>
    <form @submit.prevent="send">
      <div style="display: flex; flex-direction: column;">
        <label for="message">Message</label>
        <input type="text" id="message" v-model="message" />
      </div>
      <button type="submit" :disabled="!connected">Send</button>
    </form>
  </div>
</template>