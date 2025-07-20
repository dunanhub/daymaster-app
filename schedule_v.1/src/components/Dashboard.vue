<template>
  <div class="grid grid-cols-1 gap-6 md:grid-cols-2 lg:grid-cols-4">
    <div class="bg-white dark:bg-gray-800 p-4 shadow-xl rounded-lg transition-colors duration-300">
      <h3 class="text-lg font-semibold mb-2 text-gray-900 dark:text-gray-100 flex items-center">
        <i class="fas fa-play-circle text-blue-500 mr-2"></i> Сейчас
      </h3>
      <p v-if="currentTask" class="text-2xl font-bold text-blue-600 dark:text-blue-400">
        {{ currentTask.time }} – {{ currentTask.activity }}
      </p>
      <p v-else class="text-2xl font-bold text-blue-600 dark:text-blue-400">—</p>
      <p class="text-gray-500 dark:text-gray-300 text-sm">
        {{ currentDayOfWeek }} ({{ formattedCurrentTime }})
      </p>
    </div>

    <div class="bg-white dark:bg-gray-800 p-4 shadow-xl rounded-lg transition-colors duration-300">
      <h3 class="text-lg font-semibold mb-2 text-gray-900 dark:text-gray-100 flex items-center">
        <i class="fas fa-forward text-green-500 mr-2"></i> Следующее
      </h3>
      <p v-if="nextTask" class="text-2xl font-bold text-green-600 dark:text-green-400">
        {{ nextTask.time }} – {{ nextTask.activity }}
      </p>
      <p v-else class="text-2xl font-bold text-green-600 dark:text-green-400">
        —
      </p>
      <p class="text-gray-500 dark:text-gray-300 text-sm">
        После текущей активности
      </p>
    </div>

    <div class="bg-white dark:bg-gray-800 p-4 shadow-xl rounded-lg transition-colors duration-300">
      <h3 class="text-lg font-semibold mb-2 text-gray-900 dark:text-gray-100 flex items-center">
        <i class="fas fa-utensils text-purple-500 mr-2"></i> Что кушать
      </h3>
      <p v-if="currentMeal" class="text-2xl font-bold text-purple-600 dark:text-purple-400">
        {{ currentMeal.mealType }}: {{ currentMeal.foodItem }}
      </p>
      <p v-else class="text-2xl font-bold text-purple-600 dark:text-purple-400">
        —
      </p>
      <p class="text-gray-500 dark:text-gray-300 text-sm">
        Согласно вашему плану питания
      </p>
    </div>

    <div class="bg-white dark:bg-gray-800 p-4 shadow-xl rounded-lg transition-colors duration-300">
      <h3 class="text-lg font-semibold mb-2 text-gray-900 dark:text-gray-100 flex items-center">
        <i class="fas fa-tint text-blue-400 mr-2"></i> Счётчик воды
      </h3>
      <div class="flex items-center justify-between space-x-4">
        <span class="text-2xl font-bold text-gray-900 dark:text-gray-100">
          {{ waterCups }} {{ getCupText(waterCups) }}
        </span>
        <button
          @click="incrementWater"
          class="bg-blue-600 dark:bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-700 dark:hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 dark:focus:ring-offset-gray-700 transition ease-in-out duration-150"
        >
          <i class="fas fa-plus mr-1"></i> +1
        </button>
      </div>
      <p class="text-gray-500 dark:text-gray-300 text-sm mt-2">
        Общий объем: <span class="font-semibold text-gray-700 dark:text-gray-200">{{ waterLiters.toFixed(2) }} л</span>
      </p>
      <p class="text-gray-500 dark:text-gray-300 text-sm">Ваш прогресс по воде на сегодня</p>
    </div>

    <div class="bg-white dark:bg-gray-800 p-4 shadow-xl rounded-lg md:col-span-2 lg:col-span-4 transition-colors duration-300">
      <h3 class="text-lg font-semibold mb-2 text-gray-900 dark:text-gray-100 flex items-center">
        <i class="fas fa-exclamation-triangle text-red-500 mr-2"></i> Срочные задачи (To-Do)
      </h3>
      <ul v-if="urgentTasks.length > 0" class="list-disc pl-5 text-gray-700 dark:text-gray-300 space-y-1">
        <li v-for="task in urgentTasks" :key="task.id">{{ task.text }}</li>
      </ul>
      <p v-else class="text-gray-500 dark:text-gray-400 text-sm">Нет срочных задач на сегодня.</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, watchEffect } from 'vue';
import Papa from 'papaparse';

// --- Состояние и данные ---
const currentTime = ref(new Date());
const scheduleData = ref([]); // Для хранения расписания из CSV
const foodData = ref({});     // Для хранения меню из CSV
const waterCups = ref(0);     // Счетчик воды
const todoTasks = ref([]);    // Для хранения задач из Local Storage

