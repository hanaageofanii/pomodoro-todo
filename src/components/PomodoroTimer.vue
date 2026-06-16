<template>
  <div class="app">

    <!-- Notif Bar -->
    <div v-if="showNotifBar" class="notif-bar">
      <span>Aktifkan notifikasi supaya dapat pengingat otomatis</span>
      <button @click="requestNotif">Aktifkan</button>
    </div>

    <!-- Banner -->
    <div :class="['banner', bannerVisible ? 'show' : '', bannerType]">{{ bannerMsg }}</div>

    <!-- DESKTOP SPLIT GRID LAYOUT -->
    <div class="desktop-grid">
      
      <!-- KOLOM KIRI: MAIN TIMER VISUAL -->
      <div class="card timer-main-card">
        <div :class="['mode-badge', isBreak ? 'break' : '']">
          {{ isBreak ? 'BREAK TIME' : 'FOCUS TIME' }}
        </div>
        
        <div class="time-display-wrapper">
          <div class="time-display">{{ timeDisplay }}</div>
        </div>
        
        <div class="progress-bar">
          <div :class="['progress-fill', isBreak ? 'break' : '']" :style="{ width: progressPct + '%' }"></div>
        </div>
      </div>

      <!-- KOLOM KANAN: CONTROLS, SETTINGS, & AUDIO -->
      <div class="right-column">
        
        <!-- CARD CONTROL & CONFIG -->
        <div class="card control-card">
          <!-- Goal Section -->
          <div class="goal-section">
            <div class="goal-header">
              <span class="goal-label">Target sesi hari ini</span>
              <span class="goal-count">{{ completedSessions }} / {{ goalTarget }}</span>
            </div>
            <div class="goal-bar">
              <div class="goal-fill" :style="{ width: goalPct + '%' }"></div>
            </div>
            <div class="goal-input-row">
              <label>Target:</label>
              <input type="number" v-model.number="goalTarget" min="1" max="20" @change="saveStorage" />
              <span class="sub-lbl">sesi per hari</span>
            </div>
          </div>

          <hr class="divider" />

          <!-- Settings Row -->
          <div class="settings-row">
            <label>⏱️ Fokus: <input type="number" v-model.number="focusMin" min="1" :disabled="running" @change="onFocusMinChange" /> <span class="sub-lbl">mnt</span></label>
            <label>☕ Istirahat: <input type="number" v-model.number="breakMin" min="1" :disabled="running" @change="onBreakMinChange" /> <span class="sub-lbl">mnt</span></label>
          </div>
          
          <!-- Action Buttons -->
          <div class="btn-row">
            <button :class="['btn', 'btn-start', running ? 'running' : '']" @click="toggleTimer">
              {{ running ? 'PAUSE' : 'START' }}
            </button>
            <button class="btn btn-reset" @click="resetTimer">RESET</button>
            <button class="btn btn-mute" @click="toggleMute">{{ muted ? '🔇' : '🔊' }}</button>
          </div>
          
          <hr class="divider" />

          <!-- Quick Stats -->
          <div class="stats-row">
            <div class="stat-item">
              <div class="stat-lbl">Sesi Selesai</div>
              <div class="stat-val">🍅 {{ completedSessions }} Sesi</div>
            </div>
            <div class="stat-item">
              <div class="stat-lbl">Waktu Fokus</div>
              <div class="stat-val">⏱️ {{ totalFocusDisplay }}</div>
            </div>
          </div>
        </div>

        <!-- CARD SPOTIFY -->
        <div class="card spotify-card">
          <div class="card-title">🎧 Spotify Player</div>
          <iframe src="https://open.spotify.com/embed/playlist/37i9dQZF1DX3rxVfibe1L0?utm_source=generator&theme=0" width="100%" height="80" frameBorder="0" allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe>
        </div>

      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

// ─── CONSTANTS ───────────────────────────────────────────────────────
const TODAY = new Date().toISOString().slice(0, 10);

// ─── STATE ───────────────────────────────────────────────────────────
const focusMin  = ref(25);
const breakMin  = ref(5);
const totalSec  = ref(25 * 60);
const isBreak   = ref(false);
const running   = ref(false);
const muted     = ref(false);
const completedSessions = ref(0);
const totalFocusSec     = ref(0);
const goalTarget = ref(4);
const showNotifBar = ref(false);
const bannerVisible = ref(false);
const bannerMsg  = ref('');
const bannerType = ref('');

let timerInterval = null;
let bannerTimeout = null;
let audioCtx = null;

// ─── COMPUTED ────────────────────────────────────────────────────────
const pad = n => String(n).padStart(2, '0');

