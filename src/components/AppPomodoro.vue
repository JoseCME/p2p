<template>
  <!-- eslint-disable vue/multi-word-component-names -->
  <div class="pomodoro-container">
    <div class="pomodoro-card">
      <!-- Configuración -->
      <div class="settings">
        <h4>⚙️ Configuración</h4>
        <div class="setting-group">
          <label>Trabajo (min):</label>
          <select v-model="workTimeMinutes" @change="updateWorkTime" @click="playClickSound" @mouseenter="playHoverSound" :disabled="isRunning">
            <option v-for="min in [15, 20, 25, 30, 35, 40, 45, 50, 55, 60]" :key="min" :value="min">{{ min }}</option>
          </select>
        </div>
        <div class="setting-group">
          <label>Descanso corto (min):</label>
          <select v-model="shortBreakMinutes" @change="updateShortBreak" @click="playClickSound" @mouseenter="playHoverSound" :disabled="isRunning">
            <option v-for="min in [5, 10, 15, 20]" :key="min" :value="min">{{ min }}</option>
          </select>
        </div>
        <div class="setting-group">
          <label>Descanso largo (min):</label>
          <select v-model="longBreakMinutes" @change="updateLongBreak" @click="playClickSound" @mouseenter="playHoverSound" :disabled="isRunning">
            <option v-for="min in [15, 20, 25, 30]" :key="min" :value="min">{{ min }}</option>
          </select>
        </div>
      </div>

      <div class="tomato">
        <div class="tomato-body">
          <div class="tomato-stem"></div>
          <div class="tomato-face">
            <div class="eyes">
              <div class="eye left"></div>
              <div class="eye right"></div>
            </div>
            <div class="mouth" :class="{ 'smile': isWorkTime, 'rest': !isWorkTime }"></div>
          </div>
          <!-- Animación simple y elegante del tiempo -->
          <div class="time-fill" :style="{ height: timeFillHeight + '%' }"></div>
          <div class="time-particles">
            <div
              v-for="i in 5"
              :key="i"
              class="particle"
              :style="{
                left: particlePositions[i-1] + '%',
                top: particleTops[i-1] + '%',
                opacity: particleOpacities[i-1],
                animationDelay: i * 0.2 + 's'
              }"
            ></div>
          </div>
        </div>
      </div>

      <div class="timer-display">
        <h3>{{ currentMode }}</h3>
        <div
          class="time"
          :class="{ 'editing': isEditingTime }"
          @dblclick="startEditingTime"
          v-if="!isEditingTime"
        >
          {{ formatTime(timeLeft) }}
        </div>
        <div v-else class="time-editor">
          <input
            ref="timeInput"
            v-model="editingMinutes"
            @keyup.enter="saveEditedTime"
            @keyup.escape="cancelEditingTime"
            @blur="saveEditedTime"
            type="number"
            min="1"
            max="999"
            class="time-input"
          >
          <span>min</span>
        </div>
        <div class="progress-bar">
          <div class="progress" :style="{ width: progressPercent + '%' }"></div>
        </div>
      </div>

      <div class="controls">
        <button @click="toggleTimer" @mouseenter="playHoverSound" class="control-btn" :class="{ active: isRunning }">
          {{ isRunning ? 'Pausar' : 'Iniciar' }}
        </button>
        <button @click="resetTimer" @mouseenter="playHoverSound" class="control-btn reset">Reiniciar</button>
      </div>

      <!-- Botón para completar tarea (solo si hay tarea seleccionada) -->
      <div v-if="props.selectedTask" class="task-completion">
        <button @click="completeCurrentTask" @mouseenter="playHoverSound" class="complete-task-btn">
          ✅ Completar Tarea
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
/* eslint-disable no-undef */
import { ref, computed, onMounted, onUnmounted, nextTick } from 'vue'

const props = defineProps({
  selectedTask: {
    type: Object,
    default: null
  },
  sonidoActivado: {
    type: Boolean,
    default: true
  }
})

const emit = defineEmits(['taskCompleted', 'pomodoroCompleted'])

const POMODOROS_BEFORE_LONG_BREAK = 4

