<template>
  <section class="p-4 sm:p-6 bg-white dark:bg-gray-800 shadow-xl rounded-lg mb-8">
    <h2 class="text-3xl font-extrabold text-gray-900 dark:text-gray-100 mb-6 border-b-2 border-blue-500 pb-2">
      <i class="fas fa-clipboard-list text-blue-500 mr-3"></i> Ваш To-Do Список
    </h2>

    <form @submit.prevent="addTask" class="mb-8 p-4 bg-gray-100 dark:bg-gray-700 rounded-lg shadow-md flex flex-col sm:flex-row gap-4">
      <input
        v-model="newTaskText"
        type="text"
        placeholder="Добавить новую задачу..."
        class="flex-grow p-3 rounded-md border border-gray-300 dark:border-gray-600 dark:bg-gray-800 dark:text-gray-100 focus:outline-none focus:ring-2 focus:ring-blue-500"
        required
      />
      <select
        v-model="newTaskStatus"
        class="p-3 rounded-md border border-gray-300 dark:border-gray-600 dark:bg-gray-800 dark:text-gray-100 focus:outline-none focus:ring-2 focus:ring-blue-500"
      >
        <option v-for="status in statuses" :key="status.key" :value="status.key">
          {{ status.title }}
        </option>
      </select>
      <button
        type="submit"
        class="px-6 py-3 bg-blue-600 text-white rounded-md shadow-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 dark:focus:ring-offset-gray-700 transition ease-in-out duration-150"
      >
        <i class="fas fa-plus mr-2"></i> Добавить
      </button>
    </form>

    <div class="mb-6 flex flex-wrap gap-3">
        <button @click="toggleAllColumns"
                class="px-4 py-2 bg-gray-200 text-gray-800 rounded-lg shadow-sm hover:bg-gray-300 dark:bg-gray-700 dark:text-gray-200 dark:hover:bg-gray-600 transition ease-in-out duration-150">
          <i :class="[allColumnsCollapsed ? 'fas fa-chevron-down' : 'fas fa-chevron-up']" class="mr-2"></i>
          {{ allColumnsCollapsed ? 'Развернуть все колонки' : 'Свернуть все колонки' }}
        </button>
        <button @click="saveTasks"
                class="px-4 py-2 bg-green-600 text-white rounded-lg shadow-sm hover:bg-green-700 dark:bg-green-700 dark:hover:bg-green-600 transition ease-in-out duration-150">
          <i class="fas fa-save mr-2"></i> Сохранить
        </button>
        <button @click="loadTasks"
                class="px-4 py-2 bg-purple-600 text-white rounded-lg shadow-sm hover:bg-purple-700 dark:bg-purple-700 dark:hover:bg-purple-600 transition ease-in-out duration-150">
          <i class="fas fa-folder-open mr-2"></i> Загрузить
        </button>
        <button @click="clearAllTasks"
                class="px-4 py-2 bg-red-600 text-white rounded-lg shadow-sm hover:bg-red-700 dark:bg-red-700 dark:hover:bg-red-600 transition ease-in-out duration-150">
          <i class="fas fa-trash-alt mr-2"></i> Очистить все
        </button>
    </div>

    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
      <div v-for="status in statuses" :key="status.key"
           class="bg-gray-100 dark:bg-gray-700 p-4 rounded-lg shadow-md flex flex-col min-h-[150px]"
           @dragover.prevent="onDragOver(status.key)"
           @drop="onDrop(status.key)">
        
        <div class="flex items-center justify-between mb-4 pb-2 border-b-2"
             :class="getStatusHeaderClass(status.key)"
             @click="toggleColumnCollapse(status.key)">
          <h3 class="text-xl font-semibold text-gray-800 dark:text-gray-100 flex-grow cursor-pointer">
            <i :class="status.icon" class="mr-2"></i> {{ status.title }} ({{ getTasksByStatus(status.key).length }})
          </h3>
          <button class="p-1 rounded-full text-gray-600 dark:text-gray-300 hover:bg-gray-300 dark:hover:bg-gray-600 focus:outline-none">
            <i :class="[collapsedColumns[status.key] ? 'fas fa-chevron-down' : 'fas fa-chevron-up']"></i>
          </button>
        </div>

        <transition name="accordion">
          <div v-show="!collapsedColumns[status.key]" class="flex-grow space-y-3">
            <div v-if="getTasksByStatus(status.key).length > 0">
              <div
                v-for="task in getTasksByStatus(status.key)"
                :key="task.id"
                :class="getTaskCardClass(status.key)"
                draggable="true"
                @dragstart="onDragStart(task)"
                class="p-3 rounded-lg shadow-sm flex items-center justify-between cursor-grab transform transition-all duration-200 hover:scale-[1.02]"
              >
                <span class="text-gray-800 dark:text-gray-100 text-sm flex-grow break-words pr-2">{{ task.text }}</span>
                <button
                  @click="removeTask(task.id)"
                  class="ml-2 p-1 text-red-500 hover:text-red-700 dark:text-red-400 dark:hover:text-red-300 rounded-full focus:outline-none focus:ring-2 focus:ring-red-500"
                >
                  <i class="fas fa-times"></i>
                </button>
              </div>
            </div>
            <div v-else class="text-center text-gray-500 dark:text-gray-400 p-4 text-sm">
              Нет задач в этой колонке.
            </div>
          </div>
        </transition>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue';

// --- Данные состояния ---
const tasks = ref([]); // Массив всех задач
const newTaskText = ref(''); // Текст новой задачи
const newTaskStatus = ref('today'); // Начальный статус для новой задачи (по умолчанию "Сегодня")

