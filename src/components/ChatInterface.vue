<script setup>
import { ref } from 'vue'

const username = ref('')
const role = ref('')
const inputDialog = ref('')
const messages = ref([])
const isLoading = ref(false)
const isFirstMessage = ref(true)  // 添加标记是否是第一条消息
const lastRole = ref('')  // 记录上一次发送消息时的角色

const sendMessage = async () => {
  if (!username.value || !role.value || !inputDialog.value) {
    alert('请填写完整信息！')
    return
  }

  // 只在发送消息时检查角色是否发生变化
  if (messages.value.length > 0 && lastRole.value !== role.value) {
    if (confirm('更换AI角色将清空当前聊天记录，是否继续？')) {
      messages.value = []
    } else {
      role.value = lastRole.value
      return
    }
  }

  if (isFirstMessage.value) {
    isFirstMessage.value = false
  }

  // 更新上一次的角色
  lastRole.value = role.value

  // 添加用户消息到聊天记录
  messages.value.push({
    type: 'user',
    content: inputDialog.value
  })

  const currentInput = inputDialog.value
  inputDialog.value = ''  // 立即清空输入框

  isLoading.value = true

  try {
    const response = await fetch('http://59.110.33.91:9265/role-play-reply', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        username: username.value,
        role: role.value,
        input_dialog: currentInput
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
  }
}

const clearChat = () => {
  if (messages.value.length > 0) {
    if (confirm('确定要清空所有聊天记录吗？')) {
      messages.value = []
    }
  }
}
</script>

<template>
  <div class="app-container">
    <div class="chat-container">
      <div class="chat-header">
        <div class="header-content">
          <h1>AI 角色扮演</h1>
          <p class="subtitle">开启一段奇妙的对话之旅</p>
        </div>
        <div class="header-decoration">
          <span class="decoration-icon">🎭</span>
        </div>
      </div>

      <div class="settings-panel">
        <div class="input-group">
          <label>
            <span class="label-icon">👤</span>
            你的名字
          </label>
          <input v-model="username" placeholder="请输入你的名字" :disabled="!isFirstMessage" />
        </div>
        <div class="input-group">
          <label>
            <span class="label-icon">🎯</span>
            AI 角色
          </label>
          <input v-model="role" placeholder="请输入 AI 角色" />
        </div>
        <button 
          class="clear-button" 
          @click="clearChat" 
          :disabled="messages.length === 0"
          title="清空聊天记录"
        >
          <span class="clear-icon">🗑️</span>
          <span class="clear-text">清空记录</span>
        </button>
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
          :disabled="isLoading"
        />
        <button @click="sendMessage" :disabled="isLoading">
          <span v-if="!isLoading">发送</span>
          <span v-else>发送中...</span>
        </button>
      </div>
    </div>

    <div class="theme-panel">
      <div class="theme-header">
        <h2>角色扮演指南</h2>
        <div class="theme-decoration">✨</div>
      </div>
      
      <div class="theme-content">
        <div class="theme-section">
          <h3>🎭 角色设定</h3>
          <p>你可以让AI扮演任何角色，比如：</p>
          <ul>
            <li>历史人物：诸葛亮、莎士比亚</li>
            <li>文学角色：哈利波特、孙悟空</li>
            <li>职业角色：心理医生、律师</li>
            <li>奇幻角色：精灵、魔法师</li>
          </ul>
        </div>

        <div class="theme-section">
          <h3>💡 对话技巧</h3>
          <ul>
            <li>清晰描述角色特征</li>
            <li>设定具体场景背景</li>
            <li>保持角色一致性</li>
            <li>循序渐进地展开对话</li>
          </ul>
        </div>

        <div class="theme-section">
          <h3>🌟 示例场景</h3>
          <div class="example-scene">
            <p class="scene-title">魔法学院</p>
            <p class="scene-desc">扮演一位魔法学院的教授，指导学生掌握魔法技能</p>
          </div>
          <div class="example-scene">
            <p class="scene-title">古代茶馆</p>
            <p class="scene-desc">扮演一位说书人，讲述古代传奇故事</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.app-container {
  display: flex;
  gap: 2rem;
  max-width: 1600px;
  margin: 20px auto;
  padding: 0 20px;
  height: calc(100vh - 40px);
}

.chat-container {
  flex: 1;
  display: flex;
  flex-direction: column;
  background: linear-gradient(135deg, #f6f8fc 0%, #eef2f7 100%);
  border-radius: 24px;
  box-shadow: 0 8px 32px rgba(31, 38, 135, 0.1);
  backdrop-filter: blur(4px);
  border: 1px solid rgba(255, 255, 255, 0.18);
  padding: 1.5rem;
  min-width: 0; /* 防止flex子项溢出 */
}

.theme-panel {
  width: 360px;
  background: rgba(255, 255, 255, 0.9);
  border-radius: 24px;
  box-shadow: 0 8px 32px rgba(31, 38, 135, 0.1);
  backdrop-filter: blur(4px);
  border: 1px solid rgba(255, 255, 255, 0.18);
  padding: 1.5rem;
  overflow-y: auto;
}

.theme-header {
  text-align: center;
  margin-bottom: 2rem;
  position: relative;
}

.theme-header h2 {
  font-size: 1.8rem;
  color: #2c3e50;
  margin: 0;
  background: linear-gradient(120deg, #2c3e50, #3498db);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.theme-decoration {
  font-size: 2rem;
  margin-top: 0.5rem;
  animation: float 3s ease-in-out infinite;
}

.theme-section {
  margin-bottom: 2rem;
  padding: 1.2rem;
  background: rgba(255, 255, 255, 0.5);
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.5);
}

.theme-section h3 {
  color: #2c3e50;
  margin: 0 0 1rem 0;
  font-size: 1.2rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.theme-section p {
  color: #34495e;
  margin: 0.5rem 0;
  line-height: 1.6;
}

.theme-section ul {
  margin: 0.5rem 0;
  padding-left: 1.5rem;
  color: #34495e;
}

.theme-section li {
  margin: 0.5rem 0;
  line-height: 1.5;
}

.example-scene {
  background: rgba(52, 152, 219, 0.1);
  border-radius: 12px;
  padding: 1rem;
  margin: 0.8rem 0;
  transition: all 0.3s;
}

.example-scene:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(52, 152, 219, 0.1);
}

.scene-title {
  font-weight: 600;
  color: #2c3e50;
  margin: 0 0 0.5rem 0;
}

.scene-desc {
  color: #7f8c8d;
  font-size: 0.9rem;
  margin: 0;
}

.chat-messages {
  flex: 1;
  min-height: 0;
  overflow-y: auto;
  padding: 1.5rem;
  background: rgba(255, 255, 255, 0.9);
  border-radius: 16px;
  margin-bottom: 1.5rem;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.05);
  backdrop-filter: blur(4px);
  border: 1px solid rgba(255, 255, 255, 0.5);
}

.chat-header {
  text-align: center;
  margin-bottom: 2rem;
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1rem;
}

.header-content {
  text-align: center;
}

.header-decoration {
  font-size: 2.5rem;
  animation: float 3s ease-in-out infinite;
}

@keyframes float {
  0% { transform: translateY(0px); }
  50% { transform: translateY(-10px); }
  100% { transform: translateY(0px); }
}

.chat-header h1 {
  font-size: 2.5rem;
  color: #2c3e50;
  margin: 0;
  font-weight: 700;
  background: linear-gradient(120deg, #2c3e50, #3498db);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
}

.subtitle {
  color: #7f8c8d;
  margin-top: 0.5rem;
  font-size: 1.1rem;
  font-weight: 500;
}

.settings-panel {
  background: rgba(255, 255, 255, 0.9);
  padding: 1.5rem;
  border-radius: 16px;
  margin-bottom: 1.5rem;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.05);
  display: grid;
  grid-template-columns: 1fr 1fr auto;
  gap: 1.5rem;
  align-items: end;
  backdrop-filter: blur(4px);
  border: 1px solid rgba(255, 255, 255, 0.5);
}

.input-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.input-group label {
  font-size: 1rem;
  color: #34495e;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.label-icon {
  font-size: 1.2rem;
}

.input-group input {
  padding: 0.8rem 1rem;
  border: 2px solid #e0e0e0;
  border-radius: 12px;
  font-size: 1rem;
  background: #f8f9fa;
  transition: all 0.3s;
  color: #2c3e50;
}

.input-group input:focus {
  border-color: #3498db;
  outline: none;
  box-shadow: 0 0 0 3px rgba(52, 152, 219, 0.1);
}

.input-group input:disabled {
  background: #f1f3f5;
  cursor: not-allowed;
  opacity: 0.7;
}

.message {
  margin-bottom: 1rem;
  display: flex;
  flex-direction: column;
  animation: messageSlide 0.3s ease-out;
}

@keyframes messageSlide {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.message.user {
  align-items: flex-end;
}

.message.ai {
  align-items: flex-start;
}

.message-content {
  padding: 1rem 1.2rem;
  border-radius: 16px;
  max-width: 70%;
  line-height: 1.6;
  font-size: 1rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
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
  gap: 1rem;
  background: rgba(255, 255, 255, 0.9);
  padding: 1rem;
  border-radius: 16px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.05);
  backdrop-filter: blur(4px);
  border: 1px solid rgba(255, 255, 255, 0.5);
}

.input-area input {
  flex: 1;
  padding: 1rem;
  border: 2px solid #e0e0e0;
  border-radius: 12px;
  font-size: 1rem;
  background: #f8f9fa;
  transition: all 0.3s;
  color: #2c3e50;
}

.input-area input:focus {
  border-color: #3498db;
  outline: none;
  box-shadow: 0 0 0 3px rgba(52, 152, 219, 0.1);
}

.input-area input:disabled {
  background: #f1f3f5;
  cursor: not-allowed;
  opacity: 0.7;
}

.input-area button {
  padding: 1rem 2rem;
  background: linear-gradient(135deg, #3498db, #2980b9);
  color: white;
  border: none;
  border-radius: 12px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
  min-width: 120px;
}

.input-area button:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(52, 152, 219, 0.2);
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
  gap: 0.8rem;
  color: #7f8c8d;
  padding: 1rem;
  background: #f8f9fa;
  border-radius: 12px;
  margin-top: 1rem;
  animation: pulse 1.5s infinite;
}

@keyframes pulse {
  0% { opacity: 0.6; }
  50% { opacity: 1; }
  100% { opacity: 0.6; }
}

.loading-spinner {
  width: 24px;
  height: 24px;
  border: 3px solid #f3f3f3;
  border-top: 3px solid #3498db;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.clear-button {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.8rem 1.2rem;
  background: #f8f9fa;
  color: #6c757d;
  border: 1px solid #e9ecef;
  border-radius: 12px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: all 0.3s;
  height: fit-content;
  align-self: flex-end;
}

.clear-button:hover:not(:disabled) {
  background: #e9ecef;
  color: #495057;
  border-color: #dee2e6;
  transform: translateY(-2px);
}

.clear-button:disabled {
  background: #f8f9fa;
  color: #adb5bd;
  cursor: not-allowed;
  border-color: #e9ecef;
}

.clear-icon {
  font-size: 1.1rem;
}

.clear-text {
  font-weight: 500;
}

@media (max-width: 1200px) {
  .app-container {
    flex-direction: column;
    height: auto;
  }

  .theme-panel {
    width: auto;
  }

  .chat-container {
    margin-bottom: 2rem;
  }
}

@media (max-width: 768px) {
  .app-container {
    margin: 10px;
    padding: 0 10px;
  }

  .chat-container {
    padding: 1rem;
  }

  .theme-panel {
    padding: 1rem;
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