// Configuración personalizable
const workTimeMinutes = ref(25)
const shortBreakMinutes = ref(5)
const longBreakMinutes = ref(15)

// Estado del timer
const timeLeft = ref(workTimeMinutes.value * 60)
const isRunning = ref(false)
const isWorkTime = ref(true)
const completedPomodoros = ref(0)
const totalWorkTime = ref(0)
const currentMode = ref('Trabajo')
const isEditingTime = ref(false)
const editingMinutes = ref('')
const timeInput = ref(null)
let interval = null

// Calcular tiempo total actual
const currentTotalTime = computed(() => {
  return isWorkTime.value ? workTimeMinutes.value * 60 :
         (completedPomodoros.value % POMODOROS_BEFORE_LONG_BREAK === 0 ? longBreakMinutes.value * 60 : shortBreakMinutes.value * 60)
})

const progressPercent = computed(() => {
  return ((currentTotalTime.value - timeLeft.value) / currentTotalTime.value) * 100
})

const timerFillPercent = computed(() => {
  return ((currentTotalTime.value - timeLeft.value) / currentTotalTime.value) * 100
})

// Nueva animación simple: el tomate se "llena" con el tiempo transcurrido
const timeFillHeight = computed(() => {
  return timerFillPercent.value
})

// Partículas flotantes para efecto visual
const particlePositions = computed(() => {
  const progress = timerFillPercent.value / 100
  return [20, 35, 50, 65, 80].map((base, index) => base + Math.sin(progress * Math.PI * 2 + index * 0.5) * 10)
})

const particleTops = computed(() => {
  const progress = timerFillPercent.value / 100
  return [30, 45, 60, 75, 90].map((base) => base - progress * 40)
})

const particleOpacities = computed(() => {
  const progress = timerFillPercent.value / 100
  return [0.3, 0.5, 0.7, 0.5, 0.3].map((base, index) =>
    Math.max(0, Math.min(1, base + Math.sin(progress * Math.PI * 3 + index * 0.7) * 0.3))
  )
})

// Solicitar permisos de notificación al montar
onMounted(() => {
  if ('Notification' in window && Notification.permission === 'default') {
    Notification.requestPermission()
  }
  loadSettings()
})

// Limpiar intervalo al desmontar
onUnmounted(() => {
  if (interval) {
    clearInterval(interval)
  }
})

// Funciones de configuración
function updateWorkTime() {
  if (!isRunning.value) {
    timeLeft.value = workTimeMinutes.value * 60
    saveSettings()
  }
}

function updateShortBreak() {
  saveSettings()
}

function updateLongBreak() {
  saveSettings()
}

function saveSettings() {
  const settings = {
    workTimeMinutes: workTimeMinutes.value,
    shortBreakMinutes: shortBreakMinutes.value,
    longBreakMinutes: longBreakMinutes.value
  }
  localStorage.setItem('pomodoroSettings', JSON.stringify(settings))
}

function loadSettings() {
  const saved = localStorage.getItem('pomodoroSettings')
  if (saved) {
    const settings = JSON.parse(saved)
    workTimeMinutes.value = settings.workTimeMinutes || 25
    shortBreakMinutes.value = settings.shortBreakMinutes || 5
    longBreakMinutes.value = settings.longBreakMinutes || 15
    timeLeft.value = workTimeMinutes.value * 60
  }
}

function formatTime(seconds) {
  const mins = Math.floor(seconds / 60)
  const secs = seconds % 60
  return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
}

function showNotification(title, body) {
  if ('Notification' in window && Notification.permission === 'granted') {
    const notification = new Notification(title, {
      body: body,
      icon: '/favicon.ico',
      badge: '/favicon.ico',
      requireInteraction: true, // La notificación permanece hasta que el usuario la cierre
      silent: false // Asegura que suene incluso si el sistema tiene sonido desactivado
    })
    
    // Auto-cerrar después de 10 segundos si no hay interacción
    setTimeout(() => {
      notification.close()
    }, 10000)
    
    // Vibración adicional en móviles
    if ('vibrate' in navigator) {
      navigator.vibrate([300, 100, 300, 100, 300])
    }
  }
}