// Определяем статусы/колонки
const statuses = reactive([
  { key: 'urgent', title: 'Срочно', icon: 'fas fa-exclamation-circle', headerColor: 'border-red-500' },
  { key: 'today', title: 'Сегодня', icon: 'fas fa-sun', headerColor: 'border-yellow-500' },
  { key: 'in-progress', title: 'В процессе', icon: 'fas fa-spinner fa-spin', headerColor: 'border-blue-500' },
  { key: 'planned', title: 'В плане', icon: 'fas fa-calendar-alt', headerColor: 'border-gray-500' },
  { key: 'done', title: 'Выполнено', icon: 'fas fa-check-circle', headerColor: 'border-green-500' },
  { key: 'deferred', title: 'Отложено', icon: 'fas fa-minus-circle', headerColor: 'border-purple-500' },
]);

// Состояние сворачивания колонок
const collapsedColumns = reactive({});
// Инициализация collapsedColumns (все развернуты по умолчанию)
statuses.forEach(status => {
  collapsedColumns[status.key] = false;
});

// Вычисляемое свойство для состояния "все колонки свернуты"
const allColumnsCollapsed = computed(() => {
  return statuses.every(status => collapsedColumns[status.key]);
});

// --- Функции управления задачами ---

// Добавление задачи
const addTask = () => {
  if (newTaskText.value.trim()) {
    tasks.value.push({
      id: Date.now(), // Простой уникальный ID
      text: newTaskText.value.trim(),
      status: newTaskStatus.value,
    });
    newTaskText.value = ''; // Очищаем поле ввода
    saveTasks(); // Сохраняем после добавления
  }
};

// Удаление задачи
const removeTask = (id) => {
  tasks.value = tasks.value.filter(task => task.id !== id);
  saveTasks(); // Сохраняем после удаления
};

// Получение задач по статусу
const getTasksByStatus = (statusKey) => {
  return tasks.value.filter(task => task.status === statusKey);
};

// --- Функции Drag & Drop ---
let draggedTask = null; // Для хранения перетаскиваемой задачи

const onDragStart = (task) => {
  draggedTask = task;
};

const onDrop = (targetStatusKey) => {
  if (draggedTask) {
    // Находим индекс задачи и обновляем её статус
    const index = tasks.value.findIndex(task => task.id === draggedTask.id);
    if (index !== -1) {
      tasks.value[index].status = targetStatusKey;
      saveTasks(); // Сохраняем после изменения статуса
    }
    draggedTask = null; // Сбрасываем перетаскиваемую задачу
  }
};

const onDragOver = (statusKey) => {
  // Просто чтобы позволить drop событию сработать
};

// --- Функции сохранения/загрузки ---
const LOCAL_STORAGE_KEY = 'todo-list-tasks';

const saveTasks = () => {
  try {
    localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(tasks.value));
    console.log('Задачи сохранены.');
  } catch (e) {
    console.error('Ошибка при сохранении задач:', e);
  }
};

const loadTasks = () => {
  try {
    const savedTasks = localStorage.getItem(LOCAL_STORAGE_KEY);
    if (savedTasks) {
      tasks.value = JSON.parse(savedTasks);
      console.log('Задачи загружены.');
    } else {
      tasks.value = [];
      console.log('Нет сохраненных задач.');
    }
  } catch (e) {
    console.error('Ошибка при загрузке задач:', e);
    tasks.value = []; // Очищаем задачи в случае ошибки парсинга
  }
};

const clearAllTasks = () => {
  if (confirm('Вы уверены, что хотите удалить все задачи?')) {
    tasks.value = [];
    saveTasks(); // Сохраняем пустое состояние
  }
};


// --- Функции сворачивания/разворачивания колонок ---
const toggleColumnCollapse = (columnKey) => {
  collapsedColumns[columnKey] = !collapsedColumns[columnKey];
};

const toggleAllColumns = () => {
  const targetState = !allColumnsCollapsed.value;
  statuses.forEach(status => {
    collapsedColumns[status.key] = targetState;
  });
};

// --- Динамические классы для стилизации ---
const getStatusHeaderClass = (statusKey) => {
  const status = statuses.find(s => s.key === statusKey);
  return status ? status.headerColor : ''; // Применяем цвет границы из статуса
};

const getTaskCardClass = (statusKey) => {
  // Цвет карточки задачи в зависимости от статуса
  switch (statusKey) {
    case 'urgent': return 'bg-red-100 dark:bg-red-800 border-red-200 dark:border-red-700';
    case 'today': return 'bg-yellow-100 dark:bg-yellow-800 border-yellow-200 dark:border-yellow-700';
    case 'in-progress': return 'bg-blue-100 dark:bg-blue-800 border-blue-200 dark:border-blue-700';
    case 'planned': return 'bg-gray-200 dark:bg-gray-600 border-gray-300 dark:border-gray-500';
    case 'done': return 'bg-green-100 dark:bg-green-800 border-green-200 dark:border-green-700';
    case 'deferred': return 'bg-purple-100 dark:bg-purple-800 border-purple-200 dark:border-purple-700';
    default: return 'bg-white dark:bg-gray-800 border-gray-100 dark:border-gray-600';
  }
};


// --- Жизненный цикл ---
onMounted(() => {
  loadTasks(); // Загружаем задачи при монтировании компонента
});
</script>

<style scoped>
/* Стили для анимации аккордеона */
.accordion-enter-active,
.accordion-leave-active {
  transition: all 0.3s ease-out;
  overflow: hidden;
}

.accordion-enter-from,
.accordion-leave-to {
  max-height: 0;
  opacity: 0;
  transform: translateY(-10px);
}

.accordion-enter-to,
.accordion-leave-from {
  max-height: 1000px; /* Достаточно большое значение для любого количества задач в колонке */
  opacity: 1;
  transform: translateY(0);
}
</style>