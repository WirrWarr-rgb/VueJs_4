<template>
  <div :class="isDarkMode ? 'dark' : ''">
    <div
      class="min-h-screen bg-gray-50 dark:bg-gray-900 text-gray-900 dark:text-gray-100 p-6 transition-colors duration-300"
    >
      <div class="max-w-7xl mx-auto space-y-8">
        <div class="flex justify-between items-center">
          <h1 class="text-3xl font-bold">Таблица пользователей</h1>

          <!-- Переключатель темы -->
          <button
            @click="toggleTheme"
            class="bg-white dark:bg-gray-800 shadow rounded-lg p-3 transition-colors duration-300 border border-gray-200 dark:border-gray-700 hover:shadow-md"
            :title="isDarkMode ? 'Переключить на светлую тему' : 'Переключить на темную тему'"
          >
            <span v-if="isDarkMode" class="text-xl">🌙</span>
            <span v-else class="text-xl">☀️</span>
          </button>
        </div>

        <BaseTable
          ref="baseTableRef"
          :rows="filteredUsers"
          :totalRows="users.length"
          :headers="headers"
          :sortableColumns="['firstName', 'lastName', 'email', 'marketCap']"
          @reset-filters="handleResetFilters"
        >
          <!-- Кастомные слоты для ячеек -->
          <template #email="{ value }">
            <a
              :href="`mailto:${value}`"
              class="email-link text-blue-600 dark:text-blue-400 hover:underline"
            >
              {{ value }}
            </a>
          </template>

          <template #image="{ value }">
            <img :src="value" alt="Avatar" class="avatar w-10 h-10 rounded-full" />
          </template>

          <template #marketCap="{ value }">
            <MarketCapCell :value="value" />
          </template>

          <template #developedCountries="{ value }">
            <CountryBadges :value="value" />
          </template>

          <template #phone="{ value }">
            <PhoneFormatter :value="value" />
          </template>

          <!-- Кастомные фильтры -->
          <template #filter-marketCap>
            <select
              v-model="marketCapFilter"
              @change="applySpecialFilters"
              class="w-full border border-gray-300 dark:border-gray-600 p-2 rounded bg-white dark:bg-gray-700 text-gray-900 dark:text-gray-100 transition-colors"
            >
              <option value="">Все значения</option>
              <option value="small">Меньше 100M</option>
              <option value="medium">100M - 1B</option>
              <option value="large">Больше 1B</option>
            </select>
          </template>

          <template #filter-developedCountries>
            <select
              v-model="countryFilter"
              @change="applySpecialFilters"
              class="w-full border border-gray-300 dark:border-gray-600 p-2 rounded bg-white dark:bg-gray-700 text-gray-900 dark:text-gray-100 transition-colors"
            >
              <option value="">Все страны</option>
              <option v-for="country in uniqueCountries" :key="country" :value="country">
                {{ country }}
              </option>
            </select>
          </template>
        </BaseTable>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import BaseTable from './components/BaseTable.vue'
import MarketCapCell from './components/MarketCapCell.vue'
import CountryBadges from './components/CountryBadges.vue'
import PhoneFormatter from './components/PhoneFormatter.vue'

// Состояние для темы
const isDarkMode = ref(false)

// Заголовки таблицы
const headers = {
  firstName: 'Имя',
  lastName: 'Фамилия',
  email: 'Email',
  phone: 'Телефон',
  image: 'Фото',
  marketCap: 'Рыночная капитализация',
  developedCountries: 'Развитые страны',
}

// Данные
const users = ref([])
const baseTableRef = ref(null)

// Специальные фильтры
const marketCapFilter = ref('')
const countryFilter = ref('')

// Функция переключения темы
const toggleTheme = () => {
  isDarkMode.value = !isDarkMode.value
}

// Применяем класс темы к html элементу
watch(isDarkMode, (newValue) => {
  if (newValue) {
    document.documentElement.classList.add('dark')
  } else {
    document.documentElement.classList.remove('dark')
  }
  localStorage.setItem('darkMode', JSON.stringify(newValue))
})

// Загрузка данных и темы
onMounted(async () => {
  try {
    const response = await fetch('/data.json')
    users.value = await response.json()
  } catch (error) {
    console.error('Ошибка загрузки данных:', error)
  }

  // Загрузка темы из LocalStorage
  const savedTheme = localStorage.getItem('darkMode')
  if (savedTheme) {
    isDarkMode.value = JSON.parse(savedTheme)
    // Применяем тему сразу после загрузки
    if (isDarkMode.value) {
      document.documentElement.classList.add('dark')
    } else {
      document.documentElement.classList.remove('dark')
    }
  }
})

// Уникальные страны для фильтра
const uniqueCountries = computed(() => {
  const countries = new Set()
  users.value.forEach((user) => {
    if (Array.isArray(user.developedCountries)) {
      user.developedCountries.forEach((country) => countries.add(country))
    }
  })
  return Array.from(countries).sort()
})

// Применение специальных фильтров
const applySpecialFilters = () => {
  // Фильтрация будет происходить в computed свойстве filteredUsers
}

// Основная фильтрация данных
const filteredUsers = computed(() => {
  let filtered = users.value

  // Фильтр по рыночной капитализации
  if (marketCapFilter.value) {
    filtered = filtered.filter((user) => {
      const marketCap = user.marketCap
      switch (marketCapFilter.value) {
        case 'small':
          return marketCap < 100000000 // < 100M
        case 'medium':
          return marketCap >= 100000000 && marketCap <= 1000000000 // 100M - 1B
        case 'large':
          return marketCap > 1000000000 // > 1B
        default:
          return true
      }
    })
  }

  // Фильтр по странам
  if (countryFilter.value) {
    filtered = filtered.filter((user) => {
      if (Array.isArray(user.developedCountries)) {
        return user.developedCountries.includes(countryFilter.value)
      }
      return false
    })
  }

  return filtered
})

// Обработчик сброса фильтров
const handleResetFilters = () => {
  marketCapFilter.value = ''
  countryFilter.value = ''
}
</script>