const LOCAL_STORAGE_WATER_KEY = 'daily-water-cups';
const LOCAL_STORAGE_WATER_HISTORY_KEY = 'water-cups-history'; // НОВОЕ: ключ для истории
const LOCAL_STORAGE_TODO_KEY = 'todo-list-tasks'; // Ключ для задач из TodoPage
const CUP_VOLUME_LITERS = 0.25; // Объем одной чашки в литрах (250 мл)

// --- Загрузка данных ---
const loadSchedule = async () => {
  try {
    const res = await fetch('/schedule.csv');
    if (!res.ok) throw new Error(`HTTP error! status: ${res.status}`);
    const text = await res.text();
    const parsed = Papa.parse(text, { skipEmptyLines: true });
    if (parsed.data.length > 0) {
      const headers = parsed.data[0];
      const rows = parsed.data.slice(1);
      scheduleData.value = rows.map(row => {
        const entry = { time: row[0] };
        headers.slice(1).forEach((header, index) => {
          entry[header.split(' ')[0]] = row[index + 1]; // "Понедельник", "Вторник" и т.д.
        });
        return entry;
      });
    }
  } catch (error) {
    console.error('Error loading schedule.csv:', error);
  }
};

const loadFoods = async () => {
  try {
    const res = await fetch('/foods.csv');
    if (!res.ok) throw new Error(`HTTP error! status: ${res.status}`);
    const text = await res.text();
    const parsed = Papa.parse(text, { skipEmptyLines: true });
    if (parsed.data.length > 0) {
      const headers = parsed.data[0]; // Дни недели
      const rows = parsed.data.slice(1); // Приемы пищи

      const parsedFood = {};
      headers.slice(1).forEach((dayHeader, dayIndex) => {
        parsedFood[dayHeader] = {}; // { 'Понедельник': {}, 'Вторник': {} }
        rows.forEach(row => {
          const mealType = row[0]; // "Завтрак", "Обед"
          if (mealType && row[dayIndex + 1]) {
            parsedFood[dayHeader][mealType] = row[dayIndex + 1].trim();
          }
        });
      });
      foodData.value = parsedFood;
    }
  } catch (error) {
    console.error('Error loading foods.csv:', error);
  }
};

const loadWaterCount = () => {
  const savedWater = localStorage.getItem(LOCAL_STORAGE_WATER_KEY);
  const today = new Date().toDateString(); // Получаем строку текущей даты (без времени)

  if (savedWater) {
    const { count, date } = JSON.parse(savedWater);
    const savedDate = new Date(date).toDateString(); // Дата из сохраненной записи

    if (savedDate === today) {
      // Если это тот же день, что и последняя запись, просто загружаем счетчик
      waterCups.value = count;
    } else {
      // НОВОЕ: Если это новый день, сохраняем результат предыдущего дня в историю
      if (count > 0) { // Сохраняем только если что-то было выпито
        saveWaterHistoryEntry({ date: savedDate, count: count });
      }
      // Сбрасываем счетчик на 0 для нового дня
      waterCups.value = 0;
      // И сразу же сохраняем пустое состояние для нового дня, чтобы зафиксировать дату
      saveWaterCount();
    }
  } else {
    // Если записей нет вообще, счетчик 0
    waterCups.value = 0;
  }
};

const saveWaterCount = () => {
  localStorage.setItem(LOCAL_STORAGE_WATER_KEY, JSON.stringify({
    count: waterCups.value,
    date: new Date().toISOString() // Сохраняем полную дату со временем
  }));
};

// НОВОЕ: Функция для сохранения записи в историю воды
const saveWaterHistoryEntry = (entry) => {
  try {
    let history = JSON.parse(localStorage.getItem(LOCAL_STORAGE_WATER_HISTORY_KEY) || '[]');
    // Убедимся, что не дублируем запись для той же даты
    const existingIndex = history.findIndex(h => new Date(h.date).toDateString() === new Date(entry.date).toDateString());
    if (existingIndex !== -1) {
      // Если запись для этой даты уже есть, обновляем ее
      history[existingIndex] = entry;
    } else {
      // Иначе добавляем новую запись
      history.push(entry);
    }
    // Опционально: можно ограничить размер истории, например, до 30 дней
    // history.sort((a, b) => new Date(b.date) - new Date(a.date)); // Сортируем по убыванию даты
    // history = history.slice(0, 30); // Оставляем только последние 30 дней
    localStorage.setItem(LOCAL_STORAGE_WATER_HISTORY_KEY, JSON.stringify(history));
    console.log(`Water history updated for ${entry.date}: ${entry.count} cups`);
  } catch (e) {
    console.error('Error saving water history:', e);
  }
};

