<script setup>
import { ref, onMounted, nextTick } from 'vue'
import { marked } from 'marked'
import hljs from 'highlight.js'
import 'highlight.js/styles/github.css'
import dataWhaleLogo from './assets/datawhale-logo.svg'
import userAvatar from './assets/user-avatar.svg'
import botAvatar from './assets/bot-avatar.svg'

// 配置 marked
marked.setOptions({
  highlight: function(code, lang) {
    if (lang && hljs.getLanguage(lang)) {
      return hljs.highlight(code, { language: lang }).value
    }
    return hljs.highlightAuto(code).value
  },
  breaks: true
})

const messages = ref([])
const userInput = ref('')
const chatContainer = ref(null)
const isLoading = ref(false)
const errorMessage = ref('')

// 渲染消息（支持 Markdown）
const renderMessage = (content) => {
  return marked(content)
}

// 滚动到底部
const scrollToBottom = () => {
  if (chatContainer.value) {
    chatContainer.value.scrollTop = chatContainer.value.scrollHeight
  }
}

// 清空对话
const clearChat = () => {
  messages.value = []
}

// 发送消息
const sendMessage = async () => {
  if (!userInput.value.trim() || isLoading.value) return
  
  // 添加用户消息
  messages.value.push({
    type: 'user',
    content: userInput.value,
    timestamp: new Date().toLocaleTimeString()
  })
  
  const userMessage = userInput.value
  userInput.value = ''
  errorMessage.value = ''
  isLoading.value = true
  
  // 滚动到底部
  await nextTick()
  scrollToBottom()
  
  try {
    // TODO: 替换为实际的 API 调用
    const response = await new Promise((resolve) => {
      setTimeout(() => {
        resolve({
          content: `我收到了你的消息：${userMessage}\n\n这是一个示例回复，支持 **Markdown** 格式。\n\n\`\`\`python\nprint("Hello World!")\n\`\`\``
        })
      }, 1000)
    })
    
    messages.value.push({
      type: 'bot',
      content: response.content,
      timestamp: new Date().toLocaleTimeString()
    })
  } catch (error) {
    errorMessage.value = '发送消息失败，请重试'
    console.error('Error:', error)
  } finally {
    isLoading.value = false
    nextTick(() => scrollToBottom())
  }
}

onMounted(() => {
  // 添加欢迎消息
  messages.value.push({
    type: 'bot',
    content: '欢迎使用 Datawhale 聊天助手！我可以帮助您回答问题。',
    timestamp: new Date().toLocaleTimeString()
  })
})
</script>

<template>
  <div class="app-container">
    <header class="header">
      <div class="logo-container">
        <img :src="dataWhaleLogo" alt="Datawhale Logo" class="logo" />
        <h1 class="title">Datawhale 智能助手</h1>
      </div>
    </header>
    
    <main class="main-content">
      <div class="chat-container" ref="chatContainer">
        <div v-for="(message, index) in messages" :key="index" class="message" :class="message.type">
          <div class="message-avatar">
            <img :src="message.type === 'user' ? userAvatar : botAvatar" :alt="message.type">
          </div>
          <div class="message-wrapper">
            <div class="message-header">
              <span class="message-sender">{{ message.type === 'user' ? '你' : 'AI 助手' }}</span>
              <span class="message-time">{{ message.timestamp }}</span>
            </div>
            <div class="message-content" v-html="renderMessage(message.content)"></div>
          </div>
        </div>
        <div v-if="isLoading" class="message bot">
          <div class="message-avatar">
            <img :src="botAvatar" alt="bot">
          </div>
          <div class="message-content loading">
            <div class="typing-indicator">
              <span></span>
              <span></span>
              <span></span>
            </div>
          </div>
        </div>
      </div>
      
      <div class="input-container">
        <div class="input-wrapper">
          <textarea 
            v-model="userInput" 
            @keyup.enter.exact="sendMessage"
            @keydown.enter.exact.prevent
            placeholder="输入您的问题..."
            rows="1"
            class="input-field"
          ></textarea>
          <div v-if="errorMessage" class="error-message">{{ errorMessage }}</div>
        </div>
        <button 
          @click="sendMessage" 
          class="send-button"
          :disabled="isLoading || !userInput.trim()"
        >
          <span v-if="!isLoading">发送</span>
          <span v-else>发送中...</span>
        </button>
        <button @click="clearChat" class="clear-button">清空对话</button>
      </div>
    </main>
  </div>
