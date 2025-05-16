<template>
  <div class="container">
    <h1 class="title">POMODORO TO-DO</h1>
    <PomodoroTimer />
    <ToDoList />
  </div>
</template>

<script setup>
import PomodoroTimer from './components/PomodoroTimer.vue'
import ToDoList from './components/ToDoList.vue'
</script>

<style scoped>
.title {
  font-size: 1.2rem;
  color: #000;
  text-align: center;
  margin-bottom: 1.5rem;
  letter-spacing: 2px;
}
</style>
