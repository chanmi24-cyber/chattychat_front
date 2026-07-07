<template>
  <div class="container">
    <h2>안녕하세요, {{ nickname }}님!</h2>
    <p>상대방을 찾는 중입니다...</p>
    <div class="spinner"></div>
    <button @click="leave">나가기</button>
  </div>
</template>

<script setup>
import { onMounted} from 'vue'

const props = defineProps(['nickname'])
const emit = defineEmits(['matched', 'leave'])

let ws = null

onMounted(() => {
  connect()
})


const connect = () => {
  ws = new WebSocket('ws://localhost:8080/chat')

  ws.onopen = () => {
    ws.send(JSON.stringify({
      type: 'ENTER',
      sender: props.nickname
    }))
  }

  ws.onmessage = (event) => {
    const data = JSON.parse(event.data)
    if (data.type === 'MATCHED') {
      emit('matched', {
        roomId: data.roomId,
        partner: data.message,
        ws: ws
      })
    }
  }

  ws.onerror = (error) => {
    console.error('WebSocket 오류:', error)
  }
}

const leave = () => {
  if (ws && ws.readyState === WebSocket.OPEN) {
    ws.close()
  }
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