const timeDisplay = computed(() => {
  const m = Math.floor(totalSec.value / 60);
  const s = totalSec.value % 60;
  return pad(m) + ':' + pad(s);
});

const progressPct = computed(() => {
  const total = isBreak.value ? breakMin.value * 60 : focusMin.value * 60;
  return total > 0 ? ((total - totalSec.value) / total * 100) : 0;
});

const goalPct = computed(() => Math.min((completedSessions.value / goalTarget.value) * 100, 100));

const totalFocusDisplay = computed(() => {
  const h = Math.floor(totalFocusSec.value / 3600);
  const m = Math.floor((totalFocusSec.value % 3600) / 60);
  return h > 0 ? h + 'j ' + m + 'm' : m + 'm';
});

// ─── STORAGE ─────────────────────────────────────────────────────────
function loadStorage() {
  try {
    const d = JSON.parse(localStorage.getItem('pomodoro_data') || '{}');
    goalTarget.value = d.goalTarget || 4;
    const dayData = d.dayData || {};
    const today   = dayData[TODAY] || { sessions: 0, focusSec: 0 };
    completedSessions.value = today.sessions;
    totalFocusSec.value     = today.focusSec;
  } catch(e) {}
}

function saveStorage() {
  try {
    const d = JSON.parse(localStorage.getItem('pomodoro_data') || '{}');
    d.goalTarget = goalTarget.value;
    const dayData = d.dayData || {};
    dayData[TODAY] = { sessions: completedSessions.value, focusSec: totalFocusSec.value };
    d.dayData = dayData;
    localStorage.setItem('pomodoro_data', JSON.stringify(d));
  } catch(e) {}
}

// ─── TIMER ───────────────────────────────────────────────────────────
function toggleTimer() {
  if (!running.value) {
    running.value = true;
    clearInterval(timerInterval);
    timerInterval = setInterval(tick, 1000);
  } else {
    running.value = false;
    clearInterval(timerInterval);
  }
}

function updateTitle(m, s) {
  if (typeof document !== 'undefined') {
    document.title = running.value ? `${pad(m)}:${pad(s)} - ${isBreak.value ? 'Break' : 'Focus'}` : 'POMODORO';
  }
}

function resetTimer() {
  clearInterval(timerInterval);
  running.value  = false;
  isBreak.value  = false;
  totalSec.value = focusMin.value * 60;
  bannerVisible.value = false;
  if (typeof document !== 'undefined') document.title = 'POMODORO';
}

function tick() {
  if (totalSec.value > 0) {
    totalSec.value--;
    if (!isBreak.value) totalFocusSec.value++;
    const m = Math.floor(totalSec.value / 60);
    const s = totalSec.value % 60;
    updateTitle(m, s);
  } else {
    clearInterval(timerInterval);
    running.value = false;
    playBeep();
    if (!isBreak.value) {
      completedSessions.value++;
      saveStorage();
      isBreak.value  = true;
      totalSec.value = breakMin.value * 60;
      if (typeof document !== 'undefined') document.title = 'POMODORO';
      showBanner('☕ Sesi fokus selesai! Waktunya istirahat — tekan START untuk mulai break.', 'break-banner');
      sendNotif('Pomodoro — Fokus Selesai! ☕', 'Waktunya istirahat. Tekan START untuk mulai break.');
    } else {
      saveStorage();
      isBreak.value  = false;
      totalSec.value = focusMin.value * 60;
      if (typeof document !== 'undefined') document.title = 'POMODORO';
      showBanner('✅ Break selesai! Tekan START untuk sesi fokus baru.', 'done-banner');
      sendNotif('Pomodoro — Break Selesai! ✅', 'Timer berhenti. Tekan START untuk sesi fokus baru.');
    }
  }
}

function showBanner(msg, type) {
  bannerMsg.value     = msg;
  bannerType.value    = type;
  bannerVisible.value = true;
  clearTimeout(bannerTimeout);
  bannerTimeout = setTimeout(() => { bannerVisible.value = false; }, 7000);
}

function onFocusMinChange() {
  if (!running.value && !isBreak.value) totalSec.value = focusMin.value * 60;
}
function onBreakMinChange() {
  if (!running.value && isBreak.value) totalSec.value = breakMin.value * 60;
}

// ─── AUDIO ───────────────────────────────────────────────────────────
function getAudioCtx() {
  if (!audioCtx && typeof window !== 'undefined') {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  }
  return audioCtx;
}

function toggleMute() { muted.value = !muted.value; }

