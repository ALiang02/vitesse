<script setup lang="ts">
import axios from 'axios'
import dayjs from 'dayjs'
const user = useStorage('user', { name: '', signed: false })
const message = ref('')
const loading = ref(false)
interface Message {
  id: string
  sender: string
  content: string
  timestamp: number
}
const messageList = shallowRef<Array<Message>>([])
const notification = useNotification()
const api = axios.create({
  headers: {
    'X-Username': user.value.name,
  },
  timeout: 20000,
})
api.interceptors.response.use((response) => {
  // 2xx 范围内的状态码都会触发该函数。
  // 对响应数据做点什么
  return response
}, (error) => {
  // 超出 2xx 范围的状态码都会触发该函数。
  // 对响应错误做点什么
  notification.error({ content: error.message, duration: 3000 })
  return Promise.reject(error)
})

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
async function send() {
  if (message.value && !loading.value) {
    loading.value = true
    try {
      await api({
        url: '/v1/chat',
        method: 'post',
        data: {
          message: message.value,
        },
      })
      await getMessageList()
      message.value = ''
    }
    catch (error) {
    }
    finally {
      loading.value = false
    }
  }
}

async function getMessageList(flag?: any) {
  if (flag)
    loading.value = true
  try {
    const res = await api({
      url: '/v1/chat',
      method: 'get',
    })
    messageList.value = res.data.record
  }
  finally {
    loading.value = false
  }
}

async function clearMessageList() {
  await api({
    url: '/v1/chat',
    method: 'delete',
  })
}

onMounted(() => {
  getMessageList()
})
</script>

<template>
  <div h-full w-full flex justify-center items-center>
    <template v-if="!user.signed">
      <n-input
        v-model:value="user.name"
        :loading="loading"
        class="w-1/2!"
        placeholder="Please Input Your Name"
        :allow-input="(value: string) =>
          !value.startsWith(' ') && !value.endsWith(' ')"
        :disabled="loading"
        @keyup.enter="login"
      />
    </template>
    <template v-else>
      <div class="h-4/5 max-w-1080px w-4/5 bg  overflow-y-auto relative">
        <div v-for="message in messageList" :key="message.id" class="message-box">
          <div>
            <span class="name">{{ message.sender }}</span>
            <span class="time">{{ dayjs(message.timestamp).format('YYYY/MM/DD HH:mm:ss') }}</span>
          </div>
          <div class="message" v-html="message.content" />
        </div>
      </div>
      <div class="send-box">
        <n-input-group class="w-full mx-auto flex justify-center">
          <n-input
            v-model:value="message"
            type="textarea"
            size="large"
            round
            :loading="loading"
            :disabled="loading"
            clearable
            :autosize="{
              minRows: 1,
              maxRows: 3,
            }"
          />
          <n-button size="large" :disabled="loading" @click="send">
            Send
          </n-button>
          <n-button size="large" :disabled="loading" @click="getMessageList('loding')">
            Refresh
          </n-button>
          <n-popconfirm
            @positive-click="clearMessageList"
          >
            <template #trigger>
              <n-button size="large" :disabled="loading">
                Clear
              </n-button>
            </template>
            Are you sure?。
          </n-popconfirm>
        </n-input-group>
      </div>
    </template>
  </div>
</template>

<style scoped>
.bg {
  background-color: rgb(54,57,63);
  border-radius: 20px;
  padding: 20px;
  margin-bottom: 40px;
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
position: fixed;
left: 0;
right: 0;
width: 80%;
max-width: 1080px;
bottom: 20px;
margin: 0 auto;
}
</style>
