<!--&lt;!&ndash;-->
<!--  模板部分:-->
<!--    * 添加了 Clean 按钮，点击时调用 clearMessages 方法。-->
<!--    * 输入框和按钮放在一个 div 容器中，使用类名 input-area 进行样式布局。-->
<!--  脚本部分:-->
<!--    * sendMessage 方法：当用户输入消息后，首先将消息添加到 messages 数组中，然后调用 getBotResponse 方法获取 LLM 响应。-->
<!--    * 添加了 clearMessages 方法，该方法通过将 messages 数组清空来清除消息。-->
<!--    * getBotResponse 方法：发送 POST 请求到 /api/llm-response，并将 LLM 的响应添加到 messages 数组中。-->
<!--    * toggleMinimize 方法：切换窗口的最小化状态。-->
<!--  样式部分:-->
<!--    * 添加了 .input-area 类，以便将输入框和按钮放在一行，并确保它们之间有适当的间距。-->
<!--&ndash;&gt;-->

<!--<template>-->
<!--  <div :class="['chat-window', { minimized: isMinimized }]">-->
<!--    <div class="chat-header" @click="toggleMinimize">-->
<!--      <span>Chat AT</span>-->
<!--    </div>-->
<!--    <div v-if="!isMinimized" class="chat-body">-->
<!--      <div id="messages">-->
<!--        <div v-for="message in messages" :key="message.id" :class="['message', message.type]">-->
<!--          {{ message.text }}-->
<!--        </div>-->
<!--      </div>-->
<!--      <div class="input-area">-->
<!--        <input v-model="userInput" type="text" id="user-input" placeholder="Type a message..." @keypress.enter="sendMessage">-->
<!--        <button @click="sendMessage" id="send-button">Send</button>-->
<!--        <button @click="clearMessages" id="clean-button">Clean</button>-->
<!--      </div>-->
<!--    </div>-->
<!--  </div>-->
<!--</template>-->

<!--<script>-->
<!--export default {-->
<!--  data () {-->
<!--    return {-->
<!--      userInput: '',-->
<!--      messages: [],-->
<!--      isMinimized: false-->
<!--    }-->
<!--  },-->
<!--  methods: {-->
<!--    async sendMessage () {-->
<!--      if (this.userInput.trim() === '') return-->

<!--      // Append the user's message to the messages array-->
<!--      this.messages.push({ id: Date.now(), text: this.userInput, type: 'user-message' })-->
<!--      const userMessage = this.userInput-->
<!--      this.userInput = ''-->
<!--      // console.log(this.messages)-->
<!--      // Call the function to get the bot's response-->
<!--      await this.getBotResponse(userMessage)-->
<!--    },-->
<!--    async getBotResponse (userInput) {-->
<!--      try {-->
<!--        const response = await fetch('http://localhost:8000/api/llm-response', {-->
<!--          method: 'POST',-->
<!--          headers: {-->
<!--            'Content-Type': 'application/json; charset=UTF-8'-->
<!--          },-->
<!--          // body: JSON.stringify({ message: userInput })-->
<!--          body: JSON.stringify(userInput)-->
<!--        })-->
<!--        // const data = await response.json() // 当后端返回为JSON字符串的时候要用.json()来解析-->
<!--        const data = await response.text() // 当后端直接返回JSON字符串中content内容时，使用.text()来解析-->
<!--        // const data = await response;-->
<!--        // const botResponse = data.choices[0].message.content // 后端返回的是JSON字符串，需要提取consent-->
<!--        const botResponse = data // 当后端返回的为content-->
<!--        // Append the bot's response to the messages array 'http://localhost:8003/api/llm-response'-->
<!--        this.messages.push({ id: Date.now(), text: botResponse, type: 'bot-message' })-->
<!--        // this.messages.push({ id: Date.now(), text: data, type: 'bot-message' });-->
<!--      } catch (error) {-->
<!--        console.error('Error fetching the bot response:', error)-->

<!--        // Append an error message to the messages array-->
<!--        this.messages.push({ id: Date.now(), text: 'Error fetching the bot response.', type: 'bot-message' })-->
<!--      }-->
<!--    },-->
<!--    toggleMinimize () {-->
<!--      this.isMinimized = !this.isMinimized-->
<!--    },-->
<!--    clearMessages () {-->
<!--      this.messages = []-->
<!--    }-->
<!--  }-->
<!--}-->
<!--</script>-->

<!--<style>-->
<!--.chat-window {-->
<!--  position: fixed;-->
<!--  bottom: 20px;-->
<!--  right: 20px;-->
<!--  width: 350px;-->
<!--  height: 350px;-->
<!--  border: 1px solid #ccc;-->
<!--  padding: 10px;-->
<!--  background-color: white;-->
<!--  display: flex;-->
<!--  flex-direction: column;-->
<!--  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);-->
<!--  z-index: 1000;-->
<!--}-->

<!--.minimized {-->
<!--  height: 40px;-->
<!--}-->

<!--.chat-header {-->
<!--  display: flex;-->
<!--  justify-content: space-between;-->
<!--  align-items: center;-->
<!--  cursor: pointer;-->
<!--  border-bottom: 1px solid #ccc;-->
<!--  padding-bottom: 5px;-->
<!--  background-color: #f0f0f0;-->
<!--}-->

<!--.chat-body {-->
<!--  flex: 1;-->
<!--  overflow-y: auto;-->
<!--  margin-top: 10px;-->
<!--}-->

<!--#messages {-->
<!--  flex: 1;-->
<!--  overflow-y: auto;-->
<!--  margin-bottom: 10px;-->
<!--}-->

<!--.message {-->
<!--  margin: 5px 0;-->
<!--  padding: 10px;-->
<!--  border-radius: 5px;-->
<!--}-->

<!--.user-message {-->
<!--  background-color: #d1e7dd;-->
<!--  align-self: flex-end;-->
<!--}-->

<!--.bot-message {-->
<!--  background-color: #f8d7da;-->
<!--  align-self: flex-start;-->
<!--}-->

<!--.input-area {-->
<!--  display: flex;-->
<!--  justify-content: space-between;-->
<!--}-->

<!--#user-input {-->
<!--  flex: 1;-->
<!--  margin-right: 10px;-->
<!--}-->

<!--#send-button, #clean-button {-->
<!--  width: 60px;-->
<!--}-->
<!--</style>-->