function playBeep() {
  if (muted.value) return;
  const ctx = getAudioCtx();
  if (!ctx) return;
  try {
    const now = ctx.currentTime;
    for (let i = 0; i < 3; i++) {
      const o = ctx.createOscillator(); const g = ctx.createGain();
      o.type = 'sine'; o.frequency.value = isBreak.value ? 660 : 880;
      g.gain.setValueAtTime(0.0001, now + i * 0.3); g.gain.exponentialRampToValueAtTime(0.3, now + i * 0.3 + 0.02);
      g.gain.exponentialRampToValueAtTime(0.0001, now + i * 0.3 + 0.25);
      o.connect(g); g.connect(ctx.destination); o.start(now + i * 0.3); o.stop(now + i * 0.3 + 0.3);
    }
  } catch(e) {}
}

function sendNotif(title, body) {
  if (typeof window !== 'undefined' && 'Notification' in window && Notification.permission === 'granted') {
    new Notification(title, { body });
  }
}

function requestNotif() {
  if (typeof window !== 'undefined' && 'Notification' in window) {
    Notification.requestPermission().then(p => { if (p === 'granted') showNotifBar.value = false; });
  }
}

// ─── LIFECYCLE ───────────────────────────────────────────────────────
onMounted(() => {
  loadStorage();
  if (typeof window !== 'undefined' && 'Notification' in window && Notification.permission === 'default') {
    showNotifBar.value = true;
  }
});

onUnmounted(() => {
  clearInterval(timerInterval);
});
</script>

<style scoped>
.app {
  --bg: #0d0d1a;
  --surface: #14142a;
  --surface2: #1c1c35;
  --border: #2e2e52;
  --text: #f0eeff;
  --muted: #8b8aaa;
  --accent: #f0a04b;
  --accent2: #c599b6;
  --green: #4caf50;
  --pixel: 'Press Start 2P', monospace;
  --sans: 'Inter', sans-serif;
  --radius: 10px;
  --radius-lg: 18px;
}

.app, .app * { 
  box-sizing: border-box; 
  margin: 0; 
  padding: 0;
}

/* Dioptimalkan khusus layar Desktop/Laptop */
.app { 
  font-family: var(--sans); 
  color: var(--text);
  max-width: 850px; 
  margin: 4vh auto; 
  padding: 0 1.5rem;
  width: 100%;
}

.header { text-align: left; margin-bottom: 2rem; border-bottom: 1px solid var(--border); padding-bottom: 1rem; }
.header h1 { font-family: var(--pixel); font-size: 1.4rem; color: var(--accent2); letter-spacing: 4px; }
.header p { font-size: 13px; color: var(--muted); margin-top: 6px; }

/* LAYOUT GRID UTAMA DESKTOP */
.desktop-grid { 
  display: grid; 
  grid-template-columns: 1.2fr 1fr; 
  gap: 1.5rem; 
  align-items: start;
}

@media (max-width: 700px) {
  .desktop-grid { grid-template-columns: 1fr; }
}

.card { 
  background: var(--surface); 
  border: 1px solid var(--border); 
  border-radius: var(--radius-lg); 
  padding: 1.5rem; 
  width: 100%; 
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.2);
}
.card-title { font-family: var(--pixel); font-size: 0.6rem; letter-spacing: 2px; color: var(--muted); margin-bottom: 1rem; text-transform: uppercase; }

/* KOLOM KIRI: TIMER BESAR */
.timer-main-card { 
  display: flex; 
  flex-direction: column; 
  align-items: center; 
  justify-content: center;
  padding: 3rem 1.5rem;
  min-height: 380px;
}