function playSound(frequency = 800, duration = 200, type = 'sine') {
  if (!props.sonidoActivado) return
  
  try {
    const audioContext = new (window.AudioContext || window.webkitAudioContext)()
    const oscillator = audioContext.createOscillator()
    const gainNode = audioContext.createGain()

    oscillator.connect(gainNode)
    gainNode.connect(audioContext.destination)

    oscillator.frequency.setValueAtTime(frequency, audioContext.currentTime)
    oscillator.type = type

    gainNode.gain.setValueAtTime(0.3, audioContext.currentTime)
    gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + duration / 1000)

    oscillator.start(audioContext.currentTime)
    oscillator.stop(audioContext.currentTime + duration / 1000)
  } catch (e) {
    // Fallback si Web Audio API no está disponible
    console.log('Sonido no disponible')
  }
}

function playClickSound() {
  playSound(600, 100, 'square')
}

function playHoverSound() {
  playSound(400, 50, 'sine')
}

function playNotificationSound() {
  // Sonido de notificación más invasivo - SIEMPRE suena independientemente de la configuración
  try {
    const audioContext = new (window.AudioContext || window.webkitAudioContext)()
    
    // Crear una secuencia de tonos más audible
    const playTone = (frequency, startTime, duration) => {
      const oscillator = audioContext.createOscillator()
      const gainNode = audioContext.createGain()

      oscillator.connect(gainNode)
      gainNode.connect(audioContext.destination)

      oscillator.frequency.setValueAtTime(frequency, audioContext.currentTime + startTime)
      oscillator.type = 'triangle'

      // Volumen más alto para notificaciones
      gainNode.gain.setValueAtTime(0.6, audioContext.currentTime + startTime)
      gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + startTime + duration)

      oscillator.start(audioContext.currentTime + startTime)
      oscillator.stop(audioContext.currentTime + startTime + duration)
    }

    // Secuencia de 3 tonos ascendentes para mayor impacto
    playTone(600, 0, 0.3)    // Primer tono
    playTone(750, 0.2, 0.3)  // Segundo tono más alto
    playTone(900, 0.4, 0.5)  // Tercer tono más alto y largo
    
    // Repetir la secuencia después de una pausa
    setTimeout(() => {
      playTone(600, 0, 0.3)
      playTone(750, 0.2, 0.3)
      playTone(900, 0.4, 0.5)
    }, 800)
    
  } catch (e) {
    // Fallback: intentar usar el API de notificaciones del navegador
    if ('Notification' in window && Notification.permission === 'granted') {
      // Vibración si está disponible (móviles)
      if ('vibrate' in navigator) {
        navigator.vibrate([200, 100, 200, 100, 200])
      }
    }
    console.log('Sonido de notificación no disponible, usando vibración')
  }
}

function startEditingTime() {
  if (isRunning.value) return
  isEditingTime.value = true
  editingMinutes.value = Math.ceil(timeLeft.value / 60)
  nextTick(() => {
    if (timeInput.value) {
      timeInput.value.focus()
      timeInput.value.select()
    }
  })
}

function saveEditedTime() {
  const minutes = parseInt(editingMinutes.value)
  if (minutes > 0 && minutes <= 999) {
    timeLeft.value = minutes * 60
    playClickSound()
  }
  isEditingTime.value = false
  editingMinutes.value = ''
}

function cancelEditingTime() {
  isEditingTime.value = false
  editingMinutes.value = ''
}

function toggleTimer() {
  playClickSound()
  isRunning.value = !isRunning.value
  if (isRunning.value) {
    startTimer()
  } else {
    clearInterval(interval)
  }
}

function startTimer() {
  interval = setInterval(() => {
    timeLeft.value--
    if (isWorkTime.value) {
      totalWorkTime.value++
    }

    if (timeLeft.value <= 0) {
      clearInterval(interval)
      isRunning.value = false
      handleTimerComplete()
    }
  }, 1000)
}

