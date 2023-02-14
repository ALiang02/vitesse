<script setup lang="ts">
import axios from 'axios'
const user = useStorage('user', { name: '', signed: false })
const message = ref('')
const loading = ref(false)
const notification = useNotification()
function login() {
  loading.value = true
  setTimeout(() => {
    loading.value = false
    if (user.value.name) {
      user.value.signed = true
      notification.success({ content: `Hello ${user.value.name}, welcome to chat!`, duration: 5000 })
    }
    else { notification.error({ content: 'You can\'t submit a null name', duration: 3000 }) }
  }, 1000)
}
function send() {
  if (message.value) {
    loading.value = true
    axios({
      method: 'post',
      url: '/v1/chat',
      headers: {
        'X-Username': user.value.name,
      },
      data: {
        message: message.value,
      },
    }).finally(() => {
      loading.value = false
    })
  }
}
const messageList = ref([{
  name: 'bot',
  content: `Hello ${user.value.name}, welcome to chat!`,
  order: 0,
}, {
  name: user.value.name,
  content: '你好！',
  order: 1,
}])
</script>

<template>
  <div h-full w-full flex justify-center items-center>
    <template v-if="!user.signed">
      <n-input
        v-model:value="user.name"
        :loading="loading" class="w-1/2!" placeholder="Please Input Your Name" :allow-input="(value: string) =>
          !value.startsWith(' ') && !value.endsWith(' ')" @keyup.enter="login"
      />
    </template>
    <template v-else>
      <div class="h-4/5 max-w-1080px w-4/5 bg relative overflow-y-auto">
        <div v-for="message in messageList" :key="message.order" class="message-box">
          <div>
            <span class="name">{{ message.name }}</span>
            <span class="time">2022/09/02 11:48</span>
          </div>
          <div class="message">
            {{ message.content }}
          </div>
        </div>
        <div class="send-box">
          <n-input
            v-model:value="message"
            class="w-4/5 mx-auto"
            size="large"
            round
            @keyup.enter="send"
          />
        </div>
      </div>
    </template>
  </div>
</template>

<style scoped>
.bg {
  background-color: rgb(54,57,63);
  border-radius: 20px;
  padding: 20px;
}
.name{
  font-size: 1rem;
  font-weight: 500;
  line-height: 1.375rem;
  margin-right: 10px;
  color:  hsl(0, 0%, 100%);;
}
.time {
  font-size: 0.75rem;
    line-height: 1.375rem;
    color: hsl(214, 4%, 65.3%);;
}
.message-box{
  margin: 20px 0;
}
.send-box{
position: sticky;
left: 0;
right: 0;
width: 80%;
bottom: 20px;
margin: 0 auto;
}
</style>
