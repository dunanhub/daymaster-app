<template>
  <section class="p-4 sm:p-6 bg-white dark:bg-gray-800 shadow-xl rounded-lg mb-8">
    <h2 class="text-3xl font-extrabold text-gray-900 dark:text-gray-100 mb-6 border-b-2 border-green-500 pb-2">
      <i class="fas fa-utensils text-green-500 mr-3"></i> Ваше Меню на Неделю
    </h2>

    <div class="flex flex-wrap items-center justify-start mb-6 gap-4">
      <button @click="toggleFoodScheduleCollapse"
              class="px-4 py-2 bg-gray-200 text-gray-800 rounded-lg shadow-sm hover:bg-gray-300 dark:bg-gray-700 dark:text-gray-200 dark:hover:bg-gray-600 transition ease-in-out duration-150">
        <i :class="[foodScheduleCollapsed ? 'fas fa-chevron-down' : 'fas fa-chevron-up']" class="mr-2"></i>
        {{ foodScheduleCollapsed ? 'Развернуть меню' : 'Свернуть меню' }}
      </button>

      <button v-show="!foodScheduleCollapsed" @click="toggleAllDays"
              class="px-4 py-2 bg-gray-200 text-gray-800 rounded-lg shadow-sm hover:bg-gray-300 dark:bg-gray-700 dark:text-gray-200 dark:hover:bg-gray-600 transition ease-in-out duration-150">
        <i :class="[allDaysCollapsed ? 'fas fa-chevron-down' : 'fas fa-chevron-up']" class="mr-2"></i>
        {{ allDaysCollapsed ? 'Развернуть все дни' : 'Свернуть все дни' }}
      </button>

      </div>

    <transition name="accordion">
      <div v-show="!foodScheduleCollapsed">
        <div v-if="Object.keys(groupedFoodSchedule).length > 0" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
          <div v-for="(dayMeals, day) in groupedFoodSchedule" :key="day"
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
                <div v-for="(meal, mealIdx) in dayMeals" :key="mealIdx"
                     class="bg-white dark:bg-gray-800 p-3 rounded-lg shadow-sm border border-gray-100 dark:border-gray-600">
                  <p class="font-semibold text-lg text-green-600 dark:text-green-400">
                    <i class="fas fa-cookie-bite mr-2 text-green-500"></i> {{ meal.mealType }}
                  </p>
                  <p class="text-gray-800 dark:text-gray-200 mt-1">
                    <i class="fas fa-utensils mr-2 text-blue-500"></i> {{ meal.foodItem }}
                  </p>
                </div>
              </div>
            </transition>
          </div>
        </div>

        <div v-else class="p-6 text-center text-gray-500 dark:text-gray-400 bg-white dark:bg-gray-800 rounded-lg shadow-md">
          <p class="text-xl mb-3"><i class="fas fa-exclamation-circle mr-2"></i> Нет данных о меню для отображения.</p>
          <p>Пожалуйста, убедитесь, что ваш CSV-файл `foods.csv` находится в `src/assets/` и содержит корректные данные.</p>
          <p>Ожидаемая структура: **первая колонка - тип приема пищи, остальные - дни недели, а ячейки - блюда.**</p>
        </div>
      </div>
    </transition>

    <hr class="my-10 border-gray-300 dark:border-gray-700" />

    <div class="mt-8 p-4 sm:p-6 bg-white dark:bg-gray-800 shadow-xl rounded-lg">
      <div class="flex items-center justify-between cursor-pointer mb-4 pb-2 border-b-2 border-orange-500" @click="toggleRecipesCollapse">
        <h2 class="text-3xl font-extrabold text-gray-900 dark:text-gray-100">
          <i class="fas fa-book-open text-orange-500 mr-3"></i> Как готовить
        </h2>
        <button class="p-1 rounded-full text-gray-600 dark:text-gray-300 hover:bg-gray-200 dark:hover:bg-gray-600 focus:outline-none">
            <i :class="[recipesCollapsed ? 'fas fa-chevron-down' : 'fas fa-chevron-up']"></i>
        </button>
      </div>
      <transition name="accordion">
        <div v-show="!recipesCollapsed" class="mt-6 space-y-6">
          <div v-if="Object.keys(cookingMethods).length > 0" class="grid grid-cols-1 md:grid-cols-2 gap-6">
            <div v-for="(method, dish) in cookingMethods" :key="dish"
                 class="bg-gray-50 dark:bg-gray-700 p-4 rounded-lg shadow-sm border border-gray-200 dark:border-gray-600">
              <h3 class="text-xl font-bold text-gray-800 dark:text-gray-100 mb-2">{{ dish }}</h3>
              <p class="text-gray-700 dark:text-gray-200 text-sm leading-relaxed">{{ method }}</p>
            </div>
          </div>
          <div v-else class="p-4 text-center text-gray-500 dark:text-gray-400">
            <p><i class="fas fa-info-circle mr-2"></i> Данные о рецептах не найдены.</p>
            <p>Убедитесь, что `cookingMethods.csv` находится в `src/assets/`.</p>
          </div>
        </div>
      </transition>
    </div>

    <div class="mt-8 p-4 sm:p-6 bg-white dark:bg-gray-800 shadow-xl rounded-lg">
      <div class="flex items-center justify-between cursor-pointer mb-4 pb-2 border-b-2 border-purple-500" @click="toggleBuyListCollapse">
        <h2 class="text-3xl font-extrabold text-gray-900 dark:text-gray-100">
          <i class="fas fa-shopping-basket text-purple-500 mr-3"></i> Список покупок
        </h2>
        <button class="p-1 rounded-full text-gray-600 dark:text-gray-300 hover:bg-gray-200 dark:hover:bg-gray-600 focus:outline-none">
            <i :class="[buyListCollapsed ? 'fas fa-chevron-down' : 'fas fa-chevron-up']"></i>
        </button>
      </div>
      <transition name="accordion">
        <div v-show="!buyListCollapsed" class="mt-6 space-y-6">
          <div v-if="Object.keys(buyList).length > 0" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            <div v-for="(items, category) in buyList" :key="category"
                 class="bg-gray-50 dark:bg-gray-700 p-4 rounded-lg shadow-sm border border-gray-200 dark:border-gray-600">
              <h3 class="text-xl font-bold text-gray-800 dark:text-gray-100 mb-2">{{ category }}</h3>
              <ul class="list-disc list-inside space-y-1 text-gray-700 dark:text-gray-200">
                <li v-for="item in items" :key="item">{{ item }}</li>
              </ul>
            </div>
          </div>
          <div v-else class="p-4 text-center text-gray-500 dark:text-gray-400">
            <p><i class="fas fa-info-circle mr-2"></i> Список покупок не найден.</p>
            <p>Убедитесь, что `BuyLists.csv` находится в `src/assets/`.</p>
          </div>
        </div>
      </transition>
    </div>

  </section>
