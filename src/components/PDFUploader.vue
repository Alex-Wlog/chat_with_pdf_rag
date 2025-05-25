<script setup>
import { ref, onMounted } from 'vue'
import * as pdfjsLib from 'pdfjs-dist'

// 初始化 PDF.js worker
pdfjsLib.GlobalWorkerOptions.workerSrc = `https://cdnjs.cloudflare.com/ajax/libs/pdf.js/${pdfjsLib.version}/pdf.worker.min.js`

const props = defineProps({
  onFileProcessed: {
    type: Function,
    required: true
  }
})

const uploadedFiles = ref([])
const isProcessing = ref(false)
const errorMessage = ref('')

// 处理文件上传
const handleFileUpload = async (event) => {
  const file = event.target.files[0]
  if (!file) return
  
  if (file.type !== 'application/pdf') {
    errorMessage.value = '请上传 PDF 文件'
    return
  }

  try {
    isProcessing.value = true
    errorMessage.value = ''
    
    const fileData = {
      file,
      name: file.name,
      size: (file.size / 1024 / 1024).toFixed(2) + 'MB',
      pages: await getPageCount(file),
      status: 'processing'
    }
    
    uploadedFiles.value.push(fileData)
    
    // 读取 PDF 文本内容
    const text = await extractTextFromPDF(file)
    fileData.status = 'completed'
    
    // 调用父组件的回调函数
    props.onFileProcessed({
      name: file.name,
      text: text
    })
    
  } catch (error) {
    console.error('PDF 处理错误:', error)
    errorMessage.value = '处理 PDF 文件时出错'
  } finally {
    isProcessing.value = false
  }
}

// 获取 PDF 页数
const getPageCount = async (file) => {
  const arrayBuffer = await file.arrayBuffer()
  const pdf = await pdfjsLib.getDocument(arrayBuffer).promise
  return pdf.numPages
}

// 提取 PDF 文本
const extractTextFromPDF = async (file) => {
  const arrayBuffer = await file.arrayBuffer()
  const pdf = await pdfjsLib.getDocument(arrayBuffer).promise
  let text = ''
  
  for (let i = 1; i <= pdf.numPages; i++) {
    const page = await pdf.getPage(i)
    const content = await page.getTextContent()
    text += content.items.map(item => item.str).join(' ') + '\n'
  }
  
  return text
}

// 触发文件选择
const triggerFileInput = () => {
  document.getElementById('pdf-file-input').click()
}

// 删除文件
const removeFile = (index) => {
  uploadedFiles.value.splice(index, 1)
}
</script>

<template>
  <div class="pdf-uploader">
    <input
      type="file"
      id="pdf-file-input"
      accept=".pdf"
      @change="handleFileUpload"
      class="hidden-input"
    />
    
    <div class="upload-area" @click="triggerFileInput">
      <div class="upload-icon">📄</div>
      <div class="upload-text">
        <span v-if="!isProcessing">点击上传 PDF 文件</span>
        <span v-else>正在处理文件...</span>
      </div>
      <div class="upload-hint">支持单个 PDF 文件上传</div>
    </div>
    
    <div v-if="errorMessage" class="error-message">
      {{ errorMessage }}
    </div>
    
    <div class="file-list">
      <div v-for="(file, index) in uploadedFiles" :key="index" class="file-item">
        <div class="file-info">
          <div class="file-name">{{ file.name }}</div>
          <div class="file-meta">
            <span>{{ file.size }}</span>
            <span>{{ file.pages }} 页</span>
          </div>
        </div>
        <div class="file-status">
          <span v-if="file.status === 'processing'" class="status processing">处理中...</span>
          <span v-else-if="file.status === 'completed'" class="status completed">已完成</span>
        </div>
        <button class="remove-button" @click="removeFile(index)">删除</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.pdf-uploader {
  width: 100%;
}

.hidden-input {
  display: none;
}

.upload-area {
  border: 2px dashed var(--border-color);
  padding: 2rem 1rem;
  text-align: center;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.upload-area:hover {
  border-color: var(--primary-color);
  background-color: var(--secondary-color);
}

.upload-icon {
  font-size: 2rem;
  margin-bottom: 0.5rem;
}

.upload-text {
  font-size: 1rem;
  color: var(--text-color);
  margin-bottom: 0.5rem;
}

.upload-hint {
  font-size: 0.8rem;
  color: #666;
}

.error-message {
  color: #dc3545;
  font-size: 0.875rem;
  margin-top: 0.5rem;
}

.file-list {
  margin-top: 1rem;
}

.file-item {
  display: flex;
  align-items: center;
  padding: 0.75rem;
  background-color: #f8f9fa;
  border-radius: 4px;
  margin-bottom: 0.5rem;
}

.file-info {
  flex: 1;
}

.file-name {
  font-size: 0.9rem;
  margin-bottom: 0.25rem;
  color: var(--text-color);
}

.file-meta {
  font-size: 0.8rem;
  color: #666;
}

.file-meta span:not(:last-child) {
  margin-right: 1rem;
}

.file-status {
  margin: 0 1rem;
}

.status {
  font-size: 0.8rem;
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
}

.status.processing {
  background-color: #fff3cd;
  color: #856404;
}

.status.completed {
  background-color: #d4edda;
  color: #155724;
}

.remove-button {
  padding: 0.25rem 0.5rem;
  background-color: transparent;
  border: 1px solid #dc3545;
  color: #dc3545;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.8rem;
  transition: all 0.3s ease;
}

.remove-button:hover {
  background-color: #dc3545;
  color: white;
}
</style> 