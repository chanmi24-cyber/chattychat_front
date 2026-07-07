<template>
  <div class="container">
    <h1>💬 ChattyChat</h1>
    <p>닉네임을 입력하고 채팅을 시작하세요</p>
    <div class="form">
      <input
        v-model="nickname"
        type="text"
        placeholder="닉네임 입력"
        @keyup.enter="enter"
        maxlength="10"
      />
      <button @click="enter">입장</button>
    </div>
    <p v-if="errorMsg" class="error">{{ errorMsg }}</p>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'

const emit = defineEmits(['enter'])
const nickname = ref('')
const errorMsg = ref('')

const enter = async () => {
  if (!nickname.value.trim()) {
    errorMsg.value = '닉네임을 입력해주세요.'
    return
  }

  try {
    const res = await axios.get(`http://localhost:8080/api/nickname/check`, {
      params: { nickname: nickname.value }
    })
    if (res.data) {
      emit('enter', nickname.value)
    } else {
      errorMsg.value = '이미 사용 중인 닉네임입니다.'
    }
  } catch (e) {
    errorMsg.value = '서버 연결 오류입니다.'
  }
}
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
}
.form {
  display: flex;
  gap: 10px;
  margin-top: 20px;
}
input {
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 5px;
}
button {
  padding: 10px 20px;
  font-size: 16px;
  background-color: #42b883;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
.error {
  color: red;
  margin-top: 10px;
}
</style>