.mode-badge { 
  font-family: var(--pixel); 
  font-size: 0.6rem; 
  letter-spacing: 3px; 
  padding: 6px 14px; 
  border-radius: 6px; 
  margin-bottom: 2rem; 
  background: rgba(197,153,182,0.12); 
  color: var(--accent2); 
  border: 1px solid rgba(197,153,182,0.25); 
}
.mode-badge.break { background: rgba(76,175,80,0.12); color: #7ee882; border-color: rgba(76,175,80,0.25); }

.time-display-wrapper {
  width: 100%;
  display: flex;
  justify-content: center;
  margin-bottom: 2rem;
}

.time-display { 
  font-family: var(--pixel); 
  font-size: 4rem; 
  color: var(--text); 
  background: #05050f; 
  border: 4px solid var(--accent2); 
  border-radius: var(--radius); 
  padding: 1.5rem 0; 
  width: 100%;
  max-width: 340px;
  text-align: center;
  letter-spacing: 2px;
  box-shadow: inset 0 4px 12px rgba(0,0,0,0.5);
}

.progress-bar { width: 100%; max-width: 340px; height: 10px; background: var(--surface2); border-radius: 99px; overflow: hidden; }
.progress-fill { height: 100%; background: var(--accent); border-radius: 99px; transition: width 1s linear; }
.progress-fill.break { background: var(--green); }

/* KOLOM KANAN: KONTROL & SPOTIFY */
.right-column {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.control-card {
  padding: 1.5rem;
}

.divider {
  border: 0;
  height: 1px;
  background: var(--border);
  margin: 1.25rem 0;
}

.sub-lbl { font-size: 11px; color: var(--muted); }

/* GOALS SECTION */
.goal-section { width: 100%; }
.goal-header { display: flex; align-items: center; justify-content: space-between; margin-bottom: 0.5rem; }
.goal-label { font-size: 13px; font-weight: 500; color: var(--text); }
.goal-count { font-family: var(--pixel); font-size: 0.65rem; color: var(--accent); }
.goal-bar { height: 8px; background: var(--surface2); border-radius: 99px; overflow: hidden; }
.goal-fill { height: 100%; background: linear-gradient(90deg, var(--accent2), var(--accent)); border-radius: 99px; transition: width 0.4s ease; }
.goal-input-row { display: flex; align-items: center; gap: 8px; margin-top: 0.75rem; font-size: 12px; color: var(--muted); }
.goal-input-row input { width: 50px; background: var(--surface2); border: 1px solid var(--border); color: var(--text); border-radius: 5px; padding: 4px; font-size: 12px; text-align: center; outline: none; font-family: var(--pixel); }

/* SETTINGS INPUTS */
.settings-row { display: flex; flex-direction: column; gap: 0.75rem; width: 100%; }
.settings-row label { font-size: 13px; color: var(--text); display: flex; align-items: center; gap: 8px; }
.settings-row input[type=number] { width: 55px; background: var(--surface2); border: 1px solid var(--border); color: var(--text); border-radius: 5px; padding: 5px; font-size: 12px; font-family: var(--pixel); text-align: center; outline: none; margin-left: auto; }
.settings-row input:disabled { opacity: 0.4; cursor: not-allowed; }

/* CONTROLS BUTTONS */
.btn-row { display: flex; gap: 0.5rem; margin-top: 1.25rem; width: 100%; }
.btn { font-family: var(--pixel); font-size: 0.6rem; padding: 0.75rem 1rem; border-radius: 6px; border: none; cursor: pointer; transition: transform 0.1s, opacity 0.1s; letter-spacing: 1px; }
.btn:hover { transform: translateY(-2px); opacity: 0.95; }
.btn-start { flex: 2; background: var(--accent); color: #111; font-weight: bold; }
.btn-start.running { background: var(--green); color: #fff; }
.btn-reset { flex: 1; background: var(--surface2); color: var(--text); border: 1px solid var(--border); }
.btn-mute { background: transparent; border: 1px solid var(--border); color: var(--text); font-size: 1rem; padding: 0.5rem 0.75rem; border-radius: 6px; }

/* DESKTOP STATS TABS */
.stats-row { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; text-align: left; }
.stat-lbl { font-size: 11px; color: var(--muted); margin-bottom: 4px; text-transform: uppercase; letter-spacing: 0.5px; }
.stat-val { font-size: 14px; font-weight: 600; color: var(--accent); }

/* SPOTIFY */
.spotify-card { padding: 1rem; }
.spotify-card iframe { border-radius: var(--radius); display: block; }

/* NOTIF & BANNERS BAR */
.banner { display: none; padding: 0.6rem 1rem; border-radius: var(--radius); font-size: 12px; text-align: center; margin-bottom: 1rem; font-weight: 500; width: 100%; }
.banner.show { display: block; }
.banner.break-banner { background: rgba(76,175,80,0.12); color: #7ee882; border: 1px solid rgba(76,175,80,0.2); }
.banner.done-banner { background: rgba(240,160,75,0.12); color: var(--accent); border: 1px solid rgba(240,160,75,0.2); }

.notif-bar { background: rgba(91,155,213,0.1); border: 1px solid rgba(91,155,213,0.25); border-radius: var(--radius); padding: 10px 14px; display: flex; align-items: center; justify-content: space-between; font-size: 12px; color: #80b8e8; margin-bottom: 1rem; width: 100%; gap: 10px; }
.notif-bar button { font-size: 11px; background: rgba(91,155,213,0.18); border: 1px solid rgba(91,155,213,0.3); color: #80b8e8; border-radius: 4px; padding: 4px 12px; cursor: pointer; transition: background 0.2s; }
.notif-bar button:hover { background: rgba(91,155,213,0.3); }
</style>
