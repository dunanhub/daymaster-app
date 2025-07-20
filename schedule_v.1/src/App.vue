<template>
  <div class="min-h-screen bg-gray-50 dark:bg-gray-900 transition-colors duration-300">
    <NavBar @change-page="updatePage" />
    <main class="max-w-7xl mx-auto p-4">
      <component :is="currentViewComponent" />
    </main>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue' // Убедитесь, что 'computed' импортирован
import NavBar from './components/NavBar.vue'
import Dashboard from './components/Dashboard.vue'
import SchedulePage from './views/SchedulePage.vue'
import FoodPage from './views/FoodPage.vue'
import TodoPage from './views/TodoPage.vue'
import WorkoutPage from './views/WorkoutPage.vue'

// Реактивная переменная для хранения текущей страницы
const currentPage = ref('dashboard') // Начальная страница должна соответствовать одному из кейсов

// Функция, которая будет вызываться при событии 'change-page'
const updatePage = (pageName) => { // Изменил имя параметра для ясности
  console.log('Received change-page event:', pageName); // Добавьте это для отладки
  currentPage.value = pageName
}

// Вычисляемое свойство для динамического выбора компонента
const currentViewComponent = computed(() => {
  switch (currentPage.value) {
    case 'dashboard':
      return Dashboard
    case 'schedule': // Убедитесь, что вы используете 'schedule', если это ваш 'SchedulePage'
      return SchedulePage
    case 'food':
      return FoodPage
    case 'todo':
      return TodoPage
    case 'workout':
      return WorkoutPage
    default:
      console.warn("Unknown page:", currentPage.value, "Falling back to Dashboard.");
      return Dashboard // Компонент по умолчанию, если что-то пошло не так
  }
})
</script>