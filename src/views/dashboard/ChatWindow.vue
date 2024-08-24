<!--
  模板部分:
    * 添加了 Clean 按钮，点击时调用 clearMessages 方法。
    * 输入框和按钮放在一个 div 容器中，使用类名 input-area 进行样式布局。
  脚本部分:
    * sendMessage 方法：当用户输入消息后，首先将消息添加到 messages 数组中，然后调用 getBotResponse 方法获取 LLM 响应。
    * 添加了 clearMessages 方法，该方法通过将 messages 数组清空来清除消息。
    * getBotResponse 方法：发送 POST 请求到 /api/llm-response，并将 LLM 的响应添加到 messages 数组中。
    * toggleMinimize 方法：切换窗口的最小化状态。
  样式部分:
    * 添加了 .input-area 类，以便将输入框和按钮放在一行，并确保它们之间有适当的间距。
-->

<template>
  <div :class="['chat-window', { minimized: isMinimized }]">
    <div class="chat-header" @click="toggleMinimize">
      <span>Chat AT</span>
    </div>
    <div v-if="!isMinimized" class="chat-body">
      <!-- NEW =========================================================================================-->
      <div class="history-container">
        <div v-for="log in historyList" :key="log.id" class="history-item">
          <div :class="['message', 'user-message']">
            <p><strong>User：</strong> {{ log.userQuery }}</p>
          </div>
          <!-- 历史记录的LLM消息 -->
          <div :class="['message', 'bot-message']">
            <p><strong>AT：</strong> {{ log.llmResponse }}</p>
          </div>
        </div>
      </div>
      <div class="tableLine"><span class="midText">历史信息</span></div> <!--  分割线  -->
      <!-- NEW END =====================================================================================-->
      <div id="messages">
        <div v-for="message in messages" :key="message.id" :class="['message', message.type]">
          {{ message.text }}
        </div>
      </div>
      <div class="input-area">
        <input v-model="userInput" type="text" id="user-input" placeholder="Type a message..." @keypress.enter="sendMessage">
        <button @click="sendMessage" id="send-button">Send</button>
        <button @click="clearMessages" id="clean-button">Clean</button>
      </div>
    </div>
  </div>
</template>

<script>
import userApi from '@/api/user'
import axios from 'axios'
import Vue from 'vue'
Vue.prototype.$axios = axios

export default {
  data () {
    return {
      userid: null, // 用于存储用户ID
      userInput: '',
      messages: [],
      isMinimized: false,
      historyList: [] // 存储历史交互信息
    }
  },
  // NEW ============================================================================
  mounted () {
    // this.loadUserHistory() // mounted函数：在组件加载的时候调用loadUserHistory函数获取历史交互
    this.getUserId()
    // this.loadUserHistory()
  },
  // NEW END=========================================================================
  methods: {
    // NEW ===========================================================================
    // 获取userid
    async getUserId () {
      userApi.getCurrentUser()
        .then(response => {
          if (response && response.response && response.response.id) {
            this.userid = response.response.id // 保存用户ID
            console.log('User ID1:', this.userid) // 输出查看用户ID
            this.loadUserHistory() // 在成功获取用户ID后调用 loadHistory()
          } else {
            console.error('无法获取用户ID')
          }
        })
        .catch(error => {
          console.error('获取用户信息时发生错误:', error)
        })
    },
    // NEW ===================================================================
    // 加载历史交互信息
    async loadUserHistory () {
      try {
        console.log('开始加载!!!!')

        const userId = this.userid // 假设`userid`已经通过其他方式获取并存储在data中
        console.log('User ID:', userId)

        if (userId) {
          const response = await this.$axios.get(`/api/chat/history/${userId}`)
          this.historyList = response.data // 将返回的数据保存到data中
          console.log('历史交互信息加载成功:', this.historyList)
          console.log('历史数据:', response.data)
        } else {
          console.error('无法加载历史交互信息，用户ID未定义')
        }
      } catch (error) {
        console.error('加载历史交互信息时发生错误:', error)
      }
    },
    // NEW END========================================================================
    async sendMessage () {
      if (this.userInput.trim() === '') return

      // Append the user's message to the messages array
      this.messages.push({ id: Date.now(), text: this.userInput, type: 'user-message' })
      const userMessage = this.userInput
      this.userInput = ''
      // Call the function to get the bot's response
      await this.getBotResponse(userMessage)
    },
    async getBotResponse (userInput) {
      try {
        const response = await fetch('http://localhost:8000/api/llm-response', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json; charset=UTF-8'
          },
          // body: JSON.stringify({ message: userInput })
          body: JSON.stringify({
            prompt: userInput,
            userid: this.userid // 这个假设是 userid
          })
        })
        // const data = await response.json() // 当后端返回为JSON字符串的时候要用.json()来解析
        const data = await response.text() // 当后端直接返回JSON字符串中content内容时，使用.text()来解析
        // const data = await response;
        // const botResponse = data.choices[0].message.content // 后端返回的是JSON字符串，需要提取consent
        const botResponse = data // 当后端返回的为content
        // Append the bot's response to the messages array 'http://localhost:8003/api/llm-response'
        this.messages.push({ id: Date.now(), text: botResponse, type: 'bot-message' })
        // this.messages.push({ id: Date.now(), text: data, type: 'bot-message' });
      } catch (error) {
        console.error('Error fetching the bot response:', error)

        // Append an error message to the messages array
        this.messages.push({ id: Date.now(), text: 'Error fetching the bot response.', type: 'bot-message' })
      }
    },
    toggleMinimize () {
      this.isMinimized = !this.isMinimized
    },
    clearMessages () {
      this.messages = []
    }
  }
}
</script>

<style>
.chat-window {
  position: fixed;
  bottom: 20px;
  right: 20px;
  width: 350px;
  height: 350px;
  border: 1px solid #ccc;
  padding: 10px;
  background-color: white;
  display: flex;
  flex-direction: column;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  z-index: 1000;
}

/* NEW ==========================================*/
.history-container {
  flex: 1;
  margin: 5px 0;
  justify-content: space-between;
}
.history-item .user-message, .history-item .bot-message {
  margin: 5px 0;
  //padding: 5px;
  padding-top: 1px;
  padding-bottom: 1px;
  padding-left: 8px;
  border-radius: 5px;
  //max-width: 100%;
}
/* 分割线 */
.tableLine {
  position: relative;
  margin: 15px auto;
  width: 300px;
  height: 1px;
  background-color: #d4d4d4;
  text-align: center;
  font-size: 16px;
  color: rgba(101, 101, 101, 1);
}
.midText { /*分割线上的文字格式*/
  position: absolute;
  left: 50%;
  background-color: #ffffff;
  padding: 0 3px;
  transform: translateX(-50%) translateY(-50%);
}

/* NEW END=======================================*/
.minimized {
  height: 40px;
}

.chat-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  cursor: pointer;
  border-bottom: 1px solid #ccc;
  padding-bottom: 5px;
  background-color: #f0f0f0;
}

.chat-body {
  flex: 1;
  overflow-y: auto;
  margin-top: 10px;
}

#messages {
  flex: 1;
  overflow-y: auto;
  margin-bottom: 10px;
}

.message {
  margin: 5px 0;
  padding: 10px;
  border-radius: 5px;
}

.user-message {
  background-color: #d1e7dd;
  align-self: flex-end;
}

.bot-message {
  background-color: #f8d7da;
  align-self: flex-start;
}

.input-area {
  display: flex;
  justify-content: space-between;
}

#user-input {
  flex: 1;
  margin-right: 10px;
}

#send-button, #clean-button {
  width: 60px;
}
</style>
