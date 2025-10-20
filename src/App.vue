<script setup>
import { ref, onMounted, watch } from 'vue'
import AppLogin from './components/AppLogin.vue'
import AppPomodoro from './components/AppPomodoro.vue'

const userName = ref('')
const isLoggedIn = ref(false)
const nuevaTarea = ref('')
const tareas = ref([
 { id: 1, texto: 'Aprender Vue.js', completada: false },
 { id: 2, texto: 'Construir una app', completada: false },
 { id: 3, texto: 'Desplegar con Nginx', completada: false }
])
const tareaSeleccionada = ref(null)
const mostrarPomodoro = ref(false)
const mostrarConfiguracion = ref(false)
const sonidoActivado = ref(true)

function handleLogin(name) {
  userName.value = name
  isLoggedIn.value = true
  localStorage.setItem('userName', name)
}

function agregarTarea() {
 if (nuevaTarea.value.trim() === '') return
 tareas.value.push({ id: Date.now(), texto: nuevaTarea.value, completada: false })
 nuevaTarea.value = ''
}

function eliminarTarea(id) {
  tareas.value = tareas.value.filter(tarea => tarea.id !== id)
  // Si se elimina la tarea seleccionada, detener pomodoro
  if (tareaSeleccionada.value && tareaSeleccionada.value.id === id) {
    tareaSeleccionada.value = null
    mostrarPomodoro.value = false
  }
}

function seleccionarTarea(tarea) {
  tareaSeleccionada.value = tarea
  mostrarPomodoro.value = true
}

function completarTarea(id) {
  const tarea = tareas.value.find(t => t.id === id)
  if (tarea) {
    tarea.completada = !tarea.completada
    if (tarea.completada) {
      // Si se completa la tarea, detener pomodoro
      if (tareaSeleccionada.value && tareaSeleccionada.value.id === id) {
        tareaSeleccionada.value = null
        mostrarPomodoro.value = false
      }
    }
  }
}

function detenerPomodoro() {
  mostrarPomodoro.value = false
  tareaSeleccionada.value = null
}

function playSound(frequency = 800, duration = 200, type = 'sine') {
  if (!sonidoActivado.value) return
  
  try {
    const audioContext = new (window.AudioContext || window.webkitAudioContext)()
    const oscillator = audioContext.createOscillator()
    const gainNode = audioContext.createGain()

    oscillator.connect(gainNode)
    gainNode.connect(audioContext.destination)

    oscillator.frequency.setValueAtTime(frequency, audioContext.currentTime)
    oscillator.type = type

    gainNode.gain.setValueAtTime(0.2, audioContext.currentTime)
    gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + duration / 1000)

    oscillator.start(audioContext.currentTime)
    oscillator.stop(audioContext.currentTime + duration / 1000)
  } catch (e) {
    // Fallback si Web Audio API no está disponible
    console.log('Sonido no disponible')
  }
}

function playHoverSound() {
  playSound(400, 50, 'sine')
}

