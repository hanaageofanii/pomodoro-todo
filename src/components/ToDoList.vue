<template>
  <div>
    <h2 class="title">DAFTAR TUGAS</h2>
    <form @submit.prevent="addTask" class="form">
      <input v-model="newTask" placeholder="Input tugas" />
      <button type="submit">TAMBAH</button>
    </form>

    <ul class="task-list">
      <li v-for="(task, index) in tasks" :key="index" :class="{ done: task.done }">
        <span>{{ task.text }}</span>
        <button @click="toggleTask(index)">{{ task.done ? 'ULANGI' : 'SELESAI' }}</button>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const tasks = ref([])
const newTask = ref('')

function addTask() {
  if (!newTask.value.trim()) return
  tasks.value.push({ text: newTask.value.trim(), done: false })
  newTask.value = ''
}

function toggleTask(index) {
  tasks.value[index].done = !tasks.value[index].done
}
</script>

<style scoped>
.title {
  font-size: 0.8rem;
  letter-spacing: 3px;
  margin-bottom: 1rem;
  text-align: center;
  color: #000;
}

.form {
  display: flex;
  margin-bottom: 1rem;
}

input {
  flex: 1;
  font-family: 'Press Start 2P', cursive, monospace;
  border: 4px solid #eee;
  background: #222;
  color: #eee;
  padding: 0.5rem;
  box-shadow: 3px 3px 0 #444;
  border-radius: 4px 0 0 4px;
  font-size: 0.7rem;
  outline: none;
}

input::placeholder {
  color: #666;
}

button {
  background: #f0a04b;
  color: #222;
  font-weight: bold;
  border-radius: 0 4px 4px 0;
  border: none;
  padding: 0 1rem;
  box-shadow: 3px 3px 0 #fada7a;
  font-size: 0.7rem;
  cursor: pointer;
  transition: filter 0.2s;
}

button:hover {
  filter: brightness(1.1);
}

.task-list {
  list-style: none;
  padding: 0;
}

.task-list li {
  display: flex;
  justify-content: space-between;
  border-bottom: 2px solid #333;
  padding: 0.5rem 0;
  font-family: 'Press Start 2P', cursive, monospace;
  font-size: 0.7rem;
  user-select: none;
}

.task-list li.done span {
  text-decoration: line-through;
  color: #777;
}

.task-list li button {
  background: #4caf50;
  border: none;
  color: #222;
  padding: 0.2rem 0.7rem;
  font-weight: bold;
  border-radius: 3px;
  box-shadow: 2px 2px 0 #367c39;
  cursor: pointer;
  transition: filter 0.2s;
}

.task-list li button:hover {
  filter: brightness(1.1);
}
</style>
