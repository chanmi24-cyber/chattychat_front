<template>
  <div class="container">
    <h2>안녕하세요, {{ nickname }}님!</h2>
    <p>상대방을 찾는 중입니다...</p>
    <div class="spinner"></div>
    <button @click="leave">나가기</button>
  </div>
</template>

<script setup>
import { onMounted, onUnmounted } from 'vue'
import SockJS from 'sockjs-client'
import Stomp from 'stompjs'

const props = defineProps(['nickname'])
const emit = defineEmits(['matched', 'leave'])

let socket = null
let stompClient = null

onMounted(() => {
  connect()
})

onUnmounted(() => {
  disconnect()
})

const connect = () => {
  socket = new SockJS('http://localhost:8080/chat')
  stompClient = Stomp.over(socket)

  stompClient.connect({}, () => {
    stompClient.send('/app/chat', {}, JSON.stringify({
      type: 'ENTER',
      sender: props.nickname
    }))

    stompClient.subscribe(`/topic/user/${props.nickname}`, (message) => {
      const data = JSON.parse(message.body)
      if (data.type === 'MATCHED') {
        emit('matched', {
          roomId: data.roomId,
          partner: data.message
        })
      }
    })
  })
}

const disconnect = () => {
  if (stompClient) stompClient.disconnect()
}

const leave = () => {
  disconnect()
  emit('leave')
}
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  gap: 20px;
}
.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #ccc;
  border-top-color: #42b883;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}
@keyframes spin {
  to { transform: rotate(360deg); }
}
button {
  padding: 10px 20px;
  background-color: #ff6b6b;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
</style>