// НОВОЕ: Функция для загрузки истории воды (пока не используется в UI)
const loadWaterHistory = () => {
  try {
    const history = JSON.parse(localStorage.getItem(LOCAL_STORAGE_WATER_HISTORY_KEY) || '[]');
    return history;
  } catch (e) {
    console.error('Error loading water history:', e);
    return [];
  }
};


const loadTodoTasks = () => {
  try {
    const savedTasks = localStorage.getItem(LOCAL_STORAGE_TODO_KEY);
    if (savedTasks) {
      todoTasks.value = JSON.parse(savedTasks);
    } else {
      todoTasks.value = [];
    }
  } catch (e) {
    console.error('Ошибка при загрузке To-Do задач:', e);
    todoTasks.value = [];
  }
};

// --- Функции счетчика воды ---
const incrementWater = () => {
  waterCups.value++;
  saveWaterCount();
};

const getCupText = (count) => {
  if (count % 10 === 1 && count % 100 !== 11) return 'чашка';
  if ([2, 3, 4].includes(count % 10) && ![12, 13, 14].includes(count % 100)) return 'чашки';
  return 'чашек';
};

const waterLiters = computed(() => {
  return waterCups.value * CUP_VOLUME_LITERS;
});


// --- Вычисляемые свойства для времени и задач ---
const currentDayOfWeek = computed(() => {
  const days = ['Воскресенье', 'Понедельник', 'Вторник', 'Среда', 'Четверг', 'Пятница', 'Суббота'];
  return days[currentTime.value.getDay()];
});

const formattedCurrentTime = computed(() => {
  const hours = String(currentTime.value.getHours()).padStart(2, '0');
  const minutes = String(currentTime.value.getMinutes()).padStart(2, '0');
  return `${hours}:${minutes}`;
});

const getNearestScheduleEntry = (isNext = false) => {
  const currentMinutes = currentTime.value.getHours() * 60 + currentTime.value.getMinutes();
  let nearestEntry = null;
  let minDiff = Infinity;

  // Фильтруем расписание по текущему дню
  const todaySchedule = scheduleData.value.map(entry => {
    const timeParts = entry.time.split(':');
    const entryMinutes = parseInt(timeParts[0]) * 60 + parseInt(timeParts[1]);
    return {
      time: entry.time,
      activity: entry[currentDayOfWeek.value],
      minutes: entryMinutes
    };
  }).filter(entry => entry.activity && entry.activity.trim() !== '' && entry.activity.trim() !== '–');

  todaySchedule.sort((a, b) => a.minutes - b.minutes); // Сортируем по времени

  for (const entry of todaySchedule) {
    const diff = entry.minutes - currentMinutes;

    if (isNext) {
      // Для "Следующее": ищем первую активность, которая начнется ПОСЛЕ текущего времени
      if (diff > 0 && diff < minDiff) {
        minDiff = diff;
        nearestEntry = entry;
      }
    } else {
      // Для "Сейчас": ищем активность, которая уже началась или начнется в ближайшем будущем (до 30 минут вперед)
      // или последнюю, которая уже должна была начаться
      if (diff <= 0) { // Активность уже началась или идет сейчас
        nearestEntry = entry; // Последняя активность, которая началась или должна была начаться
      } else if (diff > 0 && diff < 60) { // Активность начнется в течение часа
         if (!nearestEntry || (entry.minutes - nearestEntry.minutes < diff)) {
           nearestEntry = entry; // Выбираем ближайшую, если текущая еще не найдена или эта ближе
         }
      }
    }
  }

  // Если для "Сейчас" не нашли, но есть ближайшая в будущем (до 30-45 минут)
  if (!isNext && !nearestEntry && todaySchedule.length > 0) {
      const firstFutureEntry = todaySchedule.find(entry => entry.minutes > currentMinutes);
      if(firstFutureEntry && (firstFutureEntry.minutes - currentMinutes) <= 45){ // Если в ближайшие 45 минут
          nearestEntry = firstFutureEntry;
      }
  }

  return nearestEntry;
};

const currentTask = computed(() => getNearestScheduleEntry(false));
const nextTask = computed(() => {
  const currentTaskEntry = currentTask.value;
  if (!currentTaskEntry) return null;

  const currentTaskTimeMinutes = currentTaskEntry.minutes;
  const currentDaySchedule = scheduleData.value.map(entry => {
    const timeParts = entry.time.split(':');
    const entryMinutes = parseInt(timeParts[0]) * 60 + parseInt(timeParts[1]);
    return {
      time: entry.time,
      activity: entry[currentDayOfWeek.value],
      minutes: entryMinutes
    };
  }).filter(entry => entry.activity && entry.activity.trim() !== '' && entry.activity.trim() !== '–');

  currentDaySchedule.sort((a, b) => a.minutes - b.minutes);

  const currentIndex = currentDaySchedule.findIndex(entry => entry.time === currentTaskEntry.time && entry.activity === currentTaskEntry.activity);
  if (currentIndex !== -1 && currentIndex + 1 < currentDaySchedule.length) {
    return currentDaySchedule[currentIndex + 1];
  }
  return null;
});


