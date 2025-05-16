<template>
  <div class="timer">
    <h2>{{ isBreak ? 'BREAK TIME' : 'FOCUS TIME' }}</h2>
    <div class="time-display">{{ minutes }}:{{ seconds }}</div>

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
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onUnmounted } from 'vue'

const focusMinutes = ref(25)
const breakMinutes = ref(5)

const FOCUS_DURATION = () => focusMinutes.value * 60
const BREAK_DURATION = () => breakMinutes.value * 60

const time = ref(FOCUS_DURATION())
const isBreak = ref(false)
const running = ref(false)

let timer

const minutes = computed(() => String(Math.floor(time.value / 60)).padStart(2, '0'))
const seconds = computed(() => String(time.value % 60).padStart(2, '0'))

function tick() {
  if (time.value > 0) {
    time.value--
  } else {
    isBreak.value = !isBreak.value
    time.value = isBreak.value ? BREAK_DURATION() : FOCUS_DURATION()
    alert(isBreak.value ? 'Waktunya istirahat!' : 'Waktunya fokus!')
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
}

watch([focusMinutes, breakMinutes], () => {
  if (!running.value) {
    time.value = isBreak.value ? BREAK_DURATION() : FOCUS_DURATION()
  }
})

onUnmounted(() => clearInterval(timer))
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
  /* box-shadow:
    4px 4px 0 #e5989b,
    8px 8px 0 #e5989b; */
  margin-bottom: 1rem;
  font-family: 'Press Start 2P', cursive, monospace;
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

.buttons button {
  background: #f0a04b;
  color: #222;
  font-weight: bold;
  padding: 0.5rem 1rem;
  margin: 0 0.3rem;
  border-radius: 4px;
  box-shadow: 2px 2px 0 #fada7a;
  transition: all 0.2s;
  cursor: pointer;
}

.buttons button.running {
  background: #4caf50;
  box-shadow: 2px 2px 0 #367c39;
}

.buttons button:hover:not(:disabled) {
  filter: brightness(1.1);
}

.settings input:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>
