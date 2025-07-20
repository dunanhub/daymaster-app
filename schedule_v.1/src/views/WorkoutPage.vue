<template>
  <section class="p-4 sm:p-6 bg-white dark:bg-gray-800 shadow-xl rounded-lg mb-8">
    <h2 class="text-3xl font-extrabold text-gray-900 dark:text-gray-100 mb-6 border-b-2 border-orange-500 pb-2">
      <i class="fas fa-dumbbell text-orange-500 mr-3"></i> Ваш План Тренировок
    </h2>

    <div class="flex flex-wrap items-center justify-start mb-6 gap-4">
      <button @click="toggleWorkoutSectionCollapse"
              class="px-4 py-2 bg-gray-200 text-gray-800 rounded-lg shadow-sm hover:bg-gray-300 dark:bg-gray-700 dark:text-gray-200 dark:hover:bg-gray-600 transition ease-in-out duration-150">
        <i :class="[workoutSectionCollapsed ? 'fas fa-chevron-down' : 'fas fa-chevron-up']" class="mr-2"></i>
        {{ workoutSectionCollapsed ? 'Развернуть план' : 'Свернуть план' }}
      </button>

      <button v-show="!workoutSectionCollapsed" @click="toggleAllDays"
              class="px-4 py-2 bg-gray-200 text-gray-800 rounded-lg shadow-sm hover:bg-gray-300 dark:bg-gray-700 dark:text-gray-200 dark:hover:bg-gray-600 transition ease-in-out duration-150">
        <i :class="[allDaysCollapsed ? 'fas fa-chevron-down' : 'fas fa-chevron-up']" class="mr-2"></i>
        {{ allDaysCollapsed ? 'Развернуть все дни' : 'Свернуть все дни' }}
      </button>
    </div>

    <transition name="accordion">
      <div v-show="!workoutSectionCollapsed">
        <div v-if="Object.keys(groupedWorkouts).length > 0" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
          <div v-for="(exercises, day) in groupedWorkouts" :key="day"
               class="bg-gray-50 dark:bg-gray-700 p-5 rounded-lg shadow-md border border-gray-200 dark:border-gray-600 flex flex-col">

            <div class="flex items-center justify-between mb-4 pb-2 border-b border-gray-300 dark:border-gray-600 cursor-pointer"
                 @click="toggleDayCollapse(day)">
              <h3 class="text-xl font-bold text-gray-800 dark:text-gray-100">
                {{ day }}
              </h3>
              <button class="p-1 rounded-full text-gray-600 dark:text-gray-300 hover:bg-gray-200 dark:hover:bg-gray-600 focus:outline-none">
                <i :class="[collapsedDays[day] ? 'fas fa-chevron-down' : 'fas fa-chevron-up']"></i>
              </button>
            </div>

            <transition name="accordion">
              <div v-show="!collapsedDays[day]" class="flex-grow space-y-3">
                <div v-for="(exercise, idx) in exercises" :key="idx"
                     class="bg-white dark:bg-gray-800 p-3 rounded-lg shadow-sm border border-gray-100 dark:border-gray-600">
                  <p class="text-gray-800 dark:text-gray-200 text-sm">
                    <i class="fas fa-running mr-2 text-blue-500"></i> {{ exercise }}
                  </p>
                </div>
              </div>
            </transition>
          </div>
        </div>

        <div v-else class="p-6 text-center text-gray-500 dark:text-gray-400 bg-white dark:bg-gray-800 rounded-lg shadow-md">
          <p class="text-xl mb-3"><i class="fas fa-exclamation-circle mr-2"></i> Нет данных о тренировках для отображения.</p>
          <p>Пожалуйста, убедитесь, что ваш CSV-файл `WorkOut.csv` находится в `src/assets/` и содержит корректные данные.</p>
          <p>Ожидаемая структура: **заголовки колонок - дни недели/типы тренировок, ячейки - упражнения.**</p>
        </div>
      </div>
    </transition>
  </section>
</template>

<script setup>
import { ref, onMounted, reactive, computed } from 'vue'
import Papa from 'papaparse'

const headers = ref([])
const rows = ref([])
const groupedWorkouts = ref({}) // Изменено название для ясности

// Состояние для сворачивания: reactive объект, где ключи - это дни недели, значения - true (свернуто) / false (развернуто)
const collapsedDays = reactive({});