const currentMeal = computed(() => {
  if (!foodData.value || !foodData.value[currentDayOfWeek.value]) {
    return null;
  }

  const currentHour = currentTime.value.getHours();
  const currentMinute = currentTime.value.getMinutes();

  // Простые диапазоны времени для определения типа приема пищи
  const mealTimeRanges = {
    'Завтрак': { start: 6, end: 9, minEnd: 30 }, // 6:00 - 9:30
    'Обед': { start: 12, end: 14, minEnd: 30 },  // 12:00 - 14:30
    'Ужин': { start: 18, end: 20, minEnd: 30 },  // 18:00 - 20:30
    'Перекус': { start: 9, end: 12, minStart: 31 }, // 9:31 - 12:00
    'Полдник': { start: 14, end: 18, minStart: 31 }  // 14:31 - 18:00
  };

  const todayMeals = foodData.value[currentDayOfWeek.value];

  // Определяем текущий тип приема пищи на основе времени
  let mealTypeToFind = null;
  for (const type in mealTimeRanges) {
    const range = mealTimeRanges[type];
    const isStart = currentHour > range.start || (currentHour === range.start && (!range.minStart || currentMinute >= range.minStart));
    const isEnd = currentHour < range.end || (currentHour === range.end && (!range.minEnd || currentMinute <= range.minEnd));

    if (isStart && isEnd) {
      mealTypeToFind = type;
      break;
    }
  }

  if (mealTypeToFind && todayMeals[mealTypeToFind]) {
    return {
      mealType: mealTypeToFind,
      foodItem: todayMeals[mealTypeToFind]
    };
  }

  // Если текущая активность из расписания указывает на прием пищи, используем это
  if (currentTask.value && currentTask.value.activity) {
    const activityLower = currentTask.value.activity.toLowerCase();
    if (activityLower.includes('завтрак') && todayMeals['Завтрак']) {
      return { mealType: 'Завтрак', foodItem: todayMeals['Завтрак'] };
    }
    if (activityLower.includes('обед') && todayMeals['Обед']) {
      return { mealType: 'Обед', foodItem: todayMeals['Обед'] };
    }
    if (activityLower.includes('ужин') && todayMeals['Ужин']) {
      return { mealType: 'Ужин', foodItem: todayMeals['Ужин'] };
    }
    // Можно добавить логику для "перекуса" и "полдника" если они явно в расписании
  }

  return null;
});


const urgentTasks = computed(() => {
  return todoTasks.value.filter(task => task.status === 'urgent');
});

// --- Жизненный цикл и интервал обновления времени ---
let timeInterval;

onMounted(() => {
  loadSchedule();
  loadFoods();
  loadWaterCount(); // Важно: вызывается один раз при монтировании для инициализации
  loadTodoTasks(); // Загружаем задачи при монтировании

  // Обновляем время каждую минуту
  timeInterval = setInterval(() => {
    currentTime.value = new Date();
    // При каждом обновлении времени, также проверяем, не сбросился ли день для счетчика воды
    // Вызов loadWaterCount здесь гарантирует, что переход дня будет отловлен
    loadWaterCount();
  }, 60 * 1000); // Каждую минуту
});

onUnmounted(() => {
  clearInterval(timeInterval);
});

// Watcher, чтобы перезагружать todoTasks, если они изменились в localStorage (например, на TodoPage)
watchEffect(() => {
  window.addEventListener('storage', (event) => {
    if (event.key === LOCAL_STORAGE_TODO_KEY) {
      loadTodoTasks();
    }
    // При изменении water counter в другой вкладке, также обновляем его здесь.
    // Важно: эта часть не должна вызывать saveWaterHistoryEntry, так как это делается только при смене дня.
    if (event.key === LOCAL_STORAGE_WATER_KEY) {
      // Простая перезагрузка текущего значения, без логики истории
      const savedWater = localStorage.getItem(LOCAL_STORAGE_WATER_KEY);
      if (savedWater) {
        const { count, date } = JSON.parse(savedWater);
        if (new Date(date).toDateString() === new Date().toDateString()) {
          waterCups.value = count;
        }
        // Если дата в localStorage изменилась на другую, loadWaterCount() при следующем вызове
        // уже обработает сохранение истории и сброс.
      }
    }
  });
});
</script>

<style scoped>
/* Кастомные стили, если понадобятся */
</style>