</template>

<script setup>
import { ref, onMounted, reactive, computed } from 'vue'
import Papa from 'papaparse'

// Данные для расписания еды
const headers = ref([])
const rows = ref([])
const groupedFoodSchedule = ref({})

// Состояние для сворачивания дней недели (внутри Меню)
const collapsedDays = reactive({});
const allDaysCollapsed = computed(() => {
  if (Object.keys(groupedFoodSchedule.value).length === 0) return true;
  return Object.keys(groupedFoodSchedule.value).every(day => collapsedDays[day]);
});
const toggleDayCollapse = (day) => { collapsedDays[day] = !collapsedDays[day]; };
const toggleAllDays = () => {
  const targetState = !allDaysCollapsed.value;
  Object.keys(groupedFoodSchedule.value).forEach(day => {
    collapsedDays[day] = targetState;
  });
};

// НОВОЕ: Состояние для сворачивания всего блока "Меню на неделю"
const foodScheduleCollapsed = ref(false); // По умолчанию развернуто
const toggleFoodScheduleCollapse = () => {
  foodScheduleCollapsed.value = !foodScheduleCollapsed.value;
};


// Данные для "Как готовить"
const cookingMethods = ref({});
const recipesCollapsed = ref(true); // По умолчанию свернуто
const toggleRecipesCollapse = () => { recipesCollapsed.value = !recipesCollapsed.value; };

// Данные для "Списка покупок"
const buyList = ref({});
const buyListCollapsed = ref(true); // По умолчанию свернуто
const toggleBuyListCollapse = () => { buyListCollapsed.value = !buyListCollapsed.value; };


// --- Функции обработки CSV ---

