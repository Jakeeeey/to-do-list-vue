<!-- App.vue -->
<script setup>
import { ref, computed, watch } from "vue";

const newTask = ref("");
const filter = ref("all");
const tasks = ref(JSON.parse(localStorage.getItem("tasks") || "[]"));

const addTask = () => {
  const text = newTask.value.trim();
  if (!text) return;
  tasks.value.push({
    id: Date.now(),
    text,
    completed: false,
    createdAt: new Date().toISOString(),
  });
  newTask.value = "";
};

const deleteTask = (id) => {
  tasks.value = tasks.value.filter((t) => t.id !== id);
};

// Inline editing
const editingId = ref(null);
const editText = ref("");

const startEdit = (task) => {
  editingId.value = task.id;
  editText.value = task.text;
};

const saveEdit = (task) => {
  const text = editText.value.trim();
  if (text) task.text = text;
  editingId.value = null;
};

const cancelEdit = () => {
  editingId.value = null;
};

const clearCompleted = () => {
  tasks.value = tasks.value.filter((t) => !t.completed);
};

// Filters
const filteredTasks = computed(() => {
  if (filter.value === "active") return tasks.value.filter((t) => !t.completed);
  if (filter.value === "completed") return tasks.value.filter((t) => t.completed);
  return tasks.value;
});

const remaining = computed(() => tasks.value.filter((t) => !t.completed).length);

// Persist
watch(
  tasks,
  (val) => localStorage.setItem("tasks", JSON.stringify(val)),
  { deep: true }
);
</script>

<template>
  <div class="min-h-screen bg-gray-100 flex items-center justify-center">
    <div class="flex flex-col bg-white p-6 md:rounded-2xl shadow-lg w-full max-md:h-screen overflow-auto max-w-lg">
      <h1 class="text-2xl font-bold mb-4 text-center">To-Do List</h1>

      <!-- Add -->
      <div class="flex gap-2 mb-4">
        <input
          v-model="newTask"
          @keyup.enter="addTask"
          type="text"
          placeholder="Add a new task..."
          class="flex-1 border rounded-lg p-2"
        />
        <button
          @click="addTask"
          class="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600"
        >
          Add
        </button>
      </div>

      <!-- Filters -->
      <div class="flex items-center justify-between mb-3 text-sm">
        <div class="flex gap-3">
          <button @click="filter = 'all'" :class="filter === 'all' ? 'font-semibold' : ''">All</button>
          <button @click="filter = 'active'" :class="filter === 'active' ? 'font-semibold' : ''">Active</button>
          <button @click="filter = 'completed'" :class="filter === 'completed' ? 'font-semibold' : ''">Completed</button>
        </div>
        <div class="text-gray-600">{{ remaining }} left</div>
      </div>

      <!-- List -->
      <ul class="flex flex-col space-y-2">
        <li
          v-for="task in filteredTasks"
          :key="task.id"
          class="flex items-center justify-between bg-gray-50 p-2 rounded-lg"
        >
          <div class="flex items-center gap-3">
            <!-- FIX: remove @change; v-model alone updates .completed -->
            <input type="checkbox" v-model="task.completed" />
            <div v-if="editingId !== task.id" class="flex items-center gap-2">
              <span :class="task.completed ? 'line-through text-gray-500' : ''">
                {{ task.text }}
              </span>
              <button class="text-xs text-blue-600" @click="startEdit(task)">Edit</button>
            </div>
            <div v-else class="flex items-center gap-2">
              <input
                v-model="editText"
                @keyup.enter="saveEdit(task)"
                @keyup.esc="cancelEdit"
                class="border max-md:w-40 rounded p-1"
                autofocus
              />
              <button class="text-xs text-green-600" @click="saveEdit(task)">Save</button>
              <button class="text-xs text-gray-500" @click="cancelEdit">Cancel</button>
            </div>
          </div>
          <button class="text-red-500 hover:text-red-700" @click="deleteTask(task.id)">✕</button>
        </li>
      </ul>

      <!-- Footer actions -->
      <div class="flex justify-end mt-4">
        <button
          class="text-sm text-gray-600 hover:text-gray-800"
          @click="clearCompleted"
        >
          Clear Completed
        </button>
      </div>
    </div>
  </div>
</template>

<style>
body { font-family: Inter, system-ui, -apple-system, Segoe UI, Roboto, Arial, Noto Sans, "Apple Color Emoji", "Segoe UI Emoji"; }
</style>
