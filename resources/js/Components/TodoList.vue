<template>
  <div class="max-w-4xl mx-auto p-6">
    <div class="shadow-2xl rounded-lg overflow-hidden">
      <!-- Clock and Date Display -->
      <div class="bg-gradient-to-r from-gray-800 to-gray-900 text-white p-8 border-b border-gray-700">
        <h2 class="text-4xl font-bold mb-2 text-transparent bg-clip-text bg-gradient-to-r from-gray-100 to-gray-300">{{ currentTime }}</h2>
        <p class="text-lg text-gray-300">{{ currentDate }}</p>
      </div>

      <!-- Todo List Content -->
      <div class="p-8">
        <h2 class="text-3xl font-semibold text-white mb-8">To-do List</h2>

        <!-- Add Task Form -->
        <form @submit.prevent="addTask" class="mb-8">
          <div class="flex gap-4">
            <input
              v-model="newTask"
              type="text"
              placeholder="Add a new task"
              required
              class="flex-grow px-4 py-3 bg-gray-800 text-white border border-gray-600 rounded-md focus:outline-none focus:ring-2 focus:ring-gray-500 placeholder-gray-400"
            />
            <button
              type="submit"
              class="px-6 py-3 bg-white text-gray-900 rounded-md hover:bg-gray-200 focus:outline-none focus:ring-2 focus:ring-gray-300 focus:ring-offset-2 focus:ring-offset-gray-800 transition duration-300 font-semibold"
            >
              Add Task
            </button>
          </div>
          <!-- Display Error Message -->
          <p v-if="errorMessage" class="mt-2 text-red-500 text-sm">{{ errorMessage }}</p>
        </form>

        <!-- Task List -->
        <ul class="space-y-4">
          <li
            v-for="task in tasks"
            :key="task.id"
            class="bg-gray-800 rounded-lg p-4 flex items-center justify-between transition duration-300 hover:bg-gray-700"
          >
            <div class="flex items-center space-x-4">
              <input
                type="checkbox"
                :id="'task-' + task.id"
                v-model="task.completed"
                @change="updateTask(task)"
                class="h-5 w-5 text-white border-gray-600 rounded focus:ring-offset-gray-800 focus:ring-gray-500"
              />
              <label
                :for="'task-' + task.id"
                :class="{ 'line-through text-gray-500': task.completed }"
                class="text-lg text-white"
              >
                {{ task.task }}
              </label>
            </div>
            <div class="flex items-center space-x-4">
              <div class="text-sm text-gray-400">
                <div>Created: {{ formatDate(task.created_at) }}</div>
                <div>Updated: {{ formatDate(task.updated_at) }}</div>
              </div>
              <button
                @click="openEditModal(task)"
                class="px-3 py-1 bg-gray-600 text-white rounded-md hover:bg-gray-500 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-offset-2 focus:ring-offset-gray-800 transition duration-300"
              >
                Edit
              </button>
              <button
                @click="confirmDelete(task)"
                class="px-3 py-1 bg-red-600 text-white rounded-md hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-offset-2 focus:ring-offset-gray-800 transition duration-300"
              >
                Delete
              </button>
            </div>
          </li>
        </ul>
      </div>
    </div>

    <!-- Edit Task Modal -->
    <div v-if="isEditModalVisible" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center">
      <div class="bg-gray-800 rounded-lg p-6 w-96">
        <h3 class="text-xl font-semibold mb-4 text-white">Edit Task</h3>
        <input
          v-model="editedTask.task"
          type="text"
          class="w-full px-3 py-2 bg-gray-700 text-white border border-gray-600 rounded-md focus:outline-none focus:ring-2 focus:ring-gray-500 mb-4"
        />
        <div class="flex justify-end space-x-2">
          <button
            @click="confirmEditTask"
            class="px-4 py-2 bg-white text-gray-900 rounded-md hover:bg-gray-200 focus:outline-none focus:ring-2 focus:ring-gray-300 focus:ring-offset-2 focus:ring-offset-gray-800 transition duration-300"
          >
            Save
          </button>
          <button
            @click="closeEditModal"
            class="px-4 py-2 bg-gray-700 text-white rounded-md hover:bg-gray-600 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-offset-2 focus:ring-offset-gray-800 transition duration-300"
          >
            Cancel
          </button>
        </div>
      </div>
    </div>

    <!-- Delete Confirmation Modal -->
    <div v-if="isDeleteModalVisible" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center">
      <div class="bg-gray-800 rounded-lg p-6 w-96">
        <h3 class="text-xl font-semibold mb-4 text-white">Confirm Deletion</h3>
        <p class="mb-4 text-gray-300">Are you sure you want to delete this task?</p>
        <div class="flex justify-end space-x-2">
          <button
            @click="deleteTask"
            class="px-4 py-2 bg-red-600 text-white rounded-md hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-offset-2 focus:ring-offset-gray-800 transition duration-300"
          >
            Delete
          </button>
          <button
            @click="closeDeleteModal"
            class="px-4 py-2 bg-gray-700 text-white rounded-md hover:bg-gray-600 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-offset-2 focus:ring-offset-gray-800 transition duration-300"
          >
            Cancel
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import axios from 'axios'

