<template>
  <!-- eslint-disable vue/multi-word-component-names -->
  <div class="login-container">
    <div class="login-card">
      <h2>¡Bienvenido!</h2>
      <p>Ingresa tu nombre para comenzar</p>
      <form @submit.prevent="handleLogin" class="login-form">
        <input
          v-model="userName"
          type="text"
          placeholder="Tu nombre"
          class="name-input"
          required
        >
        <button type="submit" class="login-btn">Comenzar</button>
      </form>
    </div>
  </div>
</template>

<script setup>
/* eslint-disable no-undef */
import { ref } from 'vue'

const userName = ref('')
const emit = defineEmits(['login'])

function handleLogin() {
  if (userName.value.trim()) {
    emit('login', userName.value.trim())
  }
}
</script>

<style scoped>
.login-container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.login-card {
  background: rgba(255, 255, 255, 0.95);
  padding: 40px;
  border-radius: 20px;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.1);
  text-align: center;
  backdrop-filter: blur(10px);
  max-width: 400px;
  width: 90%;
}

h2 {
  color: #333;
  margin-bottom: 10px;
  font-size: 2.2em;
  font-weight: 300;
}

p {
  color: #666;
  margin-bottom: 30px;
  font-size: 1.1em;
}

.login-form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.name-input {
  padding: 15px 20px;
  border-radius: 50px;
  border: 2px solid #e1e5e9;
  font-size: 1.1em;
  transition: all 0.3s ease;
  text-align: center;
}

.name-input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
  transform: scale(1.02);
}

.login-btn {
  padding: 15px 30px;
  border-radius: 50px;
  border: none;
  background: linear-gradient(45deg, #ff6b6b, #ee5a24);
  color: white;
  font-size: 1.1em;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(255, 107, 107, 0.3);
}

.login-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(255, 107, 107, 0.4);
}

.login-btn:active {
  transform: translateY(0);
}
</style>