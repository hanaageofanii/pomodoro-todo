<template>
  <div class="timer">
    <h2>{{ isBreak ? 'BREAK TIME' : 'FOCUS TIME' }}</h2>
    <div class="time-display">{{ minutes }}:{{ seconds }}</div>
    <div class="progress-bar">
      <div class="progress-fill" :class="{ break: isBreak }" :style="{ width: progressPercent + '%' }"></div>
    </div>
    <div class="settings">
      <label>
        Fokus (menit):
        <input type="number" min="1" v-model.number="focusMinutes" :disabled="running" />
      </label>
      <label>
        Istirahat (menit):
        <input type="number" min="1" v-model.number="breakMinutes" :disabled="running" />
      </label>
    </div>
    <div class="buttons">
      <button @click="toggleTimer" :class="{ running: running }">
        {{ running ? 'PAUSE' : 'START' }}
      </button>
      <button @click="resetTimer">RESET</button>
      <button @click="toggleMute" class="mute-btn">{{ muted ? '🔇' : '🔊' }}</button>
    </div>
    <div class="stats">
      <span>🍅 Sesi selesai: {{ completedSessions }}</span>
    </div>
  </div>
  <iframe
    style="border-radius: 6px"
    src="https://open.spotify.com/embed/playlist/37i9dQZF1DX3rxVfibe1L0?utm_source=generator&theme=0"
    width="100%"
    height="80"
    frameBorder="0"
    allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture"
    loading="lazy"
  >
  </iframe>
</template>
<script setup>
import { ref, computed, watch, onUnmounted, onMounted } from 'vue'

const focusMinutes = ref(25)
const breakMinutes = ref(5)
const FOCUS_DURATION = () => focusMinutes.value * 60
const BREAK_DURATION = () => breakMinutes.value * 60
const time = ref(FOCUS_DURATION())
const isBreak = ref(false)
const running = ref(false)
const muted = ref(false)
const completedSessions = ref(0)
let timer

const minutes = computed(() => String(Math.floor(time.value / 60)).padStart(2, '0'))
const seconds = computed(() => String(time.value % 60).padStart(2, '0'))

const totalDuration = computed(() => (isBreak.value ? BREAK_DURATION() : FOCUS_DURATION()))
const progressPercent = computed(() => {
  if (totalDuration.value === 0) return 0
  return ((totalDuration.value - time.value) / totalDuration.value) * 100
})

// --- Audio: bikin bunyi beep pakai Web Audio API, tanpa file eksternal ---
let audioCtx
function playBeep() {
  if (muted.value) return
  try {
    if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)()
    const now = audioCtx.currentTime
    // 3 beep berurutan
    for (let i = 0; i < 3; i++) {
      const osc = audioCtx.createOscillator()
      const gain = audioCtx.createGain()
      osc.type = 'sine'
      osc.frequency.value = isBreak.value ? 660 : 880
      gain.gain.setValueAtTime(0.0001, now + i * 0.3)
      gain.gain.exponentialRampToValueAtTime(0.3, now + i * 0.3 + 0.02)
      gain.gain.exponentialRampToValueAtTime(0.0001, now + i * 0.3 + 0.25)
      osc.connect(gain)
      gain.connect(audioCtx.destination)
      osc.start(now + i * 0.3)
      osc.stop(now + i * 0.3 + 0.3)
    }
  } catch (e) {
    console.warn('Audio gagal diputar:', e)
  }
}

function toggleMute() {
  muted.value = !muted.value
}

// --- Browser Notification ---
function notify(title, body) {
  if (!('Notification' in window)) return
  if (Notification.permission === 'granted') {
    new Notification(title, { body, icon: '🍅' })
  }
}

onMounted(() => {
  if ('Notification' in window && Notification.permission === 'default') {
    Notification.requestPermission()
  }
  // update title tab biar terlihat walau minimize
})

watch([time, running], () => {
  document.title = running.value
    ? `${minutes.value}:${seconds.value} - ${isBreak.value ? 'Break' : 'Focus'}`
    : 'POMODORO TO-DO'
})

function tick() {
  if (time.value > 0) {
    time.value--
  } else {
    if (!isBreak.value) completedSessions.value++
    isBreak.value = !isBreak.value
    time.value = isBreak.value ? BREAK_DURATION() : FOCUS_DURATION()
    playBeep()
    const msg = isBreak.value ? 'Waktunya istirahat!' : 'Waktunya fokus!'
    notify('Pomodoro Timer', msg)
  }
}

function toggleTimer() {
  running.value = !running.value
  if (running.value) {
    timer = setInterval(tick, 1000)
  } else {
    clearInterval(timer)
  }
}

function resetTimer() {
  clearInterval(timer)
  time.value = isBreak.value ? BREAK_DURATION() : FOCUS_DURATION()
  running.value = false
  document.title = 'POMODORO TO-DO'
}

watch([focusMinutes, breakMinutes], () => {
  if (!running.value) {
    time.value = isBreak.value ? BREAK_DURATION() : FOCUS_DURATION()
  }
})

onUnmounted(() => {
  clearInterval(timer)
  document.title = 'POMODORO TO-DO'
})
</script>
<style scoped>
.timer {
  text-align: center;
  margin-bottom: 2rem;
  user-select: none;
}
.timer h2 {
  font-size: 0.8rem;
  letter-spacing: 3px;
  margin-bottom: 0.5rem;
  color: #ffb4a2;
}
.time-display {
  font-size: 3rem;
  background: #222;
  color: #eee;
  padding: 1rem 2rem;
  border: 4px solid #c599b6;
  border-radius: 6px;
  margin-bottom: 0.7rem;
  font-family: 'Press Start 2P', cursive, monospace;
}
.progress-bar {
  width: 100%;
  height: 10px;
  background: #222;
  border: 2px solid #574964;
  border-radius: 6px;
  overflow: hidden;
  margin-bottom: 1rem;
}
.progress-fill {
  height: 100%;
  background: #f0a04b;
  transition: width 1s linear;
}
.progress-fill.break {
  background: #4caf50;
}
.settings {
  margin-bottom: 1rem;
  display: flex;
  justify-content: center;
  gap: 1rem;
}
.settings label {
  font-size: 0.7rem;
  color: #f9f9f9;
  font-family: 'Press Start 2P', cursive, monospace;
  user-select: none;
}
.settings input {
  width: 3rem;
  margin-left: 0.3rem;
  font-size: 0.7rem;
  padding: 0.2rem;
  border-radius: 4px;
  border: 3px solid #eee;
  background: #222;
  color: #eee;
  box-shadow: 3px 3px 0 #444;
  font-family: 'Press Start 2P', cursive, monospace;
  outline: none;
}
.buttons {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 0.7rem;
}
.buttons button {
  background: #f0a04b;
  color: #222;
  font-weight: bold;
  padding: 0.5rem 1rem;
  margin: 0 0.3rem;
  border-radius: 4px;
  box-shadow: 2px 2px 0 #fada7a;
  transition: transform 0.1s;
  cursor: pointer;
}
.buttons button.running {
  background: #4caf50;
  box-shadow: 2px 2px 0 #367c39;
}
.buttons button:hover:not(:disabled) {
  filter: brightness(1.1);
}
.buttons button:hover {
  transform: translateY(-2px);
}
.mute-btn {
  background: transparent !important;
  box-shadow: none !important;
  font-size: 1.2rem;
  padding: 0.3rem !important;
}
.stats {
  font-size: 0.65rem;
  color: #fada7a;
  font-family: 'Press Start 2P', cursive, monospace;
}
.settings input:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>
