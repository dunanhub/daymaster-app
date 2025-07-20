<template>
  <section class="p-4 sm:p-6 bg-white dark:bg-gray-800 shadow-xl rounded-lg mb-8">
    <h2 class="text-3xl font-extrabold text-gray-900 dark:text-gray-100 mb-6 border-b-2 border-blue-500 pb-2">
      <i class="fas fa-calendar-alt text-blue-500 mr-3"></i> Ваше Расписание
    </h2>

    <div class="flex flex-wrap items-center justify-between mb-6 gap-4">
      <div class="flex-grow">
        <button @click="toggleAllDays"
                class="px-4 py-2 bg-gray-200 text-gray-800 rounded-lg shadow-sm hover:bg-gray-300 dark:bg-gray-700 dark:text-gray-200 dark:hover:bg-gray-600 transition ease-in-out duration-150">
          <i :class="[allDaysCollapsed ? 'fas fa-chevron-down' : 'fas fa-chevron-up']" class="mr-2"></i>
          {{ allDaysCollapsed ? 'Развернуть все' : 'Свернуть все' }}
        </button>
      </div>
      </div>

    <div v-if="Object.keys(groupedSchedule).length > 0" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      <div v-for="(dayEvents, day) in groupedSchedule" :key="day"
           class="bg-gray-50 dark:bg-gray-700 p-5 rounded-lg shadow-md border border-gray-200 dark:border-gray-600 flex flex-col">

        <div class="flex items-center justify-between mb-4 pb-2 border-b border-gray-300 dark:border-gray-600 cursor-pointer"
             @click="toggleDayCollapse(day)">
          <h3 class="text-2xl font-bold text-gray-800 dark:text-gray-100">
            {{ day }}
          </h3>
          <button class="p-1 rounded-full text-gray-600 dark:text-gray-300 hover:bg-gray-200 dark:hover:bg-gray-600 focus:outline-none">
            <i :class="[collapsedDays[day] ? 'fas fa-chevron-down' : 'fas fa-chevron-up']"></i>
          </button>
        </div>

        <transition name="accordion">
          <div v-show="!collapsedDays[day]" class="flex-grow space-y-4">
            <div v-for="(event, eventIdx) in dayEvents" :key="eventIdx"
                 class="bg-white dark:bg-gray-800 p-3 rounded-lg shadow-sm border border-gray-100 dark:border-gray-600">
              <p class="font-semibold text-lg text-blue-600 dark:text-blue-400">
                <i class="fas fa-clock mr-2 text-blue-500"></i> {{ event.time }}
              </p>
              <p class="text-gray-800 dark:text-gray-200 mt-1">
                <i class="fas fa-flag-checkered mr-2 text-green-500"></i> {{ event.event }}
              </p>
            </div>
          </div>
        </transition>

      </div>
    </div>

    <div v-else class="p-6 text-center text-gray-500 dark:text-gray-400 bg-white dark:bg-gray-800 rounded-lg shadow-md">
      <p class="text-xl mb-3"><i class="fas fa-exclamation-circle mr-2"></i> Нет данных расписания для отображения.</p>
      <p>Пожалуйста, убедитесь, что ваш CSV-файл `schedule.csv` находится в `src/assets/` и содержит корректные данные.</p>
      <p>Ожидаемая структура: **первая колонка - время, остальные - дни недели, а ячейки - события.**</p>
    </div>
  </section>
</template>

<script setup>
import { ref, onMounted, reactive, computed } from 'vue' // Импортируем reactive и computed
import Papa from 'papaparse'

const headers = ref([])
const rows = ref([])
const groupedSchedule = ref({})

// Состояние для сворачивания: reactive объект, где ключи - это дни недели, значения - true (свернуто) / false (развернуто)
const collapsedDays = reactive({});

// Computed свойство для определения, свернуты ли ВСЕ дни
const allDaysCollapsed = computed(() => {
  // Если нет дней, считаем что все свернуто (хотя по сути их нет)
  if (Object.keys(groupedSchedule.value).length === 0) return true;
  // Проверяем, что каждый день либо отсутствует в collapsedDays, либо имеет значение true
  return Object.keys(groupedSchedule.value).every(day => collapsedDays[day]);
});

// Функция для переключения состояния сворачивания конкретного дня
const toggleDayCollapse = (day) => {
  collapsedDays[day] = !collapsedDays[day];
};

// Функция для сворачивания/разворачивания всех дней
const toggleAllDays = () => {
  const targetState = !allDaysCollapsed.value; // Если сейчас все свернуто, развернуть. Иначе - свернуть.
  Object.keys(groupedSchedule.value).forEach(day => {
    collapsedDays[day] = targetState;
  });
};


// Функция для обработки и группировки данных из вашего специфического CSV
const processScheduleData = (rawHeaders, rawRows) => {
  const schedule = {};
  const dayNames = rawHeaders.slice(1); // Дни недели начинаются со второй колонки

  // Инициализируем объект расписания для каждого дня
  dayNames.forEach(day => {
    schedule[day] = [];
    // Инициализируем состояние сворачивания по умолчанию (развернуто)
    collapsedDays[day] = false;
  });

  rawRows.forEach(row => {
    const time = row[0]; // Первая колонка - время
    if (!time) return;

    dayNames.forEach((dayName, index) => {
      const eventContent = row[index + 1];
      if (eventContent && eventContent.trim() !== '' && eventContent.trim() !== '–') {
        const cleanEvent = eventContent.replace(/^(\d{2}:\d{2}\s*–\s*|\d{1,2}:\d{2}\s*–\s*)/, '').trim();

        schedule[dayName].push({
          time: time,
          event: cleanEvent
        });
      }
    });
  });

  const dayOrder = ['Понедельник', 'Вторник', 'Среда', 'Четверг', 'Пятница', 'Суббота', 'Воскресенье'];
  const orderedSchedule = {};
  dayOrder.forEach(day => {
    if (schedule[day] && schedule[day].length > 0) {
      schedule[day].sort((a, b) => a.time.localeCompare(b.time));
      orderedSchedule[day] = schedule[day];
    }
  });

  for (const day in schedule) {
    if (!orderedSchedule[day] && schedule[day].length > 0) {
      orderedSchedule[day] = schedule[day];
    }
  }

  return orderedSchedule;
};


onMounted(async () => {
  try {
    const res = await fetch('/daymaster-app/schedule.csv')
    if (!res.ok) {
      throw new Error(`HTTP error! status: ${res.status}`);
    }
    const text = await res.text()
    const parsed = Papa.parse(text, { skipEmptyLines: true })

    if (parsed.data.length > 0) {
      headers.value = parsed.data[0];
      rows.value = parsed.data.slice(1);
      groupedSchedule.value = processScheduleData(headers.value, rows.value);
    } else {
      console.warn('CSV file is empty or contains no data rows.')
      groupedSchedule.value = {};
    }
  } catch (error) {
    console.error('Error loading or parsing CSV:', error)
    groupedSchedule.value = {};
  }
})
</script>

<style scoped>
/* Анимация аккордеона */
.accordion-enter-active,
.accordion-leave-active {
  transition: all 0.3s ease-out; /* Увеличено время для лучшего эффекта */
  overflow: hidden;
}

.accordion-enter-from,
.accordion-leave-to {
  max-height: 0;
  opacity: 0;
  transform: translateY(-10px); /* Небольшое смещение для эффекта */
}

.accordion-enter-to,
.accordion-leave-from {
  max-height: 500px; /* Достаточно большое значение для большинства контента, можно увеличить при необходимости */
  opacity: 1;
  transform: translateY(0);
}
</style>