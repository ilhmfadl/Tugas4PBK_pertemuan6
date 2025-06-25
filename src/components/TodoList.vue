<template>
  <div class="todo-container">
    <h1>📋 Todo App dengan Axios & JSON Server</h1>

    <!-- Dashboard Statistik -->
    <div class="dashboard">
      <div class="stat-card total">
        <div class="progress-ring" :style="`--percent: 100; --color: #3a7bd5, #00d2ff;`">
          <div class="progress-inner">
            <span>{{ todos.length }}</span>
          </div>
        </div>
        <h2>Total</h2>
      </div>
      <div class="stat-card completed">
        <div class="progress-ring" :style="`--percent: ${completedPercent}; --color: #56ab2f, #a8e063;`">
          <div class="progress-inner">
            <span>{{ completedTodos }}</span>
          </div>
        </div>
        <h2>Selesai</h2>
      </div>
      <div class="stat-card pending">
        <div class="progress-ring" :style="`--percent: ${pendingPercent}; --color: #f7971e, #ffd200;`">
          <div class="progress-inner">
            <span>{{ pendingTodos }}</span>
          </div>
        </div>
        <h2>Belum</h2>
      </div>
    </div>



    <!-- Form Tambah -->
    <form @submit.prevent="addTodo" class="add-form">
      <input v-model="newTodo" placeholder="Tambah todo baru..." required class="todo-input" />
      <button type="submit" :disabled="loading">Tambah</button>
    </form>

    <!-- Status -->
    <p v-if="loading">Memuat data...</p>
    <p v-if="error" class="error">{{ error }}</p>

    <!-- Daftar Todo -->
    <ul v-if="todos.length" class="todo-list">
      <li v-for="todo in todos" :key="todo.id">
        <input
          type="checkbox"
          :checked="todo.completed"
          @change="toggleCompleted(todo)"
        />
        <template v-if="editId !== todo.id">
          <span :class="{ done: todo.completed }">{{ todo.title }}</span>
          <button @click="startEdit(todo)">✏️</button>
        </template>
        <template v-else>
          <input v-model="editTitle" class="edit-input" />
          <button @click="updateTodo(todo)">💾</button>
          <button @click="cancelEdit">❌</button>
        </template>
        <button @click="deleteTodo(todo.id)">🗑️</button>
      </li>
    </ul>

    <p v-else>Tidak ada todo.</p>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import axios from 'axios'

const todos = ref([])
const newTodo = ref('')
const loading = ref(false)
const error = ref('')
const editId = ref(null)
const editTitle = ref('')
const API_URL = 'http://localhost:3000/todos'

const loadTodos = async () => {
  loading.value = true
  error.value = ''
  try {
    const res = await axios.get(API_URL)
    todos.value = res.data
  } catch (err) {
    error.value = 'Gagal memuat data.'
    console.error(err)
  } finally {
    loading.value = false
  }
}

const addTodo = async () => {
  if (!newTodo.value.trim()) return
  loading.value = true
  error.value = ''
  try {
    const res = await axios.post(API_URL, {
      title: newTodo.value,
      completed: false
    })
    todos.value.push(res.data)
    newTodo.value = ''
  } catch (err) {
    error.value = 'Gagal menambah todo.'
  } finally {
    loading.value = false
  }
}

const toggleCompleted = async (todo) => {
  loading.value = true
  try {
    const res = await axios.put(`${API_URL}/${todo.id}`, {
      ...todo,
      completed: !todo.completed
    })
    const idx = todos.value.findIndex(t => t.id === todo.id)
    if (idx !== -1) todos.value[idx] = res.data
  } catch (err) {
    error.value = 'Gagal memperbarui todo.'
  } finally {
    loading.value = false
  }
}

const deleteTodo = async (id) => {
  loading.value = true
  try {
    await axios.delete(`${API_URL}/${id}`)
    todos.value = todos.value.filter(t => t.id !== id)
  } catch (err) {
    error.value = 'Gagal menghapus todo.'
  } finally {
    loading.value = false
  }
}

const startEdit = (todo) => {
  editId.value = todo.id
  editTitle.value = todo.title
}

const cancelEdit = () => {
  editId.value = null
  editTitle.value = ''
}

const updateTodo = async (todo) => {
  if (!editTitle.value.trim()) return
  loading.value = true
  try {
    const res = await axios.put(`${API_URL}/${todo.id}`, {
      ...todo,
      title: editTitle.value
    })
    const idx = todos.value.findIndex(t => t.id === todo.id)
    if (idx !== -1) todos.value[idx] = res.data
    editId.value = null
    editTitle.value = ''
  } catch (err) {
    error.value = 'Gagal menyimpan perubahan.'
  } finally {
    loading.value = false
  }
}


