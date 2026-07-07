<template>
  <div class="container">
    <div class="header">
      <h2>{{ partner }}님과 대화 중</h2>
      <div class="buttons">
        <button class="wait-btn" @click="wait">채팅 종료</button>
        <button class="leave-btn" @click="leave">나가기</button>
      </div>
    </div>
    <div class="messages" ref="messageBox">
      <div
        v-for="(msg, index) in messages"
        :key="index"
        :class="msg.sender === nickname ? 'my-message' : 'other-message'"
      >
        <span class="sender">{{ msg.sender }}</span>
        <span class="message">{{ msg.message }}</span>
      </div>
    </div>
    <div class="input-area">
      <input
        v-model="input"
        type="text"
        placeholder="메시지 입력"
        @keyup.enter="send"
      />
      <button @click="send">전송</button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, nextTick } from 'vue'
import SockJS from 'sockjs-client'
import Stomp from 'stompjs'

const props = defineProps(['nickname', 'roomId', 'partner'])
const emit = defineEmits(['wait', 'leave'])

const messages = ref([])
const input = ref('')
const messageBox = ref(null)

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
    stompClient.subscribe(`/topic/room/${props.roomId}`, (message) => {
      const data = JSON.parse(message.body)
      if (data.type === 'LEAVE') {
        messages.value.push({
          sender: '시스템',
          message: '상대방이 나갔습니다.'
        })
      } else {
        messages.value.push(data)
      }
      scrollToBottom()
    })
  })
}

const disconnect = () => {
  if (stompClient) stompClient.disconnect()
}

const send = () => {
  if (!input.value.trim()) return
  stompClient.send('/app/chat', {}, JSON.stringify({
    type: 'TALK',
    roomId: props.roomId,
    sender: props.nickname,
    message: input.value
  }))
  input.value = ''
}

const wait = () => {
  stompClient.send('/app/chat', {}, JSON.stringify({
    type: 'WAIT',
    sender: props.nickname
  }))
  disconnect()
  emit('wait')
}

const leave = () => {
  stompClient.send('/app/chat', {}, JSON.stringify({
    type: 'LEAVE',
    sender: props.nickname
  }))
  disconnect()
  emit('leave')
}

const scrollToBottom = async () => {
  await nextTick()
  if (messageBox.value) {
    messageBox.value.scrollTop = messageBox.value.scrollHeight
  }
}
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  height: 100vh;
}
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  background-color: #42b883;
  color: white;
}
.buttons {
  display: flex;
  gap: 10px;
}
.messages {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.my-message {
  align-self: flex-end;
  background-color: #42b883;
  color: white;
  padding: 10px;
  border-radius: 10px;
  max-width: 60%;
}
.other-message {
  align-self: flex-start;
  background-color: #f0f0f0;
  padding: 10px;
  border-radius: 10px;
  max-width: 60%;
}
.sender {
  font-size: 12px;
  display: block;
  margin-bottom: 4px;
}
.input-area {
  display: flex;
  padding: 15px;
  gap: 10px;
  border-top: 1px solid #ccc;
}
input {
  flex: 1;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 5px;
}
button {
  padding: 10px 20px;
  background-color: #42b883;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
.wait-btn {
  background-color: #ffa500;
}
.leave-btn {
  background-color: #ff6b6b;
}
</style>