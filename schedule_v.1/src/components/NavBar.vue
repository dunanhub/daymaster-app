<template>
  <nav class="bg-white dark:bg-gray-800 shadow transition-colors duration-300">
    <div class="max-w-7xl mx-auto px-4 py-3 flex items-center">
      <!-- Логотип -->
      <div class="text-2xl font-bold text-blue-600 dark:text-blue-400">
        <a href="#" @click.prevent="go('dashboard')">DayMaster</a>
      </div>
      <!-- Spacer -->
      <div class="flex-1"></div>
      <!-- Навигация для ПК -->
      <ul class="hidden mr-8 md:flex md:ml-10 md:space-x-6">
        <li><a href="#" @click.prevent="go('schedule')" class="text-gray-700 dark:text-gray-200 hover:text-blue-600 dark:hover:text-blue-400">Расписание</a></li>
        <li><a href="#" @click.prevent="go('food')" class="text-gray-700 dark:text-gray-200 hover:text-blue-600 dark:hover:text-blue-400">Еда</a></li>
        <li><a href="#" @click.prevent="go('todo')" class="text-gray-700 dark:text-gray-200 hover:text-blue-600 dark:hover:text-blue-400">To-Do</a></li>
        <li><a href="#" @click.prevent="go('workout')" class="text-gray-700 dark:text-gray-200 hover:text-blue-600 dark:hover:text-blue-400">Тренировки</a></li>
      </ul>
      <!-- Dark/Light Mode -->
      <button @click="toggleTheme" class="p-1 rounded focus:outline-none focus:ring">
        <svg v-if="theme === 'light'" xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-gray-700" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 3v2m0 14v2m9-9h-2M5 12H3m15.364-6.364l-1.414 1.414M7.05 16.95l-1.414 1.414m12.728 0l-1.414-1.414M7.05 7.05L5.636 5.636M12 8a4 4 0 100 8 4 4 0 000-8z" />
        </svg>
        <svg v-else xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-yellow-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 12.79A9 9 0 1111.21 3 7 7 0 0021 12.79z" />
        </svg>
      </button>
      <!-- Кнопка для мобильного меню -->
      <button @click="toggleMenu" class="md:hidden ml-2 p-1 rounded focus:outline-none focus:ring">
        <svg v-if="!isOpen" xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-gray-700 dark:text-gray-200" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
        </svg>
        <svg v-else xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-gray-700 dark:text-gray-200" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
        </svg>
      </button>
    </div>
    <!-- Навигация для мобильного -->
    <transition name="fade" appear>
      <ul v-show="isOpen" class="md:hidden px-4 pb-3 space-y-1 bg-white dark:bg-gray-800 shadow-md">
        <li><a href="#" @click.prevent="go('schedule')" class="block px-4 py-2 text-gray-700 dark:text-gray-200 hover:text-blue-600 dark:hover:text-blue-400">Расписание</a></li>
        <li><a href="#" @click.prevent="go('food')" class="block px-4 py-2 text-gray-700 dark:text-gray-200 hover:text-blue-600 dark:hover:text-blue-400">Еда</a></li>
        <li><a href="#" @click.prevent="go('todo')" class="block px-4 py-2 text-gray-700 dark:text-gray-200 hover:text-blue-600 dark:hover:text-blue-400">To-Do</a></li>
        <li><a href="#" @click.prevent="go('workout')" class="block px-4 py-2 text-gray-700 dark:text-gray-200 hover:text-blue-600 dark:hover:text-blue-400">Тренировки</a></li>
      </ul>
    </transition>
  </nav>
</template>

<script setup>
import { ref, watchEffect, onMounted } from 'vue'
// Определяем событие для переключения страниц
const emit = defineEmits(['change-page'])

const isOpen = ref(false)
function toggleMenu() { isOpen.value = !isOpen.value }

const theme = ref('light')
function applyTheme(val) {
  document.documentElement.classList.toggle('dark', val === 'dark')
  localStorage.setItem('theme', val)
}
function toggleTheme() { theme.value = theme.value === 'light' ? 'dark' : 'light' }

function go(page) {
  console.log('Emitting change-page:', page); // <-- Добавить эту строку
  emit('change-page', page)
  isOpen.value = false
}

onMounted(() => {
  const saved = localStorage.getItem('theme')
  theme.value = saved === 'dark' ? 'dark' : 'light'
  applyTheme(theme.value)
})
watchEffect(() => applyTheme(theme.value))
</script>

<style scoped>
.fade-enter-active, .fade-leave-active { transition: opacity 0.3s ease }
.fade-enter-from, .fade-leave-to   { opacity: 0 }
.fade-enter-to, .fade-leave-from   { opacity: 1 }
</style>