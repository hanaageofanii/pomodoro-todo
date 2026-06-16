<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>POMODORO</title>
<link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&family=Inter:wght@400;500;600&display=swap" rel="stylesheet" />
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
  :root {
    --bg: #0d0d1a;
    --surface: #14142a;
    --surface2: #1c1c35;
    --border: #2e2e52;
    --text: #f0eeff;
    --muted: #8b8aaa;
    --accent: #f0a04b;
    --accent2: #c599b6;
    --green: #4caf50;
    --red: #e85d5d;
    --blue: #5b9bd5;
    --pixel: 'Press Start 2P', monospace;
    --sans: 'Inter', sans-serif;
    --radius: 8px;
    --radius-lg: 14px;
  }
  html { font-size: 16px; }
  body { font-family: var(--sans); background: var(--bg); color: var(--text); min-height: 100vh; }

  .app { max-width: 900px; margin: 0 auto; padding: 1.5rem 1rem 3rem; }
  .header { text-align: center; margin-bottom: 1.5rem; }
  .header h1 { font-family: var(--pixel); font-size: clamp(0.8rem, 2.5vw, 1.1rem); color: var(--accent2); letter-spacing: 4px; }
  .header p { font-size: 12px; color: var(--muted); margin-top: 6px; }

  .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; }
  @media (max-width: 680px) { .grid { grid-template-columns: 1fr; } }

  .card { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius-lg); padding: 1.25rem; }
  .card-title { font-family: var(--pixel); font-size: 0.55rem; letter-spacing: 2px; color: var(--muted); margin-bottom: 1rem; text-transform: uppercase; }

  .timer-card { text-align: center; }
  .mode-badge { display: inline-block; font-family: var(--pixel); font-size: 0.5rem; letter-spacing: 3px; padding: 4px 10px; border-radius: 4px; margin-bottom: 0.75rem; background: rgba(197,153,182,0.15); color: var(--accent2); border: 1px solid rgba(197,153,182,0.3); }
  .mode-badge.break { background: rgba(76,175,80,0.15); color: #7ee882; border-color: rgba(76,175,80,0.3); }
  .time-display { font-family: var(--pixel); font-size: clamp(2rem, 8vw, 3.2rem); color: var(--text); background: #090918; border: 3px solid var(--accent2); border-radius: var(--radius); padding: 0.75rem 1.5rem; display: inline-block; margin-bottom: 0.75rem; min-width: 180px; }
  .progress-bar { width: 100%; height: 8px; background: var(--surface2); border-radius: 99px; overflow: hidden; margin-bottom: 1rem; }
  .progress-fill { height: 100%; background: var(--accent); border-radius: 99px; transition: width 1s linear; }
  .progress-fill.break { background: var(--green); }

  .settings-row { display: flex; justify-content: center; gap: 1rem; flex-wrap: wrap; margin-bottom: 1rem; }
  .settings-row label { font-size: 11px; color: var(--muted); display: flex; align-items: center; gap: 6px; }
  .settings-row input[type=number] { width: 52px; background: var(--surface2); border: 1px solid var(--border); color: var(--text); border-radius: 5px; padding: 4px 6px; font-size: 12px; font-family: var(--pixel); text-align: center; outline: none; }
  .settings-row input:disabled { opacity: 0.4; }

  .btn-row { display: flex; justify-content: center; align-items: center; gap: 0.5rem; flex-wrap: wrap; margin-bottom: 0.75rem; }
  .btn { font-family: var(--pixel); font-size: 0.55rem; padding: 0.55rem 1.1rem; border-radius: 5px; border: none; cursor: pointer; transition: transform 0.1s, opacity 0.1s; letter-spacing: 1px; }
  .btn:hover { transform: translateY(-2px); opacity: 0.9; }
  .btn-start { background: var(--accent); color: #111; }
  .btn-start.running { background: var(--green); color: #fff; }
  .btn-reset { background: var(--surface2); color: var(--muted); border: 1px solid var(--border); }
  .btn-mute { background: transparent; border: 1px solid var(--border); color: var(--text); font-size: 0.9rem; padding: 0.4rem 0.6rem; border-radius: 5px; }
  .btn-icon { background: transparent; border: 1px solid var(--border); color: var(--muted); font-size: 0.9rem; padding: 0.4rem 0.7rem; border-radius: 5px; cursor: pointer; transition: all 0.15s; }
  .btn-icon:hover { border-color: var(--accent); color: var(--accent); }

  .stats-row { display: flex; justify-content: center; gap: 1.25rem; flex-wrap: wrap; }
  .stat-item { text-align: center; }
  .stat-val { font-family: var(--pixel); font-size: 0.75rem; color: var(--accent); }
  .stat-lbl { font-size: 10px; color: var(--muted); margin-top: 3px; }

  .banner { display: none; padding: 0.5rem 1rem; border-radius: var(--radius); font-size: 12px; text-align: center; margin-bottom: 0.75rem; font-weight: 500; }
  .banner.show { display: block; }
  .banner.break-banner { background: rgba(76,175,80,0.15); color: #7ee882; border: 1px solid rgba(76,175,80,0.3); }
  .banner.done-banner { background: rgba(240,160,75,0.15); color: var(--accent); border: 1px solid rgba(240,160,75,0.3); }

  .goal-section { margin-bottom: 1rem; }
  .goal-header { display: flex; align-items: center; justify-content: space-between; margin-bottom: 0.4rem; }
  .goal-label { font-size: 12px; color: var(--muted); }
  .goal-count { font-family: var(--pixel); font-size: 0.6rem; color: var(--accent); }
  .goal-bar { height: 6px; background: var(--surface2); border-radius: 99px; overflow: hidden; }
  .goal-fill { height: 100%; background: linear-gradient(90deg, var(--accent2), var(--accent)); border-radius: 99px; transition: width 0.4s ease; }
  .goal-input-row { display: flex; align-items: center; gap: 8px; margin-top: 0.5rem; }
  .goal-input-row label { font-size: 11px; color: var(--muted); white-space: nowrap; }
  .goal-input-row input { width: 50px; background: var(--surface2); border: 1px solid var(--border); color: var(--text); border-radius: 5px; padding: 3px 6px; font-size: 12px; text-align: center; outline: none; font-family: var(--pixel); }

  .streak-display { display: flex; align-items: center; gap: 10px; margin-bottom: 0.75rem; }
  .streak-num { font-family: var(--pixel); font-size: 1.5rem; color: var(--accent); }
  .streak-info { }
  .streak-title { font-size: 13px; font-weight: 600; color: var(--text); }
  .streak-sub { font-size: 11px; color: var(--muted); margin-top: 2px; }
  .streak-dots { display: flex; gap: 5px; flex-wrap: wrap; }
  .streak-dot { width: 22px; height: 22px; border-radius: 4px; background: var(--surface2); border: 1px solid var(--border); display: flex; align-items: center; justify-content: center; font-size: 10px; }
  .streak-dot.done { background: rgba(240,160,75,0.25); border-color: var(--accent); }
  .streak-dot.today { background: rgba(197,153,182,0.25); border-color: var(--accent2); }

  .todo-input-row { display: flex; gap: 6px; margin-bottom: 0.75rem; }
  .todo-input { flex: 1; background: var(--surface2); border: 1px solid var(--border); color: var(--text); border-radius: var(--radius); padding: 8px 12px; font-size: 13px; outline: none; font-family: var(--sans); transition: border 0.15s; }
  .todo-input:focus { border-color: var(--accent2); }
  .todo-input::placeholder { color: var(--muted); }
  .btn-add { background: var(--accent2); color: #111; border: none; border-radius: var(--radius); padding: 8px 14px; font-size: 18px; cursor: pointer; font-weight: bold; transition: opacity 0.15s; }
  .btn-add:hover { opacity: 0.85; }
  .todo-list { list-style: none; display: flex; flex-direction: column; gap: 5px; max-height: 220px; overflow-y: auto; }
  .todo-item { display: flex; align-items: center; gap: 8px; background: var(--surface2); border: 1px solid var(--border); border-radius: var(--radius); padding: 8px 10px; transition: opacity 0.2s; }
  .todo-item.done { opacity: 0.5; }
  .todo-cb { width: 17px; height: 17px; accent-color: var(--accent); cursor: pointer; flex-shrink: 0; }
  .todo-text { flex: 1; font-size: 13px; cursor: pointer; }
  .todo-item.done .todo-text { text-decoration: line-through; color: var(--muted); }
  .todo-del { background: none; border: none; color: var(--muted); font-size: 15px; cursor: pointer; padding: 0 2px; }
  .todo-del:hover { color: var(--red); }
  .todo-empty { font-size: 12px; color: var(--muted); text-align: center; padding: 1rem; }

  .log-list { list-style: none; max-height: 200px; overflow-y: auto; display: flex; flex-direction: column; gap: 4px; }
  .log-item { display: flex; align-items: center; gap: 10px; background: var(--surface2); border-radius: var(--radius); padding: 7px 10px; font-size: 12px; }
  .log-time { font-family: var(--pixel); font-size: 0.5rem; color: var(--accent2); min-width: 50px; }
  .log-desc { flex: 1; color: var(--text); }
  .log-dur { font-size: 11px; color: var(--muted); }
  .log-empty { font-size: 12px; color: var(--muted); text-align: center; padding: 1rem; }
  .log-actions { display: flex; justify-content: flex-end; margin-top: 0.5rem; }

  .ambient-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 6px; margin-bottom: 0.75rem; }
  @media (max-width: 380px) { .ambient-grid { grid-template-columns: repeat(2, 1fr); } }
  .ambient-btn { background: var(--surface2); border: 1px solid var(--border); border-radius: var(--radius); padding: 0.5rem 0.3rem; cursor: pointer; text-align: center; transition: all 0.15s; color: var(--muted); font-size: 11px; }
  .ambient-btn:hover { border-color: var(--accent2); color: var(--text); }
  .ambient-btn.active { background: rgba(197,153,182,0.15); border-color: var(--accent2); color: var(--accent2); }
  .ambient-btn .amb-icon { font-size: 20px; display: block; margin-bottom: 3px; }
  .vol-row { display: flex; align-items: center; gap: 8px; }
  .vol-row label { font-size: 11px; color: var(--muted); }
  .vol-row input[type=range] { flex: 1; accent-color: var(--accent2); }

  .spotify-section { grid-column: 1 / -1; }
  .spotify-section iframe { border-radius: var(--radius); }

  .notif-bar { background: rgba(91,155,213,0.12); border: 1px solid rgba(91,155,213,0.3); border-radius: var(--radius); padding: 8px 14px; display: flex; align-items: center; justify-content: space-between; font-size: 12px; color: #80b8e8; margin-bottom: 1rem; }
  .notif-bar button { font-size: 11px; background: rgba(91,155,213,0.2); border: 1px solid rgba(91,155,213,0.4); color: #80b8e8; border-radius: 4px; padding: 3px 10px; cursor: pointer; }

  ::-webkit-scrollbar { width: 4px; }
  ::-webkit-scrollbar-track { background: var(--surface2); }
  ::-webkit-scrollbar-thumb { background: var(--border); border-radius: 2px; }
</style>
</head>
<body>
<div id="app">
  <div class="app">
    <div class="header">
      <h1>🍅 POMODORO</h1>
      <p>Fokus sepenuhnya. Istirahat sungguh-sungguh.</p>
    </div>

    <!-- Notif Bar -->
    <div v-if="showNotifBar" class="notif-bar">
      <span>Aktifkan notifikasi supaya dapat pengingat otomatis</span>
      <button @click="requestNotif">Aktifkan</button>
    </div>

    <!-- Banner -->
    <div :class="['banner', bannerVisible ? 'show' : '', bannerType]">{{ bannerMsg }}</div>

    <div class="grid">
      <!-- TIMER CARD -->
      <div class="card timer-card" style="grid-column: 1 / -1">
        <div :class="['mode-badge', isBreak ? 'break' : '']">{{ isBreak ? 'BREAK TIME' : 'FOCUS TIME' }}</div>
        <div>
          <div class="time-display">{{ timeDisplay }}</div>
        </div>
        <div class="progress-bar">
          <div :class="['progress-fill', isBreak ? 'break' : '']" :style="{ width: progressPct + '%' }"></div>
        </div>

        <!-- Goal -->
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
            <span style="font-size:11px;color:var(--muted)">sesi per hari</span>
          </div>
        </div>

        <div class="settings-row">
          <label>Fokus (mnt): <input type="number" v-model.number="focusMin" min="1" :disabled="running" @change="onFocusMinChange" /></label>
          <label>Istirahat (mnt): <input type="number" v-model.number="breakMin" min="1" :disabled="running" @change="onBreakMinChange" /></label>
        </div>
        <div class="btn-row">
          <button :class="['btn', 'btn-start', running ? 'running' : '']" @click="toggleTimer">{{ running ? 'PAUSE' : 'START' }}</button>
          <button class="btn btn-reset" @click="resetTimer">RESET</button>
          <button class="btn btn-mute" @click="toggleMute">{{ muted ? '🔇' : '🔊' }}</button>
        </div>
        <div class="stats-row">
          <div class="stat-item"><div class="stat-val">{{ completedSessions }}</div><div class="stat-lbl">🍅 Sesi</div></div>
          <div class="stat-item"><div class="stat-val">{{ totalFocusDisplay }}</div><div class="stat-lbl">⏱ Total fokus</div></div>
          <div class="stat-item"><div class="stat-val">{{ streakCount }}</div><div class="stat-lbl">🔥 Streak</div></div>
        </div>
      </div>

      <!-- TODO LIST -->
      <div class="card">
        <div class="card-title">📋 To-Do List</div>
        <div class="todo-input-row">
          <input class="todo-input" v-model="todoInputVal" type="text" placeholder="Tambah tugas..." @keydown.enter="addTodo" maxlength="80" />
          <button class="btn-add" @click="addTodo">+</button>
        </div>
        <ul class="todo-list">
          <li v-if="todoItems.length === 0" class="todo-empty">Belum ada tugas. Tambahkan sekarang!</li>
          <li v-for="t in todoItems" :key="t.id" :class="['todo-item', t.done ? 'done' : '']">
            <input class="todo-cb" type="checkbox" :checked="t.done" @change="toggleTodo(t.id)" />
            <span class="todo-text" @click="toggleTodo(t.id)">{{ t.text }}</span>
            <button class="todo-del" @click="deleteTodo(t.id)">✕</button>
          </li>
        </ul>
      </div>

      <!-- SESSION LOG -->
      <div class="card">
        <div class="card-title">📜 Riwayat Sesi</div>
        <ul class="log-list">
          <li v-if="sessionLog.length === 0" class="log-empty">Belum ada sesi hari ini.</li>
          <li v-for="(l, i) in reversedLog" :key="i" class="log-item">
            <span class="log-time">{{ l.time }}</span>
            <span class="log-desc">{{ l.type }}</span>
            <span class="log-dur">{{ l.dur }}</span>
          </li>
        </ul>
        <div class="log-actions">
          <button class="btn-icon" @click="exportCSV" style="font-size:12px;padding:5px 10px;">⬇ Export CSV</button>
        </div>
      </div>

      <!-- STREAK -->
      <div class="card">
        <div class="card-title">🔥 Streak Harian</div>
        <div class="streak-display">
          <div class="streak-num">{{ streakCount }}</div>
          <div class="streak-info">
            <div class="streak-title">{{ streakTitle }}</div>
            <div class="streak-sub">{{ streakSub }}</div>
          </div>
        </div>
        <div class="streak-dots">
          <div v-for="dot in streakDots" :key="dot.date" :class="['streak-dot', dot.cls]" :title="dot.date">{{ dot.icon }}</div>
        </div>
      </div>

      <!-- AMBIENT SOUND -->
      <div class="card">
        <div class="card-title">🎵 Ambient Sound</div>
        <div class="ambient-grid">
          <button v-for="amb in ambientOptions" :key="amb.key" :class="['ambient-btn', activeAmbient === amb.key ? 'active' : '']" @click="setAmbient(amb.key)">
            <span class="amb-icon">{{ amb.icon }}</span>{{ amb.label }}
          </button>
        </div>
        <div class="vol-row">
          <label>Volume</label>
          <input type="range" v-model.number="ambVolume" min="0" max="1" step="0.05" @input="setAmbientVolume" />
          <span style="font-size:11px;color:var(--muted)">{{ Math.round(ambVolume * 100) }}%</span>
        </div>
      </div>

      <!-- SPOTIFY -->
      <div class="card spotify-section">
        <div class="card-title">🎧 Spotify — Lo-fi Mix</div>
        <iframe src="https://open.spotify.com/embed/playlist/37i9dQZF1DX3rxVfibe1L0?utm_source=generator&theme=0" width="100%" height="80" frameBorder="0" allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe>
      </div>
    </div>
  </div>
</div>

<script>
const { createApp, ref, computed, onMounted, onUnmounted } = Vue;

createApp({
  setup() {
    // ─── CONSTANTS ───────────────────────────────────────────────────────
    const TODAY = new Date().toISOString().slice(0, 10);

    const ambientOptions = [
      { key: 'off',    icon: '🔇', label: 'Mati' },
      { key: 'rain',   icon: '🌧️', label: 'Hujan' },
      { key: 'white',  icon: '🌊', label: 'White Noise' },
      { key: 'cafe',   icon: '☕', label: 'Kafe' },
      { key: 'forest', icon: '🌿', label: 'Hutan' },
      { key: 'fire',   icon: '🔥', label: 'Perapian' },
    ];

    // ─── STATE ───────────────────────────────────────────────────────────
    const focusMin  = ref(25);
    const breakMin  = ref(5);
    const totalSec  = ref(25 * 60);
    const isBreak   = ref(false);
    const running   = ref(false);
    const muted     = ref(false);
    const completedSessions = ref(0);
    const totalFocusSec     = ref(0);
    const todoItems  = ref([]);
    const todoInputVal = ref('');
    const sessionLog = ref([]);
    const goalTarget = ref(4);
    const activeAmbient = ref('off');
    const ambVolume  = ref(0.3);
    const showNotifBar = ref(false);
    const bannerVisible = ref(false);
    const bannerMsg  = ref('');
    const bannerType = ref('');
    const streakCount = ref(0);
    const streakLastDate = ref('');

    let timerInterval = null;
    let bannerTimeout = null;
    let audioCtx = null;
    let ambientNodes = null;
    let ambientGainNode = null;

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

    const reversedLog = computed(() => [...sessionLog.value].reverse());

    const streakTitle = computed(() => {
      const c = streakCount.value;
      if (c === 0) return 'Mulai streak-mu!';
      if (c < 3)   return 'Baru mulai! 💪';
      if (c < 7)   return 'Keren! Lagi panas 🔥';
      return 'Luar biasa! 🏆';
    });

    const streakSub = computed(() => {
      const c = streakCount.value;
      if (c === 0) return 'Selesaikan minimal 1 sesi hari ini';
      if (c < 3)   return 'Pertahankan terus ya!';
      if (c < 7)   return 'Sudah ' + c + ' hari berturut-turut!';
      return c + ' hari — kamu konsisten banget!';
    });

    const streakDots = computed(() => {
      const dots = [];
      for (let i = 6; i >= 0; i--) {
        const d = new Date(Date.now() - i * 86400000).toISOString().slice(0, 10);
        let cls = '', icon = '';
        if (d === TODAY) { cls = 'today'; icon = '📍'; }
        else if (streakLastDate.value && i > 0 && streakCount.value >= (7 - i)) { cls = 'done'; icon = '✓'; }
        dots.push({ date: d, cls, icon });
      }
      return dots;
    });

    // ─── STORAGE ─────────────────────────────────────────────────────────
    function loadStorage() {
      try {
        const d = JSON.parse(localStorage.getItem('pomodoro_data') || '{}');
        todoItems.value  = d.todos || [];
        goalTarget.value = d.goalTarget || 4;
        const streakData = d.streak || { count: 0, lastDate: '' };
        streakCount.value    = streakData.count;
        streakLastDate.value = streakData.lastDate;
        const dayData = d.dayData || {};
        const today   = dayData[TODAY] || { sessions: 0, focusSec: 0, log: [] };
        completedSessions.value = today.sessions;
        totalFocusSec.value     = today.focusSec;
        sessionLog.value        = today.log;
      } catch(e) {}
    }

    function saveStorage() {
      try {
        const d = JSON.parse(localStorage.getItem('pomodoro_data') || '{}');
        d.todos      = todoItems.value;
        d.goalTarget = goalTarget.value;
        d.streak     = { count: streakCount.value, lastDate: streakLastDate.value };
        const dayData = d.dayData || {};
        dayData[TODAY] = { sessions: completedSessions.value, focusSec: totalFocusSec.value, log: sessionLog.value };
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

    function resetTimer() {
      clearInterval(timerInterval);
      running.value  = false;
      isBreak.value  = false;
      totalSec.value = focusMin.value * 60;
      bannerVisible.value = false;
      document.title = 'POMODORO';
    }

    function tick() {
      if (totalSec.value > 0) {
        totalSec.value--;
        if (!isBreak.value) totalFocusSec.value++;
        const m = Math.floor(totalSec.value / 60);
        const s = totalSec.value % 60;
        document.title = running.value ? pad(m) + ':' + pad(s) + ' - ' + (isBreak.value ? 'Break' : 'Focus') : 'POMODORO';
      } else {
        clearInterval(timerInterval);
        running.value = false;
        playBeep();
        if (!isBreak.value) {
          completedSessions.value++;
          const now = new Date();
          const timeStr = pad(now.getHours()) + ':' + pad(now.getMinutes());
          sessionLog.value.push({ time: timeStr, type: 'Fokus', dur: focusMin.value + 'm' });
          saveStorage();
          checkStreak();
          isBreak.value  = true;
          totalSec.value = breakMin.value * 60;
          document.title = 'POMODORO';
          showBanner('☕ Sesi fokus selesai! Waktunya istirahat — tekan START untuk mulai break.', 'break-banner');
          sendNotif('Pomodoro — Fokus Selesai! ☕', 'Waktunya istirahat. Tekan START untuk mulai break.');
        } else {
          const now = new Date();
          const timeStr = pad(now.getHours()) + ':' + pad(now.getMinutes());
          sessionLog.value.push({ time: timeStr, type: 'Break', dur: breakMin.value + 'm' });
          saveStorage();
          isBreak.value  = false;
          totalSec.value = focusMin.value * 60;
          document.title = 'POMODORO';
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

    // ─── STREAK ──────────────────────────────────────────────────────────
    function checkStreak() {
      const yesterday = new Date(Date.now() - 86400000).toISOString().slice(0, 10);
      if (streakLastDate.value === TODAY) return;
      if (streakLastDate.value === yesterday || streakLastDate.value === '') streakCount.value++;
      else streakCount.value = 1;
      streakLastDate.value = TODAY;
      saveStorage();
    }

    // ─── TODO ────────────────────────────────────────────────────────────
    function addTodo() {
      const text = todoInputVal.value.trim();
      if (!text) return;
      todoItems.value.push({ id: Date.now(), text, done: false });
      todoInputVal.value = '';
      saveStorage();
    }

    function toggleTodo(id) {
      const t = todoItems.value.find(x => x.id === id);
      if (t) { t.done = !t.done; saveStorage(); }
    }

    function deleteTodo(id) {
      todoItems.value = todoItems.value.filter(x => x.id !== id);
      saveStorage();
    }

    // ─── LOG ─────────────────────────────────────────────────────────────
    function exportCSV() {
      if (!sessionLog.value.length) { alert('Belum ada sesi untuk di-export.'); return; }
      let csv = 'Waktu,Tipe,Durasi\n';
      sessionLog.value.forEach(l => { csv += `${l.time},${l.type},${l.dur}\n`; });
      const blob = new Blob([csv], { type: 'text/csv' });
      const a = document.createElement('a');
      a.href = URL.createObjectURL(blob);
      a.download = 'pomodoro_' + TODAY + '.csv';
      a.click();
    }

    // ─── AUDIO ───────────────────────────────────────────────────────────
    function getAudioCtx() {
      if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
      return audioCtx;
    }

    function stopAmbient() {
      if (ambientNodes) { ambientNodes.forEach(n => { try { n.stop(); } catch(e) {} }); ambientNodes = null; }
      if (ambientGainNode) { try { ambientGainNode.disconnect(); } catch(e) {} ambientGainNode = null; }
    }

    function makeNoiseBuffer(ctx, type) {
      const sr = ctx.sampleRate, len = sr * 3;
      const buf = ctx.createBuffer(1, len, sr);
      const d   = buf.getChannelData(0);
      if (type === 'white') { for (let i = 0; i < len; i++) d[i] = Math.random() * 2 - 1; }
      else if (type === 'pink') {
        let b0=0, b1=0, b2=0, b3=0, b4=0, b5=0;
        for (let i = 0; i < len; i++) {
          const w = Math.random() * 2 - 1;
          b0=0.99886*b0+w*0.0555179; b1=0.99332*b1+w*0.0750759;
          b2=0.96900*b2+w*0.1538520; b3=0.86650*b3+w*0.3104856;
          b4=0.55000*b4+w*0.5329522; b5=-0.7616*b5-w*0.0168980;
          d[i] = (b0+b1+b2+b3+b4+b5+w*0.5362)/7;
        }
      } else if (type === 'brown') {
        let last = 0;
        for (let i = 0; i < len; i++) {
          const w = Math.random() * 2 - 1;
          d[i] = (last + 0.02 * w) / 1.02; last = d[i]; d[i] *= 3.5;
        }
      }
      return buf;
    }

    function setAmbient(type) {
      stopAmbient();
      activeAmbient.value = type;
      if (type === 'off') return;
      const ctx = getAudioCtx();
      ambientGainNode = ctx.createGain();
      ambientGainNode.gain.value = ambVolume.value;
      ambientGainNode.connect(ctx.destination);
      ambientNodes = [];
      const vol = ambientGainNode.gain.value;

      if (type === 'rain') {
        const buf = makeNoiseBuffer(ctx, 'pink');
        const src = ctx.createBufferSource(); src.buffer = buf; src.loop = true;
        const filt = ctx.createBiquadFilter(); filt.type = 'bandpass'; filt.frequency.value = 600; filt.Q.value = 0.5;
        src.connect(filt); filt.connect(ambientGainNode); src.start();
        ambientNodes.push(src);
        let drips = setInterval(() => {
          if (!ambientNodes) return clearInterval(drips);
          try {
            const o = ctx.createOscillator(); const g = ctx.createGain();
            o.frequency.value = 800 + Math.random() * 400; o.type = 'sine';
            g.gain.setValueAtTime(0, ctx.currentTime); g.gain.linearRampToValueAtTime(vol * 0.08, ctx.currentTime + 0.01);
            g.gain.exponentialRampToValueAtTime(0.0001, ctx.currentTime + 0.3);
            o.connect(g); g.connect(ambientGainNode); o.start(); o.stop(ctx.currentTime + 0.35);
          } catch(e) {}
        }, 200 + Math.random() * 300);
        ambientNodes.push({ stop: () => clearInterval(drips) });
      } else if (type === 'white') {
        const buf = makeNoiseBuffer(ctx, 'white');
        const src = ctx.createBufferSource(); src.buffer = buf; src.loop = true;
        const filt = ctx.createBiquadFilter(); filt.type = 'lowpass'; filt.frequency.value = 3000;
        src.connect(filt); filt.connect(ambientGainNode); src.start();
        ambientNodes.push(src);
      } else if (type === 'cafe') {
        const buf = makeNoiseBuffer(ctx, 'brown');
        const src = ctx.createBufferSource(); src.buffer = buf; src.loop = true;
        const filt = ctx.createBiquadFilter(); filt.type = 'bandpass'; filt.frequency.value = 400; filt.Q.value = 0.3;
        src.connect(filt); filt.connect(ambientGainNode); src.start();
        ambientNodes.push(src);
        let bumps = setInterval(() => {
          if (!ambientNodes) return clearInterval(bumps);
          try {
            const o = ctx.createOscillator(); const g = ctx.createGain();
            o.frequency.value = 200 + Math.random() * 300; o.type = 'triangle';
            g.gain.setValueAtTime(0, ctx.currentTime); g.gain.linearRampToValueAtTime(vol * 0.04, ctx.currentTime + 0.08);
            g.gain.exponentialRampToValueAtTime(0.0001, ctx.currentTime + 0.5);
            o.connect(g); g.connect(ambientGainNode); o.start(); o.stop(ctx.currentTime + 0.6);
          } catch(e) {}
        }, 400 + Math.random() * 600);
        ambientNodes.push({ stop: () => clearInterval(bumps) });
      } else if (type === 'forest') {
        const buf = makeNoiseBuffer(ctx, 'pink');
        const src = ctx.createBufferSource(); src.buffer = buf; src.loop = true;
        const filt = ctx.createBiquadFilter(); filt.type = 'highpass'; filt.frequency.value = 800;
        src.connect(filt); filt.connect(ambientGainNode); src.start();
        ambientNodes.push(src);
        let birds = setInterval(() => {
          if (!ambientNodes) return clearInterval(birds);
          try {
            const o = ctx.createOscillator(); const g = ctx.createGain();
            o.frequency.setValueAtTime(1800 + Math.random() * 800, ctx.currentTime);
            o.frequency.linearRampToValueAtTime(2400 + Math.random() * 400, ctx.currentTime + 0.15);
            o.type = 'sine'; g.gain.setValueAtTime(0, ctx.currentTime);
            g.gain.linearRampToValueAtTime(vol * 0.06, ctx.currentTime + 0.05);
            g.gain.exponentialRampToValueAtTime(0.0001, ctx.currentTime + 0.35);
            o.connect(g); g.connect(ambientGainNode); o.start(); o.stop(ctx.currentTime + 0.4);
          } catch(e) {}
        }, 600 + Math.random() * 800);
        ambientNodes.push({ stop: () => clearInterval(birds) });
      } else if (type === 'fire') {
        const buf = makeNoiseBuffer(ctx, 'brown');
        const src = ctx.createBufferSource(); src.buffer = buf; src.loop = true;
        const filt = ctx.createBiquadFilter(); filt.type = 'bandpass'; filt.frequency.value = 200; filt.Q.value = 0.4;
        src.connect(filt); filt.connect(ambientGainNode); src.start();
        ambientNodes.push(src);
        let crackle = setInterval(() => {
          if (!ambientNodes) return clearInterval(crackle);
          try {
            const g = ctx.createGain(); const buf2 = makeNoiseBuffer(ctx, 'white');
            const s2 = ctx.createBufferSource(); s2.buffer = buf2;
            const f2 = ctx.createBiquadFilter(); f2.type = 'bandpass'; f2.frequency.value = 3000 + Math.random() * 2000;
            g.gain.setValueAtTime(0, ctx.currentTime); g.gain.linearRampToValueAtTime(vol * 0.12, ctx.currentTime + 0.005);
            g.gain.exponentialRampToValueAtTime(0.0001, ctx.currentTime + 0.08);
            s2.connect(f2); f2.connect(g); g.connect(ambientGainNode); s2.start(); s2.stop(ctx.currentTime + 0.1);
          } catch(e) {}
        }, 150 + Math.random() * 250);
        ambientNodes.push({ stop: () => clearInterval(crackle) });
      }
    }

    function setAmbientVolume() {
      if (ambientGainNode) ambientGainNode.gain.value = ambVolume.value;
    }

    function toggleMute() { muted.value = !muted.value; }

    function playBeep() {
      if (muted.value) return;
      try {
        const ctx = getAudioCtx(); const now = ctx.currentTime;
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
      if ('Notification' in window && Notification.permission === 'granted') new Notification(title, { body });
    }

    function requestNotif() {
      if ('Notification' in window) {
        Notification.requestPermission().then(p => { if (p === 'granted') showNotifBar.value = false; });
      }
    }

    // ─── LIFECYCLE ───────────────────────────────────────────────────────
    onMounted(() => {
      loadStorage();
      if ('Notification' in window && Notification.permission === 'default') showNotifBar.value = true;
    });

    onUnmounted(() => {
      clearInterval(timerInterval);
      stopAmbient();
    });

    return {
      focusMin, breakMin, totalSec, isBreak, running, muted,
      completedSessions, totalFocusSec, todoItems, todoInputVal,
      sessionLog, goalTarget, activeAmbient, ambVolume,
      showNotifBar, bannerVisible, bannerMsg, bannerType,
      streakCount, streakLastDate,
      ambientOptions,
      timeDisplay, progressPct, goalPct, totalFocusDisplay,
      reversedLog, streakTitle, streakSub, streakDots,
      toggleTimer, resetTimer, onFocusMinChange, onBreakMinChange,
      addTodo, toggleTodo, deleteTodo,
      exportCSV, setAmbient, setAmbientVolume, toggleMute,
      requestNotif, saveStorage,
    };
  }
}).mount('#app');
</script>
</body>
</html>