function playClickSound() {
  playSound(600, 100, 'square')
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

function cargarUsuario() {
  const savedUser = localStorage.getItem('userName')
  if (savedUser) {
    userName.value = savedUser
    isLoggedIn.value = true
  }
  // Cargar preferencia de sonido
  const savedSonido = localStorage.getItem('sonidoActivado')
  if (savedSonido !== null) {
    sonidoActivado.value = JSON.parse(savedSonido)
  }
}

function toggleConfiguracion() {
  mostrarConfiguracion.value = !mostrarConfiguracion.value
  playClickSound()
}

function toggleSonido() {
  sonidoActivado.value = !sonidoActivado.value
  playClickSound()
  // Guardar preferencia en localStorage
  localStorage.setItem('sonidoActivado', sonidoActivado.value)
}

function cerrarSesion() {
  playClickSound()
  // Limpiar datos de sesión
  userName.value = ''
  isLoggedIn.value = false
  tareaSeleccionada.value = null
  mostrarPomodoro.value = false
  mostrarConfiguracion.value = false
  
  // Limpiar localStorage
  localStorage.removeItem('userName')
  localStorage.removeItem('tareas')
  localStorage.removeItem('sonidoActivado')
  localStorage.removeItem('pomodoroSettings')
}

onMounted(() => {
  cargarUsuario()
  cargarTareas()
})

watch(tareas, () => {
  guardarTareas()
}, { deep: true })
</script>

<template>
  <!-- Solo mostrar login si es primera vez -->
  <div v-if="!isLoggedIn" class="app">
    <AppLogin @login="handleLogin" />
  </div>

  <!-- Mostrar aplicación completa si ya está logueado -->
  <div v-else class="app">
    <div class="app-container">
      <header class="app-header">
        <div class="header-content">
          <div>
            <h1>¡Hola, {{ userName }}! 👋</h1>
            <p>Organiza tu tiempo y tareas</p>
          </div>
          <button @click="toggleConfiguracion" @mouseenter="playHoverSound" class="config-btn">
            ⚙️
          </button>
        </div>
      </header>

      <!-- Panel de configuración -->
      <div v-if="mostrarConfiguracion" class="config-panel">
        <div class="config-content">
          <h3>⚙️ Configuración</h3>
          <div class="config-options">
            <div class="config-option">
              <label>Sonido:</label>
              <button @click="toggleSonido" @mouseenter="playHoverSound" class="toggle-btn" :class="{ active: sonidoActivado }">
                {{ sonidoActivado ? '🔊 Activado' : '🔇 Desactivado' }}
              </button>
            </div>
            <div class="config-option">
              <button @click="cerrarSesion" @mouseenter="playHoverSound" class="logout-btn">
                🚪 Cerrar Sesión
              </button>
            </div>
          </div>
          <button @click="toggleConfiguracion" @mouseenter="playHoverSound" class="close-config-btn">✕</button>
        </div>
      </div>

      <div class="main-content">
        <!-- Sección de tareas -->
        <div class="tasks-section">
          <h2>Lista de Tareas</h2>

          <div class="input-section">
           <input v-model="nuevaTarea" @keyup.enter="agregarTarea" placeholder="Añadir nueva tarea" class="task-input">
           <button @click="agregarTarea" @mouseenter="playHoverSound" class="add-btn">Agregar</button>
          </div>

          <transition-group name="task" tag="ul" class="task-list">
           <li v-for="tarea in tareas" :key="tarea.id" class="task-item" :class="{ 'completada': tarea.completada, 'seleccionada': tareaSeleccionada && tareaSeleccionada.id === tarea.id }">
            <div class="task-content">
              <input type="checkbox" :checked="tarea.completada" @change="completarTarea(tarea.id)" class="task-checkbox">
              <span class="task-text" :class="{ 'completed-text': tarea.completada }">{{ tarea.texto }}</span>
            </div>
            <div class="task-actions">
              <button @click="seleccionarTarea(tarea)" @mouseenter="playHoverSound" class="pomodoro-btn" :disabled="tarea.completada || (tareaSeleccionada && tareaSeleccionada.id === tarea.id)">
                {{ tareaSeleccionada && tareaSeleccionada.id === tarea.id ? '⏱️ Activo' : '🍅 Pomodoro' }}
              </button>
              <button @click="eliminarTarea(tarea.id)" @mouseenter="playHoverSound" class="delete-btn">🗑️</button>
            </div>
           </li>
          </transition-group>
        </div>

        <!-- Sección de pomodoro (solo si hay tarea seleccionada) -->
        <div v-if="mostrarPomodoro && tareaSeleccionada" class="pomodoro-section">
          <div class="pomodoro-header">
            <h3>Trabajando en: {{ tareaSeleccionada.texto }}</h3>
            <button @click="detenerPomodoro" @mouseenter="playHoverSound" class="close-pomodoro-btn">✕</button>
          </div>
          <AppPomodoro :selectedTask="tareaSeleccionada" :sonidoActivado="sonidoActivado" @taskCompleted="completarTarea" @pomodoroCompleted="onPomodoroCompleted" />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.app {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.app-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.app-header {
  text-align: center;
  color: white;
  margin-bottom: 30px;
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  max-width: 1200px;
  margin: 0 auto;
}

.header-content > div {
  flex: 1;
}

.config-btn {
  background: rgba(255, 255, 255, 0.2);
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  width: 50px;
  height: 50px;
  font-size: 1.5em;
  color: white;
  cursor: pointer;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
}

.config-btn:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: scale(1.1);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.config-panel {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  backdrop-filter: blur(5px);
}

.config-content {
  background: rgba(255, 255, 255, 0.95);
  padding: 30px;
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  backdrop-filter: blur(10px);
  max-width: 400px;
  width: 90%;
  position: relative;
}

.config-content h3 {
  margin: 0 0 20px 0;
  color: #333;
  text-align: center;
}

.config-options {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.config-option {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.config-option label {
  font-weight: 600;
  color: #555;
}

.toggle-btn {
  padding: 10px 20px;
  border-radius: 25px;
  border: none;
  font-size: 0.9em;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.toggle-btn.active {
  background: linear-gradient(45deg, #4caf50, #45a049);
  color: white;
  box-shadow: 0 4px 15px rgba(76, 175, 80, 0.3);
}

.toggle-btn:not(.active) {
  background: linear-gradient(45deg, #757575, #616161);
  color: white;
  box-shadow: 0 4px 15px rgba(117, 117, 117, 0.3);
}

.toggle-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.2);
}

.logout-btn {
  width: 100%;
  padding: 12px;
  border-radius: 25px;
  border: none;
  background: linear-gradient(45deg, #f44336, #d32f2f);
  color: white;
  font-size: 1em;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(244, 67, 54, 0.3);
}

.logout-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(244, 67, 54, 0.4);
}

.close-config-btn {
  position: absolute;
  top: 15px;
  right: 15px;
  background: #f44336;
  color: white;
  border: none;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  cursor: pointer;
  font-size: 1.2em;
  transition: all 0.3s ease;
}

.close-config-btn:hover {
  background: #d32f2f;
  transform: scale(1.1);
}

.main-content {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 30px;
  align-items: start;
}

.pomodoro-section,
.tasks-section {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  padding: 30px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
}

.tasks-section h2 {
  text-align: center;
  color: #333;
  margin-bottom: 30px;
  font-size: 1.8em;
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
  border: 2px solid #e1e5e9;
  width: 70%;
  margin-right: 15px;
  font-size: 1.1em;
  transition: all 0.3s ease;
}

.task-input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
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
  background-color: rgba(255, 255, 255, 0.9);
  padding: 20px;
  margin-bottom: 15px;
  border-radius: 15px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(5px);
  transition: all 0.3s ease;
}

.task-item:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}

.task-item.completada {
  background-color: rgba(76, 175, 80, 0.1);
  opacity: 0.7;
}

.task-item.seleccionada {
  background-color: rgba(102, 126, 234, 0.1);
  border: 2px solid rgba(102, 126, 234, 0.3);
  box-shadow: 0 4px 20px rgba(102, 126, 234, 0.2);
}

.task-content {
  display: flex;
  align-items: center;
  flex: 1;
}

.task-checkbox {
  margin-right: 15px;
  width: 18px;
  height: 18px;
  cursor: pointer;
  accent-color: #4caf50;
}

.task-text {
  flex: 1;
  font-size: 1.1em;
}

.task-text.completed-text {
  text-decoration: line-through;
  color: #666;
}

.task-actions {
  display: flex;
  gap: 10px;
  align-items: center;
}

.pomodoro-btn {
  padding: 8px 15px;
  border-radius: 20px;
  border: none;
  background: linear-gradient(45deg, #ff6b6b, #ee5a24);
  color: white;
  font-size: 0.9em;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 2px 8px rgba(255, 107, 107, 0.3);
}

.pomodoro-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(255, 107, 107, 0.4);
}

.pomodoro-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  background: #ccc;
}

.delete-btn {
  background: linear-gradient(45deg, #ff4757, #ff3838);
  color: white;
  border: none;
  padding: 8px 12px;
  border-radius: 50%;
  cursor: pointer;
  font-size: 1.1em;
  transition: all 0.3s ease;
  box-shadow: 0 2px 8px rgba(255, 71, 87, 0.3);
}

.delete-btn:hover {
  transform: scale(1.1);
  box-shadow: 0 4px 12px rgba(255, 71, 87, 0.4);
}

.delete-btn:active {
  transform: scale(0.95);
}

.pomodoro-section {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  padding: 30px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
  max-width: 400px;
  width: 100%;
}

.pomodoro-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding-bottom: 15px;
  border-bottom: 2px solid #e1e5e9;
}

.pomodoro-header h3 {
  margin: 0;
  color: #333;
  font-size: 1.1em;
  font-weight: 600;
}

.close-pomodoro-btn {
  background: #f44336;
  color: white;
  border: none;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  cursor: pointer;
  font-size: 1.2em;
  transition: all 0.3s ease;
}

.close-pomodoro-btn:hover {
  background: #d32f2f;
  transform: scale(1.1);
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

/* Responsive */
@media (max-width: 768px) {
  .header-content {
    flex-direction: column;
    gap: 15px;
  }

  .header-content > div {
    text-align: center;
  }

  .config-btn {
    width: 40px;
    height: 40px;
    font-size: 1.2em;
  }

  .main-content {
    grid-template-columns: 1fr;
    gap: 20px;
  }

  .app-header h1 {
    font-size: 2em;
  }

  .pomodoro-section,
  .tasks-section {
    padding: 20px;
  }

  .task-item {
    flex-direction: column;
    align-items: flex-start;
    gap: 15px;
  }

  .task-actions {
    width: 100%;
    justify-content: space-between;
  }

  .pomodoro-btn {
    flex: 1;
  }

  .task-input {
    width: 60%;
  }

  .config-content {
    margin: 20px;
    padding: 20px;
  }

  .config-options {
    gap: 15px;
  }
}
</style>
