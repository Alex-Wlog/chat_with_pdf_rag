<script setup>
import { ref, onMounted, nextTick } from 'vue'
import { marked } from 'marked'
import hljs from 'highlight.js'
import 'highlight.js/styles/github.css'
import dataWhaleLogo from './assets/icon_whitebg.jpg'
import userAvatar from './assets/user-avatar.svg'
import botAvatar from './assets/bot-avatar.svg'
import PDFUploader from './components/PDFUploader.vue'

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
const selectedModel = ref('gpt-3.5-turbo')
const uploadedPDFContent = ref('')

// 处理 PDF 文件上传完成
const handlePDFProcessed = (fileData) => {
  uploadedPDFContent.value = fileData.text
  messages.value.push({
    type: 'system',
    content: `已成功上传并处理文件：${fileData.name}`,
    timestamp: new Date().toLocaleTimeString()
  })
}

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
  // 添加欢迎消息
  messages.value.push({
    type: 'bot',
    content: '欢迎使用 金融研报智能问答系统！我可以帮助您回答问题。',
    timestamp: new Date().toLocaleTimeString()
  })
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
    content: '欢迎使用 金融研报智能问答系统！我可以帮助您回答问题。',
    timestamp: new Date().toLocaleTimeString()
  })
})
</script>

<template>
  <div class="app-container">
    <header class="header">
      <div class="logo-container">
        <img :src="dataWhaleLogo" alt="Datawhale Logo" class="logo" />
        <h1 class="title">基于大语言模型的金融研报年报智能问答系统</h1>
      </div>
    </header>
    
    <main class="main-content">
      <!-- 左侧面板 -->
      <div class="left-panel">
        <div class="panel-header">
          <h2>知识库配置</h2>
        </div>
        <div class="panel-content">
          <div class="config-section">
            <h3>模型选择</h3>
            <select v-model="selectedModel" class="select-input">
              <option value="gpt-3.5-turbo">GPT-3.5-Turbo</option>
              <option value="deepseek">DeepSeek</option>
              <option value="other">其他模型</option>
            </select>
          </div>
          <div class="config-section">
            <h3>文档上传</h3>
            <PDFUploader :onFileProcessed="handlePDFProcessed" />
          </div>
        </div>
      </div>

      <!-- 右侧聊天区域 -->
      <div class="right-panel">
        <div class="chat-container" ref="chatContainer">
          <div v-for="(message, index) in messages" :key="index" 
               class="message" :class="message.type">
            <div class="message-avatar">
              <img :src="message.type === 'user' ? userAvatar : botAvatar" :alt="message.type">
            </div>
            <div class="message-wrapper">
              <div class="message-header">
                <span class="message-sender">
                  {{ message.type === 'user' ? '用户' : message.type === 'system' ? '系统' : 'AI 助手' }}
                </span>
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
              placeholder="请输入您的问题..."
              rows="3"
              class="input-field"
            ></textarea>
            <div v-if="errorMessage" class="error-message">{{ errorMessage }}</div>
          </div>
          <div class="button-group">
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
        </div>
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
  --panel-width: 300px;
}

body {
  margin: 0;
  font-family: "Times New Roman", SimSun, serif;
  color: var(--text-color);
  background-color: var(--background-color);
}

.app-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.header {
  background-color: #ffffff;
  padding: 1rem 2rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  z-index: 100;
}

.logo-container {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1.5rem;
  max-width: 1200px;
  margin: 0 auto;
  position: relative;
}

.logo {
  position: absolute;
  left: 0;
  height: 50px;
  width: auto;
}

.title {
  margin: 0;
  font-size: 1.8rem;
  color: var(--text-color);
  font-weight: bold;
  text-align: center;
  flex: 1;
  padding: 0 60px; /* 为logo预留空间 */
}

