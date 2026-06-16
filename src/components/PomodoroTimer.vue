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
      <span>🍅 Sesi fokus: {{ completedSessions }}</span>
      <span class="sep">|</span>
      <span>⏱️ Total fokus: {{ totalFocusLabel }}</span>
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
let timer
const minutes = computed(() => String(Math.floor(time.value / 60)).padStart(2, '0'))
const seconds = computed(() => String(time.value % 60).padStart(2, '0'))

// --- INOVASI: progress bar ---
const totalDuration = computed(() => (isBreak.value ? BREAK_DURATION() : FOCUS_DURATION()))
const progressPercent = computed(() => {
  if (totalDuration.value === 0) return 0
  return ((totalDuration.value - time.value) / totalDuration.value) * 100
})

// --- INOVASI: statistik ---
const completedSessions = ref(0)
const totalFocusSeconds = ref(0)
const totalFocusLabel = computed(() => {
  const h = Math.floor(totalFocusSeconds.value / 3600)
  const m = Math.floor((totalFocusSeconds.value % 3600) / 60)
  return h > 0 ? `${h}j ${m}m` : `${m}m`
})

// --- INOVASI: mute toggle ---
const muted = ref(false)
function toggleMute() {
  muted.value = !muted.value
}

// --- INOVASI: suara beep via Web Audio API, tanpa file eksternal ---
let audioCtx
function playBeep() {
  if (muted.value) return
  try {
    if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)()
    const now = audioCtx.currentTime
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

// --- INOVASI: browser notification ---
function notify(title, body) {
  if (!('Notification' in window)) return
  if (Notification.permission === 'granted') {
    new Notification(title, { body })
  }
}
onMounted(() => {
  if ('Notification' in window && Notification.permission === 'default') {
    Notification.requestPermission()
  }
})

// --- INOVASI: judul tab ikut menampilkan countdown ---
watch([time, running], () => {
  document.title = running.value
    ? `${minutes.value}:${seconds.value} - ${isBreak.value ? 'Break' : 'Focus'}`
    : 'POMODORO TO-DO'
})

function tick() {
  if (time.value > 0) {
    time.value--
    if (!isBreak.value) totalFocusSeconds.value++
  } else {
    if (!isBreak.value) completedSessions.value++
    isBreak.value = !isBreak.value
    time.value = isBreak.value ? BREAK_DURATION() : FOCUS_DURATION()
    playBeep()
    notify('Pomodoro Timer', isBreak.value ? 'Waktunya istirahat!' : 'Waktunya fokus!')

    // Timer SELALU berhenti begitu sesi habis.
    // User harus klik START secara manual untuk mulai sesi berikutnya.
    running.value = false
    clearInterval(timer)
  }
}

function toggleTimer() {
  running.value = !running.value
  if (running.value) {
    clearInterval(timer) // cegah dobel interval kalau toggle dipencet cepat
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
  /* Tambahan: berikan padding kanan-kiri agar tidak menempel di layar HP */
  padding: 0 1rem; 
  max-width: 450px;
  margin-left: auto;
  margin-right: auto;
}
.timer h2 {
  font-size: 0.8rem;
  letter-spacing: 3px;
  margin-bottom: 0.5rem;
  color: #ffb4a2;
}
.time-display {
  /* Diubah dari 3rem menjadi clamp agar ukurannya dinamis mengikuti lebar layar */
  font-size: clamp(1.8rem, 8vw, 3rem);
  background: #222;
  color: #eee;
  padding: 1rem; /* disederhanakan agar tidak terlalu memakan ruang */
  border: 4px solid #c599b6;
  border-radius: 6px;
  margin-bottom: 0.7rem;
  font-family: 'Press Start 2P', cursive, monospace;
  box-sizing: border-box;
}
.progress-bar {
  width: 100%;
  height: 10px;
  background: #222;
  border: 2px solid #574964;
  border-radius: 6px;
  overflow: hidden;
  margin-bottom: 1rem;
  box-sizing: border-box;
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
  margin-bottom: 1.2rem;
  display: flex;
  justify-content: center;
  gap: 1rem;
  flex-wrap: wrap; /* Agar input turun ke bawah jika layar sangat sempit */
}
.settings label {
  /* Diubah menjadi clamp agar teks label mengecil di HP jadul */
  font-size: clamp(0.55rem, 2vw, 0.7rem);
  color: #f9f9f9;
  font-family: 'Press Start 2P', cursive, monospace;
  user-select: none;
  display: flex;
  align-items: center;
}
.settings input {
  width: 3.5rem; /* Sedikit dilebarkan agar angka 2 digit aman */
  margin-left: 0.3rem;
  font-size: clamp(0.55rem, 2vw, 0.7rem);
  padding: 0.2rem;
  border-radius: 4px;
  border: 3px solid #eee;
  background: #222;
  color: #eee;
  box-shadow: 3px 3px 0 #444;
  font-family: 'Press Start 2P', cursive, monospace;
  outline: none;
  box-sizing: border-box;
}
.buttons {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 1rem;
  flex-wrap: wrap; /* Tombol akan menyesuaikan diri jika space kurang */
}
.buttons button {
  background: #f0a04b;
  color: #222;
  font-weight: bold;
  /* Menggunakan clamp untuk ukuran teks tombol */
  font-size: clamp(0.7rem, 2.5vw, 0.9rem);
  padding: 0.5rem 0.8rem;
  margin: 0; /* Margin dihapus karena sudah diatur oleh gap */
  border-radius: 4px;
  box-shadow: 2px 2px 0 #fada7a;
  transition: transform 0.1s;
  cursor: pointer;
  white-space: nowrap; /* Teks button tidak akan patah ke bawah */
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
  font-size: 1.2rem !important;
  padding: 0.3rem !important;
}
.stats {
  /* Menggunakan clamp agar teks statistik tidak meluber keluar layar */
  font-size: clamp(0.5rem, 1.8vw, 0.6rem);
  color: #fada7a;
  font-family: 'Press Start 2P', cursive, monospace;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
  gap: 0.3rem;
}
.stats .sep {
  margin: 0 0.2rem;
  opacity: 0.5;
}
.settings input:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* --- RESPONSIVE BREAKPOINT FOR SMALL DEVICES --- */
@media (max-width: 380px) {
  .settings {
    flex-direction: column; /* Label fokus & istirahat bertumpuk vertikal di HP sangat kecil */
    align-items: center;
    gap: 0.5rem;
  }
  .stats .sep {
    display: none; /* Sembunyikan garis pemisah '|' jika mode vertikal */
  }
  .stats {
    flex-direction: column; /* Statistik bertumpuk atas-bawah */
    gap: 0.4rem;
  }
}
</style>
