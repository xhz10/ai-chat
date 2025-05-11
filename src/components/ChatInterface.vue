<script setup>
import { ref } from 'vue'

const username = ref('')
const role = ref('')
const inputDialog = ref('')
const messages = ref([])
const isLoading = ref(false)

const sendMessage = async () => {
  if (!username.value || !role.value || !inputDialog.value) {
    alert('请填写完整信息！')
    return
  }

  // 添加用户消息到聊天记录
  messages.value.push({
    type: 'user',
    content: inputDialog.value
  })

  isLoading.value = true

  try {
    const response = await fetch('http://localhost:9265/role-play-reply', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        username: username.value,
        role: role.value,
        input_dialog: inputDialog.value
      })
    })

    const data = await response.json()
    let reply = data.reply || data.res || '抱歉，我现在无法回答。'
    // 如果reply是带引号的字符串，自动去除
    if (typeof reply === 'string' && reply.startsWith('"') && reply.endsWith('"')) {
      try {
        reply = JSON.parse(reply)
      } catch (e) {
        // 解析失败就原样展示
      }
    }
    messages.value.push({
      type: 'ai',
      content: reply
    })
  } catch (error) {
    console.error('Error:', error)
    messages.value.push({
      type: 'ai',
      content: '抱歉，发生了错误，请稍后重试。'
    })
  } finally {
    isLoading.value = false
    inputDialog.value = ''
  }
}
</script>

<template>
  <div class="chat-container">
    <div class="chat-header">
      <h1>AI 角色扮演</h1>
      <p class="subtitle">与 AI 展开一场有趣的对话</p>
    </div>

    <div class="settings-panel">
      <div class="input-group">
        <label>你的名字</label>
        <input v-model="username" placeholder="请输入你的名字" />
      </div>
      <div class="input-group">
        <label>AI 角色</label>
        <input v-model="role" placeholder="请输入 AI 角色" />
      </div>
    </div>

    <div class="chat-messages" ref="messagesContainer">
      <div v-for="(message, index) in messages" :key="index" 
           :class="['message', message.type]">
        <div class="message-content">
          {{ message.content }}
        </div>
      </div>
      <div v-if="isLoading" class="loading">
        <div class="loading-spinner"></div>
        <span>AI 正在思考中...</span>
      </div>
    </div>

    <div class="input-area">
      <input 
        v-model="inputDialog" 
        @keyup.enter="sendMessage"
        placeholder="输入你想说的话..." 
      />
      <button @click="sendMessage" :disabled="isLoading">
        <span v-if="!isLoading">发送</span>
        <span v-else>发送中...</span>
      </button>
    </div>
  </div>
</template>

<style scoped>
.chat-container {
  max-width: 1200px;
  margin: 40px auto;
  padding: 1.5rem 2.5rem 1.5rem 2.5rem;
  display: flex;
  flex-direction: column;
  background: linear-gradient(135deg, #f5f7fa 0%, #e4e8eb 100%);
  border-radius: 24px;
  box-shadow: 0 6px 32px 0 rgba(52, 152, 219, 0.08);
}

.chat-header {
  text-align: center;
  margin-bottom: 1.2rem;
}

.chat-header h1 {
  font-size: 2.2rem;
  color: #2c3e50;
  margin: 0;
  font-weight: 700;
  background: linear-gradient(120deg, #2c3e50, #3498db);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.subtitle {
  color: #7f8c8d;
  margin-top: 0.3rem;
  font-size: 1rem;
}

.settings-panel {
  background: white;
  padding: 1rem 1.5rem;
  border-radius: 14px;
  margin-bottom: 1rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.2rem;
}

.input-group {
  display: flex;
  flex-direction: column;
}

.input-group label {
  font-size: 0.95rem;
  color: #34495e;
  margin-bottom: 0.3rem;
  font-weight: 600;
}

.input-group input {
  padding: 0.7rem 1rem;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  font-size: 1rem;
  background: #f8f9fa;
  transition: all 0.3s;
}

.input-group input:focus {
  border-color: #3498db;
  outline: none;
  box-shadow: 0 0 0 2px rgba(52, 152, 219, 0.08);
}

.chat-messages {
  height: 350px;
  min-height: 200px;
  max-height: 350px;
  overflow-y: auto;
  padding: 1rem 1.5rem;
  background: white;
  border-radius: 14px;
  margin-bottom: 1rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
}

.message {
  margin-bottom: 0.7rem;
  display: flex;
  flex-direction: column;
}

.message.user {
  align-items: flex-end;
}

.message.ai {
  align-items: flex-start;
}

.message-content {
  padding: 0.8rem 1.2rem;
  border-radius: 10px;
  max-width: 70%;
  line-height: 1.5;
  font-size: 1rem;
  animation: messageAppear 0.3s ease;
}

.message.user .message-content {
  background: linear-gradient(135deg, #3498db, #2980b9);
  color: white;
  border-bottom-right-radius: 4px;
}

.message.ai .message-content {
  background: #f8f9fa;
  color: #2c3e50;
  border-bottom-left-radius: 4px;
}

.input-area {
  display: flex;
  gap: 0.8rem;
  background: white;
  padding: 0.7rem 1rem;
  border-radius: 14px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
}

.input-area input {
  flex: 1;
  padding: 0.8rem;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  font-size: 1rem;
  background: #f8f9fa;
  transition: all 0.3s;
}

.input-area input:focus {
  border-color: #3498db;
  outline: none;
  box-shadow: 0 0 0 2px rgba(52, 152, 219, 0.08);
}

.input-area button {
  padding: 0.8rem 1.5rem;
  background: linear-gradient(135deg, #3498db, #2980b9);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
  min-width: 100px;
}

.input-area button:hover:not(:disabled) {
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(52, 152, 219, 0.12);
}

.input-area button:disabled {
  background: #bdc3c7;
  cursor: not-allowed;
  transform: none;
  box-shadow: none;
}

.loading {
  display: flex;
  align-items: center;
  gap: 0.7rem;
  color: #7f8c8d;
  padding: 0.7rem;
  background: #f8f9fa;
  border-radius: 8px;
  margin-top: 0.7rem;
}

.loading-spinner {
  width: 20px;
  height: 20px;
  border: 3px solid #f3f3f3;
  border-top: 3px solid #3498db;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

@keyframes messageAppear {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 自定义滚动条样式 */
.chat-messages::-webkit-scrollbar {
  width: 8px;
}

.chat-messages::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 4px;
}

.chat-messages::-webkit-scrollbar-thumb {
  background: #c1c1c1;
  border-radius: 4px;
}

.chat-messages::-webkit-scrollbar-thumb:hover {
  background: #a8a8a8;
}
</style> 