const tasks = ref([])
const newTask = ref('')
const editedTask = ref(null)
const isEditModalVisible = ref(false)
const isDeleteModalVisible = ref(false)
const taskToDelete = ref(null)
const currentTime = ref('')
const currentDate = ref('')
const errorMessage = ref('') // To display validation errors

let timer

onMounted(() => {
  fetchTasks()
  updateDateTime()
  timer = setInterval(updateDateTime, 1000)
})

onUnmounted(() => {
  clearInterval(timer)
})

const updateDateTime = () => {
  const now = new Date()
  currentTime.value = now.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit', second: '2-digit' })
  currentDate.value = now.toLocaleDateString('en-US', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' })
}

const fetchTasks = async () => {
  try {
    const response = await axios.get('/api/tasks')
    tasks.value = response.data
  } catch (error) {
    console.error('Error fetching tasks:', error)
  }
}

// Validate input to prevent HTML tags
const addTask = async () => {
  errorMessage.value = '' // Clear previous errors

  // Validate input: Disallow specific patterns like HTML tags
  const invalidPattern = /<|>/g
  if (!newTask.value.trim() || invalidPattern.test(newTask.value)) {
    errorMessage.value = 'Invalid input: HTML tags or special characters are not allowed.'
    return
  }

  try {
    const response = await axios.post('/api/tasks', { task: newTask.value.trim() })
    tasks.value.push(response.data)
    newTask.value = ''
  } catch (error) {
    console.error('Error adding task:', error)
  }
}

const updateTask = async (task) => {
  try {
    await axios.patch(`/api/tasks/${task.id}`, { completed: task.completed })
  } catch (error) {
    console.error('Error updating task:', error)
  }
}

const openEditModal = (task) => {
  editedTask.value = { ...task }
  isEditModalVisible.value = true
}

const closeEditModal = () => {
  isEditModalVisible.value = false
  editedTask.value = null
}

const confirmEditTask = async () => {
  if (editedTask.value && editedTask.value.task.trim()) {
    try {
      const response = await axios.patch(`/api/tasks/${editedTask.value.id}`, { task: editedTask.value.task })
      const index = tasks.value.findIndex(t => t.id === editedTask.value.id)
      if (index !== -1) {
        tasks.value[index] = response.data
      }
      closeEditModal()
    } catch (error) {
      console.error('Error updating task:', error)
    }
  }
}

const confirmDelete = (task) => {
  taskToDelete.value = task
  isDeleteModalVisible.value = true
}

const closeDeleteModal = () => {
  isDeleteModalVisible.value = false
  taskToDelete.value = null
}

const deleteTask = async () => {
  try {
    await axios.delete(`/api/tasks/${taskToDelete.value.id}`)
    tasks.value = tasks.value.filter(t => t.id !== taskToDelete.value.id)
    closeDeleteModal()
  } catch (error) {
    console.error('Error deleting task:', error)
  }
}

const formatDate = (dateString) => {
  const date = new Date(dateString)
  return date.toLocaleString('en-US', { hour: '2-digit', minute: '2-digit', second: '2-digit', month: 'short', day: 'numeric', year: 'numeric' })
}
</script>

<style scoped>
/* Add your custom styles here */
</style>