// Computed свойство для определения, свернуты ли ВСЕ дни
const allDaysCollapsed = computed(() => {
  if (Object.keys(groupedWorkouts.value).length === 0) return true;
  return Object.keys(groupedWorkouts.value).every(day => collapsedDays[day]);
});

// НОВОЕ: Состояние для сворачивания всего блока "План Тренировок"
const workoutSectionCollapsed = ref(false); // По умолчанию развернуто
const toggleWorkoutSectionCollapse = () => {
  workoutSectionCollapsed.value = !workoutSectionCollapsed.value;
};

// Функция для переключения состояния сворачивания конкретного дня
const toggleDayCollapse = (day) => {
  collapsedDays[day] = !collapsedDays[day];
};

// Функция для сворачивания/разворачивания всех дней
const toggleAllDays = () => {
  const targetState = !allDaysCollapsed.value;
  Object.keys(groupedWorkouts.value).forEach(day => {
    collapsedDays[day] = targetState;
  });
};

// Функция для обработки и группировки данных из WorkOut.csv
const processWorkoutData = (rawHeaders, rawRows) => {
  const workoutSchedule = {};
  const dayNames = rawHeaders; // Заголовки - это дни тренировок

  // Инициализируем объект расписания для каждого дня
  dayNames.forEach(day => {
    if (day && day.trim() !== '') { // Убедимся, что день не пустой
      workoutSchedule[day.trim()] = [];
      // Инициализируем состояние сворачивания по умолчанию (развернуто)
      collapsedDays[day.trim()] = false;
    }
  });

  rawRows.forEach(row => {
    dayNames.forEach((dayName, index) => {
      if (!dayName || dayName.trim() === '') return; // Пропускаем пустые заголовки

      const exercise = row[index]; // Упражнение в соответствующей колонке дня
      if (exercise && exercise.trim() !== '' && exercise.trim() !== '–') { // Пропускаем пустые или '-' ячейки
        if (workoutSchedule[dayName.trim()]) { // Проверяем, что день существует
          workoutSchedule[dayName.trim()].push(exercise.trim());
        }
      }
    });
  });

  // Отсортируем дни недели, если нужно, или просто вернем как есть
  // Если у вас есть фиксированный порядок дней (Пн, Вт, ..., Вс), можно использовать:
  // const dayOrder = ['Понедельник (Full Body)', 'Вторник (Ноги + Пресс)', 'Среда', 'Четверг', 'Пятница', 'Суббота (Вечер — Верх тела)', 'Воскресенье (Full Body + Лёгкая нагрузка)'];
  // const orderedWorkoutSchedule = {};
  // dayOrder.forEach(day => {
  //   if (workoutSchedule[day] && workoutSchedule[day].length > 0) {
  //     orderedWorkoutSchedule[day] = workoutSchedule[day];
  //   }
  // });
  // for (const day in workoutSchedule) { // Добавим дни, которые могли быть в CSV, но не в нашем dayOrder
  //   if (!orderedWorkoutSchedule[day] && workoutSchedule[day].length > 0) {
  //     orderedWorkoutSchedule[day] = workoutSchedule[day];
  //   }
  // }
  // return orderedWorkoutSchedule;

  return workoutSchedule; // Возвращаем как есть, без строгой сортировки по дням недели
};


onMounted(async () => {
  try {
    const res = await fetch('/daymaster-app/WorkOut.csv') // Изменено на WorkOut.csv
    if (!res.ok) {
      throw new Error(`HTTP error! status: ${res.status}`);
    }
    const text = await res.text()
    const parsed = Papa.parse(text, { skipEmptyLines: true })

    if (parsed.data.length > 0) {
      headers.value = parsed.data[0];
      rows.value = parsed.data.slice(1);
      groupedWorkouts.value = processWorkoutData(headers.value, rows.value);
    } else {
      console.warn('CSV file is empty or contains no data rows.')
      groupedWorkouts.value = {};
    }
  } catch (error) {
    console.error('Error loading or parsing WorkOut.csv:', error)
    groupedWorkouts.value = {};
  }
})
</script>

<style scoped>
/* Анимация аккордеона - те же стили, что и для SchedulePage */
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
  max-height: 1000px; /* Увеличьте, если у вас очень много упражнений в день */
  opacity: 1;
  transform: translateY(0);
}
</style>