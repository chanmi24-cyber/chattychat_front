<template>
  <div>
    <NicknameForm v-if="step === 'nickname'" @enter="handleEnter" />
    <WaitingRoom v-if="step === 'waiting'" :nickname="nickname" @matched="handleMatched" @leave="handleLeave" />
    <ChatRoom v-if="step === 'chat'" :nickname="nickname" :room-id="roomId" :partner="partner" :ws="ws" @wait="handleWait" @leave="handleLeave" />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import NicknameForm from './components/NicknameForm.vue'
import WaitingRoom from './components/WaitingRoom.vue'
import ChatRoom from './components/ChatRoom.vue'

const step = ref('nickname')
const nickname = ref('')
const roomId = ref('')
const partner = ref('')
const ws = ref(null)

const handleEnter = (name) => {
  nickname.value = name
  step.value = 'waiting'
}

const handleMatched = (data) => {
  roomId.value = data.roomId
  partner.value = data.partner
  ws.value = data.ws
  step.value = 'chat'
}

const handleWait = () => {
  step.value = 'waiting'
}

const handleLeave = () => {
  nickname.value = ''
  roomId.value = ''
  partner.value = ''
  ws.value = null
  step.value = 'nickname'
}
</script>