function handleTimerComplete() {
  if (isWorkTime.value) {
    completedPomodoros.value++
    const isLongBreak = completedPomodoros.value % POMODOROS_BEFORE_LONG_BREAK === 0
    timeLeft.value = isLongBreak ? longBreakMinutes.value * 60 : shortBreakMinutes.value * 60
    currentMode.value = isLongBreak ? 'Descanso Largo' : 'Descanso Corto'

    // Notificación de fin de trabajo
    showNotification('¡Pomodoro Completado!', `Has completado ${completedPomodoros.value} pomodoros. ¡Toma un descanso!`)
    playNotificationSound()

    // Emitir evento de pomodoro completado
    emit('pomodoroCompleted', {
      completedPomodoros: completedPomodoros.value,
      isLongBreak: isLongBreak
    })
  } else {
    timeLeft.value = workTimeMinutes.value * 60
    currentMode.value = 'Trabajo'

    // Notificación de fin de descanso
    showNotification('¡Descanso Terminado!', 'Es hora de volver al trabajo. ¡Tú puedes!')
    playNotificationSound()
  }
  isWorkTime.value = !isWorkTime.value
}

function completeCurrentTask() {
  playClickSound()
  if (props.selectedTask) {
    emit('taskCompleted', props.selectedTask.id)
  }
}

function resetTimer() {
  playClickSound()
  clearInterval(interval)
  isRunning.value = false
  timeLeft.value = workTimeMinutes.value * 60
  isWorkTime.value = true
  currentMode.value = 'Trabajo'
}
</script>

<style scoped>
.pomodoro-container {
  padding: 20px;
  display: flex;
  justify-content: center;
}

.pomodoro-card {
  background: rgba(255, 255, 255, 0.95);
  padding: 30px;
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  text-align: center;
  backdrop-filter: blur(10px);
  max-width: 400px;
  width: 100%;
}

.settings {
  margin-bottom: 30px;
  padding: 20px;
  background: rgba(102, 126, 234, 0.1);
  border-radius: 15px;
  border: 1px solid rgba(102, 126, 234, 0.2);
}

.settings h4 {
  margin: 0 0 15px 0;
  color: #333;
  font-size: 1.2em;
  font-weight: 600;
}