</template>

<style>
:root {
  --primary-color: #1976d2;
  --secondary-color: #e3f2fd;
  --background-color: #f5f5f5;
  --text-color: #333333;
  --border-color: #dddddd;
}

body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  color: var(--text-color);
}

.app-container {
  height: 100vh;
  display: flex;
  flex-direction: column;
}

.header {
  background-color: #ffffff;
  padding: 1rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.logo-container {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
}

.logo {
  height: 40px;
}

.title {
  margin: 0;
  font-size: 1.5rem;
  color: var(--primary-color);
}

.main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: 1rem;
  background-color: var(--background-color);
  gap: 1rem;
}

.chat-container {
  flex: 1;
  overflow-y: auto;
  padding: 1rem;
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.message {
  display: flex;
  margin-bottom: 1.5rem;
  align-items: flex-start;
  animation: fadeIn 0.3s ease-in-out;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.message-avatar {
  width: 40px;
  height: 40px;
  margin-right: 1rem;
  flex-shrink: 0;
}

.message-avatar img {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.message-wrapper {
  flex: 1;
}

.message-header {
  margin-bottom: 0.5rem;
  font-size: 0.9rem;
  color: #666;
}

.message-time {
  margin-left: 0.5rem;
  font-size: 0.8rem;
  color: #999;
}

.message-content {
  padding: 1rem;
  border-radius: 8px;
  background-color: var(--background-color);
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
}

.message.user .message-content {
  background-color: var(--secondary-color);
}

.message-content :deep(pre) {
  background-color: #f8f9fa;
  padding: 1rem;
  border-radius: 4px;
  overflow-x: auto;
}

.message-content :deep(code) {
  font-family: 'Fira Code', monospace;
  font-size: 0.9em;
}

.input-container {
  display: flex;
  gap: 1rem;
  padding: 1rem;
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.input-wrapper {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.input-field {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid var(--border-color);
  border-radius: 4px;
  resize: none;
  font-family: inherit;
  font-size: 1rem;
  transition: border-color 0.3s ease;
}

.input-field:focus {
  outline: none;
  border-color: var(--primary-color);
}

.error-message {
  color: #dc3545;
  font-size: 0.875rem;
  margin-top: 0.5rem;
}

.send-button, .clear-button {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 4px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
}

.send-button {
  background-color: var(--primary-color);
  color: white;
}

.send-button:hover:not(:disabled) {
  background-color: #1565c0;
}

.send-button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.clear-button {
  background-color: #f5f5f5;
  color: #666;
}

.clear-button:hover {
  background-color: #e0e0e0;
}

.typing-indicator {
  display: flex;
  gap: 0.5rem;
  padding: 0.5rem;
}

.typing-indicator span {
  width: 8px;
  height: 8px;
  background-color: #999;
  border-radius: 50%;
  animation: bounce 1.4s infinite ease-in-out;
}

.typing-indicator span:nth-child(1) { animation-delay: -0.32s; }
.typing-indicator span:nth-child(2) { animation-delay: -0.16s; }

@keyframes bounce {
  0%, 80%, 100% { transform: scale(0); }
  40% { transform: scale(1); }
}

/* 响应式设计 */
@media (max-width: 768px) {
  .header {
    padding: 0.5rem;
  }
  
  .logo {
    height: 32px;
  }
  
  .title {
    font-size: 1.2rem;
  }
  
  .main-content {
    padding: 0.5rem;
  }
  
  .input-container {
    flex-direction: column;
  }
  
  .send-button, .clear-button {
    width: 100%;
  }
}
</style>
