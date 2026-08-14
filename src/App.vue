<script setup>
import { ref } from 'vue'
import LoginPage from './components/LoginPage.vue'
import Dashboard from './components/Dashboard.vue'

const isLoggedIn = ref(false)
const currentUser = ref(null)

const handleLogin = (username) => {
  currentUser.value = username
  isLoggedIn.value = true
}

const handleLogout = () => {
  isLoggedIn.value = false
  currentUser.value = null
}
</script>

<template>
  <div class="app">
    <LoginPage v-if="!isLoggedIn" @login="handleLogin" />
    <Dashboard v-else :user="currentUser" @logout="handleLogout" />
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  background: #0f1419;
  color: #e0e0e0;
  overflow-x: hidden;
}

.app {
  width: 100%;
  min-height: 100vh;
}
</style>
