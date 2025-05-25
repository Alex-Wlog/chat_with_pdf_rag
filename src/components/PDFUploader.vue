<script setup>
import { ref } from 'vue'

const props = defineProps({
  onFileProcessed: {
    type: Function,
    required: true
  }
})

const uploadedFiles = ref([])
const isProcessing = ref(false)
const errorMessage = ref('')
const showMethodDialog = ref(false)
const currentFile = ref(null)

const parseOptions = [
  { id: 'direct', name: '直接解析', description: '直接提取 PDF 文本内容，适合文本较为规整的文档' },
  { id: 'structured', name: '结构化解析', description: '识别文档结构（标题、段落等），适合格式规范的报告' },
  { id: 'semantic', name: '语义解析', description: '深度语义分析，提取关键信息和上下文关系' }
]

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
      status: 'uploaded'
    }
    
    uploadedFiles.value = [fileData] // 只保留最新上传的文件
    currentFile.value = fileData
    showMethodDialog.value = true
    
  } catch (error) {
    console.error('文件处理错误:', error)
    errorMessage.value = '文件处理出错'
  } finally {
    isProcessing.value = false
  }
}

// 选择解析方法
const selectParseMethod = async (methodId) => {
  if (!currentFile.value) return
  
  try {
    isProcessing.value = true
    currentFile.value.status = 'processing'
    showMethodDialog.value = false
    
    // 创建 FormData 对象
    const formData = new FormData()
    formData.append('file', currentFile.value.file)
    formData.append('method', methodId)
    
    // TODO: 调用后端 API 进行文件解析
    // const response = await fetch('/api/parse-pdf', {
    //   method: 'POST',
    //   body: formData
    // })
    // const result = await response.json()
    
    // 模拟 API 调用
    await new Promise(resolve => setTimeout(resolve, 1500))
    
    currentFile.value.status = 'completed'
    
    // 调用父组件回调
    props.onFileProcessed({
      name: currentFile.value.name,
      method: methodId,
      status: 'success'
    })
    
  } catch (error) {
    console.error('解析错误:', error)
    currentFile.value.status = 'error'
    errorMessage.value = '文件解析失败'
  } finally {
    isProcessing.value = false
  }
}

// 触发文件选择
const triggerFileInput = () => {
  document.getElementById('pdf-file-input').click()
}

// 删除文件
const removeFile = (index) => {
  uploadedFiles.value.splice(index, 1)
  currentFile.value = null
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
    
    <div class="upload-area" @click="triggerFileInput" v-if="!uploadedFiles.length">
      <div class="upload-icon">📄</div>
      <div class="upload-text">
        <span v-if="!isProcessing">点击上传 PDF 文件</span>
        <span v-else>正在处理文件...</span>
      </div>
      <div class="upload-hint">支持 PDF 文件上传，建议大小不超过 20MB</div>
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
          </div>
        </div>
        <div class="file-status">
          <span v-if="file.status === 'uploaded'" class="status uploaded">待解析</span>
          <span v-else-if="file.status === 'processing'" class="status processing">解析中...</span>
          <span v-else-if="file.status === 'completed'" class="status completed">已完成</span>
          <span v-else-if="file.status === 'error'" class="status error">解析失败</span>
        </div>
        <button class="remove-button" @click="removeFile(index)">删除</button>
      </div>
    </div>

    <!-- 解析方法选择对话框 -->
    <div class="dialog-overlay" v-if="showMethodDialog">
      <div class="dialog">
        <h3 class="dialog-title">选择解析方法</h3>
        <div class="dialog-content">
          <div 
            v-for="option in parseOptions" 
            :key="option.id"
            class="parse-option"
            @click="selectParseMethod(option.id)"
          >
            <h4>{{ option.name }}</h4>
            <p>{{ option.description }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.pdf-uploader {
  width: 100%;
  position: relative;
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

.file-status {
  margin: 0 1rem;
}

.status {
  font-size: 0.8rem;
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
}

.status.uploaded {
  background-color: #e9ecef;
  color: #495057;
}

.status.processing {
  background-color: #fff3cd;
  color: #856404;
}

.status.completed {
  background-color: #d4edda;
  color: #155724;
}

.status.error {
  background-color: #f8d7da;
  color: #721c24;
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

/* 对话框样式 */
.dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.dialog {
  background: white;
  border-radius: 8px;
  padding: 1.5rem;
  width: 90%;
  max-width: 500px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.dialog-title {
  margin: 0 0 1rem 0;
  font-size: 1.2rem;
  color: var(--text-color);
}

.dialog-content {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.parse-option {
  padding: 1rem;
  border: 1px solid var(--border-color);
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.parse-option:hover {
  background-color: var(--secondary-color);
  border-color: var(--primary-color);
}

.parse-option h4 {
  margin: 0 0 0.5rem 0;
  color: var(--text-color);
}

.parse-option p {
  margin: 0;
  font-size: 0.9rem;
  color: #666;
}
</style> 