.main-content {
  flex: 1;
  display: flex;
  max-width: 1400px;
  margin: 0 auto;
  padding: 2rem;
  gap: 2rem;
  min-height: calc(100vh - 90px);
  overflow: hidden; /* 防止内部溢出 */
}

/* 左侧面板样式 */
.left-panel {
  width: var(--panel-width);
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  display: flex;
  flex-direction: column;
  max-height: calc(100vh - 130px); /* 设置最大高度 */
  overflow: hidden; /* 防止内部溢出 */
}

.panel-header {
  padding: 1rem;
  border-bottom: 1px solid var(--border-color);
}

.panel-header h2 {
  margin: 0;
  font-size: 1.2rem;
  color: var(--text-color);
}

.panel-content {
  padding: 1rem;
  flex: 1;
  overflow-y: auto; /* 内部可滚动 */
}

.config-section {
  margin-bottom: 2rem;
}

.config-section h3 {
  margin: 0 0 1rem 0;
  font-size: 1rem;
  color: var(--text-color);
}

.select-input {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid var(--border-color);
  border-radius: 4px;
  font-size: 0.9rem;
}

.upload-area {
  border: 2px dashed var(--border-color);
  padding: 1rem;
  text-align: center;
  border-radius: 4px;
  margin-bottom: 1rem;
}

.file-input {
  display: none;
}

.upload-button {
  background-color: var(--secondary-color);
  color: var(--primary-color);
  border: 1px solid var(--primary-color);
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.9rem;
}

/* 右侧聊天区域样式 */
.right-panel {
  flex: 1;
  display: flex;
  flex-direction: column;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  max-height: calc(100vh - 130px); /* 设置最大高度 */
  overflow: hidden; /* 防止内部溢出 */
}

.chat-container {
  flex: 1;
  overflow-y: auto; /* 聊天历史可滚动 */
  padding: 1.5rem;
  margin-bottom: 0;
  /* 自定义滚动条样式 */
  scrollbar-width: thin;
  scrollbar-color: rgba(0, 0, 0, 0.2) transparent;
}

/* Webkit浏览器的滚动条样式 */
.chat-container::-webkit-scrollbar {
  width: 6px;
}

.chat-container::-webkit-scrollbar-track {
  background: transparent;
}

.chat-container::-webkit-scrollbar-thumb {
  background-color: rgba(0, 0, 0, 0.2);
  border-radius: 3px;
}

.chat-container::-webkit-scrollbar-thumb:hover {
  background-color: rgba(0, 0, 0, 0.3);
}

.message {
  display: flex;
  margin-bottom: 1.5rem;
  align-items: flex-start;
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
}

.message-wrapper {
  flex: 1;
  max-width: 80%;
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
  font-size: 1rem;
  line-height: 1.6;
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
  font-family: "Consolas", "Microsoft YaHei", monospace;
  font-size: 0.9em;
}

.input-container {
  padding: 1rem;
  border-top: 1px solid var(--border-color);
  background: white;
  position: relative;
  z-index: 1;
  margin: 0 1rem 1rem 1rem; /* 添加左右边距 */
}

.input-wrapper {
  margin-bottom: 1rem;
  position: relative; /* 添加相对定位 */
}

.input-field {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid var(--border-color);
  border-radius: 4px;
  resize: vertical;
  font-family: inherit;
  font-size: 1rem;
  line-height: 1.5;
  min-height: 60px;
  max-height: 150px;
  overflow-y: auto;
  box-sizing: border-box; /* 确保padding不会增加宽度 */
}

.input-field:focus {
  outline: none;
  border-color: var(--primary-color);
}

.button-group {
  display: flex;
  gap: 1rem;
  justify-content: flex-end;
  padding: 0 1rem; /* 添加左右内边距 */
}

.send-button, .clear-button {
  padding: 0.75rem 2rem;
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

/* 添加系统消息样式 */
.message.system .message-content {
  background-color: #e8eaf6;
  color: #3f51b5;
}

.message.system .message-sender {
  color: #3f51b5;
}
</style>
