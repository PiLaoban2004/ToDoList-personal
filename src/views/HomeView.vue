<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Todo {
  id: number
  text: string
  completed: boolean
  image: string | null
}

const newTodo = ref('')
const todos = ref<Todo[]>([])
let nextId = 1

const STORAGE_KEY = 'vue-todos'
const DATE_KEY = 'todo-date'

onMounted(() => {
  const today = new Date().toISOString().slice(0, 10)
  const lastDate = localStorage.getItem(DATE_KEY)

  if (lastDate === today) {
    const storedTodos = localStorage.getItem(STORAGE_KEY)
    if (storedTodos) {
      todos.value = JSON.parse(storedTodos)
      nextId = todos.value.length > 0 ? Math.max(...todos.value.map(t => t.id)) + 1 : 1
    }
  } else {
    localStorage.removeItem(STORAGE_KEY)
    localStorage.setItem(DATE_KEY, today)
    todos.value = []
  }
})

function addTodo() {
  if (newTodo.value.trim() === '') return
  todos.value.push({
    id: nextId++,
    text: newTodo.value,
    completed: false,
    image: null
  })
  newTodo.value = ''
  saveTodos()
}

function removeTodo(id: number) {
  todos.value = todos.value.filter(todo => todo.id !== id)
  saveTodos()
}

function toggleTodo(id: number) {
  const todo = todos.value.find(todo => todo.id === id)
  if (todo) {
    todo.completed = !todo.completed
    saveTodos()
  }
}

function saveTodos() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(todos.value))
}

function takePicture(id: number) {
  const todo = todos.value.find(todo => todo.id === id)
  if (!todo) return

  const video = document.createElement('video')
  const canvas = document.createElement('canvas')
  const streamPromise = navigator.mediaDevices.getUserMedia({ video: true })

  streamPromise.then(stream => {
    video.srcObject = stream
    video.play()

    video.onloadedmetadata = () => {
      canvas.width = video.videoWidth
      canvas.height = video.videoHeight
      const context = canvas.getContext('2d')
      if (context) {
        context.drawImage(video, 0, 0, canvas.width, canvas.height)
        todo.image = canvas.toDataURL('image/png')
        saveTodos()
      }
      stream.getTracks().forEach(track => track.stop())
    }
  }).catch(err => {
    console.error("Error accessing camera: ", err);
    alert("无法访问摄像头。请检查权限设置。");
  });
}
</script>

<template>
  <main class="todo-app">
    <h1>待办清单</h1>
    <el-form @submit.prevent="addTodo" class="add-todo-form">
      <el-input v-model="newTodo" placeholder="添加新的待办事项" />
      <el-button type="primary" @click="addTodo">添加</el-button>
    </el-form>
    <el-card v-for="todo in todos" :key="todo.id" class="todo-item">
      <div class="todo-main-content">
        <div class="todo-content">
          <el-checkbox v-model="todo.completed" @change="() => toggleTodo(todo.id)" />
          <span :class="{ completed: todo.completed }">{{ todo.text }}</span>
        </div>
        <div class="todo-actions">
          <el-button type="primary" size="small" @click="() => takePicture(todo.id)">拍照</el-button>
          <el-button type="danger" size="small" @click="() => removeTodo(todo.id)">删除</el-button>
        </div>
      </div>
      <el-image v-if="todo.image" :src="todo.image" fit="contain" class="todo-image"></el-image>
    </el-card>
  </main>
</template>

<style scoped>
.todo-app {
  max-width: 600px;
  margin: 50px auto;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
  background-color: #f9f9f9;
}

h1 {
  text-align: center;
  color: #333;
}

.add-todo-form {
  display: flex;
  margin-bottom: 20px;
}

.add-todo-form .el-input {
  flex-grow: 1;
  margin-right: 10px;
}

.todo-item {
  margin-bottom: 10px;
  transition: box-shadow 0.3s;
}

.todo-item:hover {
  box-shadow: 0 4px 16px 0 rgba(0, 0, 0, 0.15);
}

.todo-main-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
}

.todo-content {
  display: flex;
  align-items: center;
  flex-grow: 1; /* Allow content to take up available space */
  margin-right: 10px; /* Add some space between content and actions */
}

.todo-content span {
  margin-left: 10px;
  font-size: 16px;
  word-break: break-word; /* Break long words */
}

.completed {
  text-decoration: line-through;
  color: #999;
}

.todo-actions {
  display: flex;
  gap: 10px;
  flex-shrink: 0; /* Prevent buttons from shrinking */
}

.todo-image {
  margin-top: 10px;
  max-width: 100%;
  border-radius: 4px;
}

/* Responsive styles for mobile */
@media (max-width: 768px) {
  .todo-app {
    margin: 0;
    padding: 15px;
    border-radius: 0;
    box-shadow: none;
    min-height: 100vh;
    background-color: #fff;
  }

  .todo-main-content {
    flex-wrap: wrap; /* Allow items to wrap on smaller screens */
  }

  .todo-content {
    width: 100%;
    margin-bottom: 10px; /* Add space when actions wrap below */
    margin-right: 0;
  }

  .todo-actions {
    width: 100%;
    justify-content: flex-end; /* Align buttons to the right */
  }
}
</style>