.setting-group {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.setting-group:last-child {
  margin-bottom: 0;
}

.setting-group label {
  font-weight: 500;
  color: #555;
  font-size: 0.9em;
}

.setting-group select {
  padding: 8px 12px;
  border-radius: 8px;
  border: 2px solid #e1e5e9;
  background: white;
  font-size: 0.9em;
  cursor: pointer;
  transition: all 0.3s ease;
  min-width: 60px;
}

.setting-group select:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.setting-group select:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.tomato {
  margin-bottom: 30px;
}

.tomato-body {
  width: 150px;
  height: 180px;
  background: linear-gradient(135deg, #ff6b6b 0%, #ee5a24 100%);
  border-radius: 50% 50% 50% 50% / 60% 60% 40% 40%;
  position: relative;
  margin: 0 auto;
  box-shadow: 0 8px 25px rgba(255, 107, 107, 0.3);
  animation: pulse 2s infinite;
  overflow: hidden;
}

/* Animación simple del tiempo dentro del tomate */
.time-fill {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: linear-gradient(to top,
    rgba(255, 255, 255, 0.8) 0%,
    rgba(255, 255, 255, 0.4) 30%,
    rgba(255, 255, 255, 0.1) 70%,
    rgba(255, 255, 255, 0) 100%);
  border-radius: 0 0 50% 50% / 0 0 40% 40%;
  transition: height 0.5s ease-out;
  pointer-events: none;
}

.time-particles {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  pointer-events: none;
}

.particle {
  position: absolute;
  width: 6px;
  height: 6px;
  background: radial-gradient(circle, rgba(255, 255, 255, 0.9) 0%, rgba(255, 255, 255, 0.3) 70%, transparent 100%);
  border-radius: 50%;
  transition: all 0.8s ease-out;
  animation: float 3s ease-in-out infinite;
}

@keyframes float {
  0%, 100% {
    transform: translateY(0px) scale(1);
  }
  50% {
    transform: translateY(-8px) scale(1.1);
  }
}

.tomato-stem {
  width: 20px;
  height: 30px;
  background: linear-gradient(to bottom, #4caf50, #388e3c);
  position: absolute;
  top: -15px;
  left: 50%;
  transform: translateX(-50%);
  border-radius: 10px 10px 0 0;
}

.tomato-face {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 100%;
  height: 100%;
}

.eyes {
  display: flex;
  justify-content: space-around;
  align-items: center;
  height: 40%;
}

.eye {
  width: 15px;
  height: 15px;
  background: #333;
  border-radius: 50%;
  position: relative;
}

.eye.left {
  left: -10px;
}

.eye.right {
  right: -10px;
}

.mouth {
  width: 30px;
  height: 15px;
  border: 3px solid #333;
  border-top: none;
  border-radius: 0 0 50% 50%;
  margin: 0 auto;
  margin-top: 10px;
  transition: all 0.3s ease;
}

.mouth.smile {
  border-radius: 50% 50% 0 0;
  border-bottom: none;
  border-top: 3px solid #333;
  transform: rotate(180deg);
}

.mouth.rest {
  width: 20px;
  height: 10px;
}

.timer-display h3 {
  color: #333;
  margin-bottom: 10px;
  font-size: 1.2em;
  font-weight: 300;
}

.time {
  font-size: 3em;
  font-weight: bold;
  color: #333;
  margin-bottom: 20px;
  font-family: 'Courier New', monospace;
  cursor: pointer;
  transition: all 0.3s ease;
  user-select: none;
}

.time:hover {
  transform: scale(1.05);
  text-shadow: 0 0 10px rgba(102, 126, 234, 0.3);
}

.time.editing {
  opacity: 0.5;
}

.time-editor {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  margin-bottom: 20px;
}

.time-input {
  width: 80px;
  padding: 10px;
  border-radius: 10px;
  border: 2px solid #667eea;
  font-size: 1.5em;
  font-weight: bold;
  text-align: center;
  font-family: 'Courier New', monospace;
  background: rgba(102, 126, 234, 0.1);
  color: #333;
}

.time-input:focus {
  outline: none;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.3);
  background: white;
}

.progress-bar {
  width: 100%;
  height: 8px;
  background: #e1e5e9;
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 30px;
}

.progress {
  height: 100%;
  background: linear-gradient(90deg, #ff6b6b, #ee5a24);
  border-radius: 4px;
  transition: width 0.3s ease;
}

.controls {
  display: flex;
  gap: 15px;
  margin-bottom: 30px;
  justify-content: center;
}

.control-btn {
  padding: 12px 25px;
  border-radius: 25px;
  border: none;
  font-size: 1em;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  min-width: 100px;
}

.control-btn.active {
  background: linear-gradient(45deg, #ff4757, #ff3838);
  color: white;
  box-shadow: 0 4px 15px rgba(255, 71, 87, 0.3);
}

.control-btn.active:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(255, 71, 87, 0.4);
}

.control-btn:not(.active) {
  background: linear-gradient(45deg, #667eea, #764ba2);
  color: white;
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.3);
}

.control-btn:not(.active):hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.4);
}

.control-btn.reset {
  background: #6c757d;
  color: white;
  box-shadow: 0 4px 15px rgba(108, 117, 125, 0.3);
}

.control-btn.reset:hover {
  background: #5a6268;
  box-shadow: 0 6px 20px rgba(108, 117, 125, 0.4);
}

.stats {
  display: flex;
  justify-content: space-around;
  padding-top: 20px;
  border-top: 1px solid #e1e5e9;
}

.stat {
  text-align: center;
}

.stat-number {
  display: block;
  font-size: 1.5em;
  font-weight: bold;
  color: #333;
}

.stat-label {
  font-size: 0.9em;
  color: #666;
  margin-top: 5px;
}

.task-completion {
  margin-top: 20px;
  padding-top: 20px;
  border-top: 2px solid #e1e5e9;
}

.complete-task-btn {
  width: 100%;
  padding: 15px;
  border-radius: 25px;
  border: none;
  background: linear-gradient(45deg, #4caf50, #45a049);
  color: white;
  font-size: 1.1em;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(76, 175, 80, 0.3);
}

.complete-task-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(76, 175, 80, 0.4);
}

.complete-task-btn:active {
  transform: translateY(0);
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
}
</style>