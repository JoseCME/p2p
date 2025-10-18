<script setup>
import { ref, onMounted, watch } from 'vue'
const nuevaTarea = ref('')
const tareas = ref([
 { id: 1, texto: 'Aprender Vue.js' },
 { id: 2, texto: 'Construir una app' },
 { id: 3, texto: 'Desplegar con Nginx' }
])
function agregarTarea() {
 if (nuevaTarea.value.trim() === '') return
 tareas.value.push({ id: Date.now(), texto: nuevaTarea.value })
 nuevaTarea.value = ''
}
function eliminarTarea(id) {
  tareas.value = tareas.value.filter(tarea => tarea.id !== id)
}
function guardarTareas() {
  localStorage.setItem('tareas', JSON.stringify(tareas.value))
}

function cargarTareas() {
  const tareasGuardadas = localStorage.getItem('tareas')
  if (tareasGuardadas) {
    tareas.value = JSON.parse(tareasGuardadas)
  }
}

onMounted(() => {
  cargarTareas()
})

watch(tareas, () => {
  guardarTareas()
}, { deep: true })
</script>

<template>
 <div class="app-container">
  <h1>Mi Lista de Tareas</h1>

  <div class="input-section">
   <input v-model="nuevaTarea" @keyup.enter="agregarTarea" placeholder="Añadir nueva tarea" class="task-input">
   <button @click="agregarTarea" class="add-btn">Agregar</button>
  </div>

  <transition-group name="task" tag="ul" class="task-list">
   <li v-for="tarea in tareas" :key="tarea.id" class="task-item">
    {{ tarea.texto }}
    <button @click="eliminarTarea(tarea.id)" class="delete-btn">🗑️</button>
   </li>
  </transition-group>
 </div>
</template>

<style scoped>
.app-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 30px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  color: #333;
  min-height: 500px;
}

h1 {
  text-align: center;
  color: #fff;
  font-size: 2.5em;
  margin-bottom: 30px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
  font-weight: 300;
}

.input-section {
  display: flex;
  justify-content: center;
  margin-bottom: 30px;
}

.task-input {
  padding: 15px 20px;
  border-radius: 50px;
  border: none;
  width: 70%;
  margin-right: 15px;
  font-size: 1.1em;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.task-input:focus {
  outline: none;
  box-shadow: 0 4px 20px rgba(102, 126, 234, 0.4);
  transform: scale(1.02);
}

.add-btn {
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

.add-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(255, 107, 107, 0.4);
}

.add-btn:active {
  transform: translateY(0);
}

.task-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.task-item {
  background-color: rgba(255, 255, 255, 0.95);
  padding: 20px;
  margin-bottom: 15px;
  border-radius: 15px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
  transition: all 0.3s ease;
}

.task-item:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}

.delete-btn {
  background: linear-gradient(45deg, #ff4757, #ff3838);
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 50%;
  cursor: pointer;
  font-size: 1.2em;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(255, 71, 87, 0.3);
}

.delete-btn:hover {
  transform: scale(1.1);
  box-shadow: 0 6px 20px rgba(255, 71, 87, 0.4);
}

.delete-btn:active {
  transform: scale(0.95);
}

/* Animaciones tipo Duolingo */
.task-enter-active {
  animation: slideIn 0.5s ease-out;
}

.task-leave-active {
  animation: slideOut 0.4s ease-in;
}

.task-move {
  transition: transform 0.3s ease;
}

@keyframes slideIn {
  0% {
    opacity: 0;
    transform: translateX(100%) scale(0.8);
  }
  50% {
    opacity: 0.5;
    transform: translateX(-10%) scale(1.05);
  }
  100% {
    opacity: 1;
    transform: translateX(0) scale(1);
  }
}

@keyframes slideOut {
  0% {
    opacity: 1;
    transform: translateX(0) scale(1);
  }
  100% {
    opacity: 0;
    transform: translateX(-100%) scale(0.8);
  }
}
</style>