const completedTodos = computed(() => todos.value.filter(t => t.completed).length)
const pendingTodos = computed(() => todos.value.filter(t => !t.completed).length)

const completedPercent = computed(() => {
  const total = todos.value.length;
  return total === 0 ? 0 : Math.round((completedTodos.value / total) * 100);
});

const pendingPercent = computed(() =>
  todos.value.length ? (pendingTodos.value / todos.value.length) * 100 : 0
)

onMounted(loadTodos)
</script>

<style scoped>
.todo-container {
  max-width: 700px;
  margin: 40px auto;
  font-family: 'Segoe UI', sans-serif;
  background: #ffffffdd;
  padding: 30px;
  border-radius: 16px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
}

h1 {
  text-align: center;
  margin-bottom: 20px;
  color: #333;
}

/* Dashboard */
.dashboard {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
  margin-bottom: 25px;
}

.stat-card {
  padding: 10px;
  text-align: center;
  border-radius: 14px;
  color: #fff;
  position: relative;
}

.stat-card.total {
  background: linear-gradient(135deg, #3a7bd5, #00d2ff);
}
.stat-card.completed {
  background: linear-gradient(135deg, #56ab2f, #a8e063);
}
.stat-card.pending {
  background: linear-gradient(135deg, #f7971e, #ffd200);
}

.progress-circle {
  --size: 80px;
  --stroke: 12px;
  --bg: #ffffff44;
  --fill: #fff;
  --percent: 75;

  width: var(--size);
  height: var(--size);
  border-radius: 50%;
  background: conic-gradient(
    var(--fill) calc(var(--percent) * 1%),
    var(--bg) 0
  );
  display: flex;
  align-items: center;
  justify-content: center;
  margin: auto;
  margin-bottom: 8px;
  box-shadow: inset 0 0 6px rgba(255, 255, 255, 0.3);
}

.progress-text {
  font-weight: bold;
  font-size: 18px;
  color: #fff;
}

/* Input Form */
.add-form {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.todo-input {
  flex: 1;
  padding: 12px 18px;
  border-radius: 50px;
  border: none;
  background: #f1f3f6;
  font-size: 15px;
  box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.05);
}

.todo-input:focus {
  outline: none;
  background: #fff;
  box-shadow: 0 0 0 3px rgba(0, 198, 255, 0.25);
}

/* Tombol */
button {
  padding: 10px 16px;
  border: none;
  background: linear-gradient(135deg, #00c6ff, #0072ff);
  color: white;
  border-radius: 12px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s ease;
}

button:disabled {
  background: #ccc;
  cursor: not-allowed;
}

button:hover:not(:disabled) {
  transform: scale(1.05);
  background: linear-gradient(135deg, #0072ff, #00c6ff);
}

/* Todo List */
ul.todo-list {
  list-style: none;
  padding: 0;
}

.todo-list li {
  display: flex;
  align-items: center;
  background: #f5f5f5;
  padding: 12px;
  margin-bottom: 10px;
  border-radius: 10px;
}

.todo-list li span {
  flex: 1;
  margin-left: 10px;
}

.todo-list li span.done {
  text-decoration: line-through;
  color: #888;
}

.todo-list li input[type="checkbox"] {
  transform: scale(1.2);
  cursor: pointer;
}

.edit-input {
  flex: 1;
  padding: 10px;
  border-radius: 8px;
  border: 1px solid #ccc;
  font-weight: 500;
}

.error {
  color: red;
  text-align: center;
}

.progress-ring {
  --size: 90px;
  --percent: 0;
  --fill1: #00c6ff;
  --track: #e6e6e6;

  width: var(--size);
  height: var(--size);
  border-radius: 50%;
  background: conic-gradient(
    var(--fill1) calc(var(--percent) * 1%),
    var(--track) 0
  );
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.08);
  transition: all 0.3s ease;
}

.progress-inner {
  width: 65%;
  height: 65%;
  border-radius: 50%;
  background: radial-gradient(circle at top left, #f0f0f3, #dcdce4);
  box-shadow: inset -3px -3px 6px rgba(255, 255, 255, 0.6),
              inset 3px 3px 6px rgba(0, 0, 0, 0.1);
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  color: #333;
  font-size: 16px;
  transition: all 0.3s ease;
}


.progress-inner span {
  font-weight: bold;
  font-size: 16px;
  color: #333;
}



@keyframes ringFadeIn {
  0% {
    transform: scale(0.9);
    opacity: 0;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}


</style>