// Обработка foods.csv (та же, что и раньше)
const processFoodData = (rawHeaders, rawRows) => {
  const foodSchedule = {};
  const dayNames = rawHeaders.slice(1);

  dayNames.forEach(day => {
    foodSchedule[day] = [];
    collapsedDays[day] = false; // Инициализация для дней недели
  });

  rawRows.forEach(row => {
    const mealType = row[0];
    if (!mealType) return;

    dayNames.forEach((dayName, index) => {
      const foodItem = row[index + 1];
      if (foodItem && foodItem.trim() !== '' && foodItem.trim() !== '–') {
        foodSchedule[dayName].push({
          mealType: mealType,
          foodItem: foodItem.trim()
        });
      }
    });
  });

  const dayOrder = ['Понедельник', 'Вторник', 'Среда', 'Четверг', 'Пятница', 'Суббота', 'Воскресенье'];
  const orderedFoodSchedule = {};
  dayOrder.forEach(day => {
    if (foodSchedule[day] && foodSchedule[day].length > 0) {
      orderedFoodSchedule[day] = foodSchedule[day];
    }
  });

  for (const day in foodSchedule) {
    if (!orderedFoodSchedule[day] && foodSchedule[day].length > 0) {
      orderedFoodSchedule[day] = foodSchedule[day];
    }
  }

  return orderedFoodSchedule;
};

// Новая функция: Обработка cookingMethods.csv
const processCookingMethods = (data) => {
  const methods = {};
  data.forEach(row => {
    if (row.length >= 2 && row[0] && row[1]) {
      methods[row[0].trim()] = row[1].trim(); // Блюдо: Рецепт
    }
  });
  return methods;
};

// Новая функция: Обработка BuyLists.csv
const processBuyList = (rawHeaders, rawRows) => {
  const list = {};
  rawHeaders.forEach(category => {
    if (category && category.trim() !== '') {
      list[category.trim()] = [];
    }
  });

  rawRows.forEach(row => {
    rawHeaders.forEach((category, index) => {
      const item = row[index];
      if (item && item.trim() !== '') {
        list[category.trim()].push(item.trim());
      }
    });
  });
  return list;
};


// --- Загрузка данных при монтировании ---
onMounted(async () => {
  // Загрузка foods.csv
  try {
    const res = await fetch('/daymaster-app/foods.csv')
    if (!res.ok) throw new Error(`HTTP error! status: ${res.status} for foods.csv`);
    const text = await res.text()
    const parsed = Papa.parse(text, { skipEmptyLines: true })
    if (parsed.data.length > 0) {
      headers.value = parsed.data[0];
      rows.value = parsed.data.slice(1);
      groupedFoodSchedule.value = processFoodData(headers.value, rows.value);
    } else {
      console.warn('foods.csv is empty.')
    }
  } catch (error) {
    console.error('Error loading or parsing foods.csv:', error)
    groupedFoodSchedule.value = {};
  }

  // Загрузка cookingMethods.csv
  try {
    const res = await fetch('/daymaster-app/cookingMethods.csv')
    if (!res.ok) throw new Error(`HTTP error! status: ${res.status} for cookingMethods.csv`);
    const text = await res.text()
    const parsed = Papa.parse(text, { skipEmptyLines: true })
    if (parsed.data.length > 0) {
      // Здесь headers не нужны, т.к. файл имеет простую структуру "блюдо, рецепт"
      cookingMethods.value = processCookingMethods(parsed.data.slice(1)); // Пропускаем заголовок
    } else {
      console.warn('cookingMethods.csv is empty.')
    }
  } catch (error) {
    console.error('Error loading or parsing cookingMethods.csv:', error)
    cookingMethods.value = {};
  }

  // Загрузка BuyLists.csv
  try {
    const res = await fetch('/daymaster-app/BuyLists.csv')
    if (!res.ok) throw new Error(`HTTP error! status: ${res.status} for BuyLists.csv`);
    const text = await res.text()
    const parsed = Papa.parse(text, { skipEmptyLines: true })
    if (parsed.data.length > 0) {
      const buyListHeaders = parsed.data[0];
      const buyListRows = parsed.data.slice(1);
      buyList.value = processBuyList(buyListHeaders, buyListRows);
    } else {
      console.warn('BuyLists.csv is empty.')
    }
  } catch (error) {
    console.error('Error loading or parsing BuyLists.csv:', error)
    buyList.value = {};
  }
})
</script>

<style scoped>
/* Анимация аккордеона - те же стили, что и раньше */
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
  max-height: 1000px; /* Увеличено значение max-height для рецептов и списков покупок */
  opacity: 1;
  transform: translateY(0);
}
</style>