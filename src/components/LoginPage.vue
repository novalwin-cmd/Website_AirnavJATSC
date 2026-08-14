<script setup>
import { ref } from 'vue'

const username = ref('')
const password = ref('')
const error = ref('')

const emit = defineEmits(['login'])

const handleLogin = () => {
  error.value = ''

  if ((username.value === 'admin' && password.value === 'airnav2026') ||
      (username.value === 'engineer' && password.value === 'monitoring')) {
    emit('login', username.value)
  } else {
    error.value = 'Invalid credentials!'
  }
}

const handleKeydown = (e) => {
  if (e.key === 'Enter') {
    handleLogin()
  }
}
</script>

<template>
  <div class="login-container">
    <div class="login-box">
      <div class="login-header">
        <div class="brand">AIRNAV</div>
        <h1>Secure access</h1>
        <p>Sign in dengan salah satu configured local accounts. Admin dapat use monitoring dan control, sementara Engineer hanya monitoring.</p>
      </div>

      <form @submit.prevent="handleLogin">
        <div class="form-group">
          <label>Username</label>
          <input
            v-model="username"
            type="text"
            placeholder="admin or engineer"
            required
            @keydown="handleKeydown"
          >
        </div>

        <div class="form-group">
          <label>Password</label>
          <input
            v-model="password"
            type="password"
            placeholder="••••••••"
            required
            @keydown="handleKeydown"
          >
        </div>

        <div v-if="error" class="error-message">{{ error }}</div>

        <button type="submit" class="login-btn">Login</button>
      </form>

      <div class="available-accounts">
        <h3>Available accounts</h3>
        <ul>
          <li><strong>admin</strong> / airnav2026 (control + monitoring)</li>
          <li><strong>engineer</strong> / monitoring (monitoring only)</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<style scoped>
.login-container {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background: linear-gradient(135deg, #0f1419 0%, #1a2332 100%);
}

.login-box {
  background: rgba(26, 35, 50, 0.8);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(76, 219, 189, 0.2);
  border-radius: 20px;
  padding: 60px 40px;
  width: 100%;
  max-width: 450px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
}

.login-header {
  text-align: center;
  margin-bottom: 50px;
}

.login-header .brand {
  font-size: 12px;
  letter-spacing: 3px;
  color: #4cdbbd;
  margin-bottom: 15px;
  text-transform: uppercase;
}

.login-header h1 {
  font-size: 36px;
  color: #ffffff;
  margin-bottom: 15px;
  font-weight: 600;
}

.login-header p {
  color: #a0aec0;
  font-size: 14px;
  line-height: 1.6;
}

.form-group {
  margin-bottom: 25px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  color: #cbd5e0;
  font-size: 14px;
}

.form-group input {
  width: 100%;
  padding: 12px 16px;
  background: rgba(30, 41, 59, 0.6);
  border: 2px solid rgba(76, 219, 189, 0.3);
  border-radius: 8px;
  color: #ffffff;
  font-size: 14px;
  transition: all 0.3s ease;
}

.form-group input:focus {
  outline: none;
  border-color: #4cdbbd;
  box-shadow: 0 0 0 3px rgba(76, 219, 189, 0.1);
}

.form-group input::placeholder {
  color: #718096;
}

.error-message {
  color: #ef4444;
  font-size: 13px;
  margin-bottom: 15px;
  padding: 10px;
  background: rgba(239, 68, 68, 0.1);
  border: 1px solid rgba(239, 68, 68, 0.3);
  border-radius: 4px;
  text-align: center;
}

.login-btn {
  width: 100%;
  padding: 14px;
  background: #4cdbbd;
  border: none;
  border-radius: 8px;
  color: #0f1419;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  margin-bottom: 30px;
}

.login-btn:hover {
  background: #3ac9ad;
  transform: translateY(-2px);
  box-shadow: 0 10px 25px rgba(76, 219, 189, 0.3);
}

.available-accounts {
  background: rgba(30, 41, 59, 0.6);
  border: 1px solid rgba(76, 219, 189, 0.2);
  border-radius: 8px;
  padding: 20px;
}

.available-accounts h3 {
  color: #ffffff;
  font-size: 14px;
  margin-bottom: 12px;
}

.available-accounts ul {
  list-style: none;
}

.available-accounts li {
  padding: 8px 0;
  font-size: 13px;
  color: #a0aec0;
}

.available-accounts strong {
  color: #4cdbbd;
}
</style>
