<template>
  <div class="space-y-8">
    <div class="bg-white dark:bg-gray-800 shadow rounded-xl p-6 transition-colors duration-300">
      <div class="flex justify-between items-center mb-6">
        <h2 class="text-lg font-semibold">Данные пользователей</h2>
        <button
          v-if="rows.length > 0"
          @click="resetFilters"
          class="bg-red-500 hover:bg-red-600 dark:bg-red-600 dark:hover:bg-red-700 text-white px-4 py-2 rounded text-sm transition-colors"
        >
          Очистить фильтры
        </button>
      </div>

      <div class="table-container">
        <table class="base-table">
          <thead>
            <tr>
              <th v-for="(value, key) in headers" :key="key" class="header-cell">
                <div class="header-content">
                  <slot :name="`header-${key}`" :key="key" :value="value">
                    <div class="header-default">
                      <span class="header-text">{{ value }}</span>
                      <button
                        v-if="sortableColumns.includes(key)"
                        @click="sortColumn(key)"
                        class="sort-btn"
                        :class="{ active: sortKey === key }"
                      >
                        {{ sortOrders[key] > 0 ? '↑' : '↓' }}
                      </button>
                    </div>
                  </slot>
                </div>
              </th>
            </tr>
            <!-- Строка фильтров -->
            <tr class="filter-row">
              <th v-for="(value, key) in headers" :key="`filter-${key}`" class="filter-cell">
                <slot :name="`filter-${key}`" :key="key">
                  <input
                    v-model="filters[key]"
                    @input="applyFilter(key, $event.target.value)"
                    :placeholder="`Поиск...`"
                    class="filter-input"
                  />
                </slot>
              </th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(row, index) in sortedAndFilteredRows" :key="index" class="table-row">
              <td v-for="(headerValue, key) in headers" :key="key" class="table-cell">
                <slot :name="key" :value="row[key]" :row="row">
                  {{ row[key] }}
                </slot>
              </td>
            </tr>
            <tr v-if="sortedAndFilteredRows.length === 0">
              <td :colspan="Object.keys(headers).length" class="empty-cell">
                Нет данных для отображения
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="table-controls">
        <span class="results-count">
          Показано: {{ sortedAndFilteredRows.length }} из {{ totalRows }}
        </span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  rows: {
    type: Array,
    required: true,
  },
  totalRows: {
    type: Number,
    required: true,
  },
  headers: {
    type: Object,
    required: true,
  },
  sortableColumns: {
    type: Array,
    default: () => ['firstName', 'lastName', 'email', 'marketCap'],
  },
})

// Реактивные состояния
const filters = ref({})
const sortKey = ref('')
const sortOrders = ref({})

// Инициализируем порядок сортировки для всех колонок
watch(
  () => props.headers,
  (newHeaders) => {
    Object.keys(newHeaders).forEach((key) => {
      if (!(key in sortOrders.value)) {
        sortOrders.value[key] = 1 // 1 для возрастания, -1 для убывания
      }
    })
  },
  { immediate: true },
)

// Отфильтрованные данные (без сортировки)
const filteredRows = computed(() => {
  if (Object.keys(filters.value).length === 0) {
    return props.rows
  }

  return props.rows.filter((row) => {
    return Object.entries(filters.value).every(([key, filterValue]) => {
      if (!filterValue || filterValue.trim() === '') return true

      const cellValue = String(row[key]).toLowerCase()
      return cellValue.includes(filterValue.toLowerCase())
    })
  })
})

// Отсортированные и отфильтрованные данные
const sortedAndFilteredRows = computed(() => {
  if (!sortKey.value) return filteredRows.value

  return [...filteredRows.value].sort((a, b) => {
    const aVal = a[sortKey.value]
    const bVal = b[sortKey.value]

    // Для числовых значений
    if (typeof aVal === 'number' && typeof bVal === 'number') {
      return (aVal - bVal) * sortOrders.value[sortKey.value]
    }

    // Для строковых значений
    return String(aVal).localeCompare(String(bVal)) * sortOrders.value[sortKey.value]
  })
})

// Функция сортировки
const sortColumn = (key) => {
  if (sortKey.value === key) {
    // Меняем направление сортировки
    sortOrders.value[key] = sortOrders.value[key] * -1
  } else {
    // Новая колонка для сортировки
    sortKey.value = key
    sortOrders.value[key] = 1
  }
}

// Функция для применения фильтра
const applyFilter = (key, value) => {
  if (value && value.trim() !== '') {
    filters.value[key] = value.trim()
  } else {
    delete filters.value[key]
  }
}

// Сброс всех фильтров и сортировки
const resetFilters = () => {
  filters.value = {}
  sortKey.value = ''
  Object.keys(sortOrders.value).forEach((key) => {
    sortOrders.value[key] = 1
  })
  emit('reset-filters')
}

const emit = defineEmits(['reset-filters'])

// Предоставляем функции для использования в родительском компоненте
defineExpose({
  applyFilter,
  resetFilters,
})
</script>

<style scoped>
.table-container {
  overflow-x: auto;
  margin: 0 -0.5rem;
}

.base-table {
  width: 100%;
  border-collapse: collapse;
  min-width: 800px;
  border-radius: 0.75rem;
  overflow: hidden;
}

/* Заголовки - СВЕТЛАЯ ТЕМА БЕЛЫЕ, ТЕМНАЯ ТЕМА ТЕМНЫЕ */
.header-cell {
  padding: 1rem;
  text-align: left;
  background: #ffffff;
  color: #111827;
  font-weight: 600;
  border-bottom: 2px solid #e5e7eb;
}

.dark .header-cell {
  background: #111827;
  color: #f9fafb;
  border-bottom-color: #374151;
}

.header-cell:not(:last-child) {
  border-right: 1px solid #e5e7eb;
}

.dark .header-cell:not(:last-child) {
  border-right-color: #374151;
}

/* ЯЧЕЙКИ - СВЕТЛАЯ ТЕМА БЕЛЫЕ, ТЕМНАЯ ТЕМА ТЕМНЫЕ */
.table-cell {
  padding: 1rem;
  border-bottom: 1px solid #e5e7eb;
  background: #ffffff;
  color: #374151;
  font-weight: 500;
}

.dark .table-cell {
  background: #111827;
  color: #f9fafb;
  border-bottom-color: #374151;
}

.table-cell:not(:last-child) {
  border-right: 1px solid #e5e7eb;
}

.dark .table-cell:not(:last-child) {
  border-right-color: #374151;
}

.table-row:hover .table-cell {
  background: #f9fafb;
}

.dark .table-row:hover .table-cell {
  background: #1f2937;
}

/* Строка фильтров - СВЕТЛАЯ ТЕМА СВЕТЛАЯ, ТЕМНАЯ ТЕМА ТЕМНАЯ */
.filter-row {
  background: #f8fafc !important;
}

.dark .filter-row {
  background: #111827 !important;
}

.filter-cell {
  padding: 0.5rem 1rem !important;
  border-bottom: 1px solid #e5e7eb !important;
}

.dark .filter-cell {
  border-bottom-color: #374151 !important;
}

.filter-cell:not(:last-child) {
  border-right: 1px solid #e5e7eb !important;
}

.dark .filter-cell:not(:last-child) {
  border-right-color: #374151 !important;
}

/* Поля ввода фильтров */
.filter-input {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #d1d5db;
  border-radius: 0.375rem;
  background: white;
  color: #374151;
  font-size: 0.875rem;
}

.dark .filter-input {
  background: #1f2937;
  border-color: #374151;
  color: #f9fafb;
}

.filter-input:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.2);
}

.filter-input::placeholder {
  color: #9ca3af;
}

/* Select элементы */
select {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #d1d5db;
  border-radius: 0.375rem;
  background: white;
  color: #374151;
  cursor: pointer;
}

.dark select {
  background: #1f2937;
  border-color: #374151;
  color: #f9fafb;
}

select:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.2);
}

/* Кнопки сортировки */
.sort-btn {
  background: rgba(0, 0, 0, 0.1);
  border: none;
  color: #374151;
  cursor: pointer;
  padding: 0.25rem 0.5rem;
  border-radius: 0.25rem;
  font-size: 0.75rem;
}

.dark .sort-btn {
  background: rgba(255, 255, 255, 0.1);
  color: #f9fafb;
}

.sort-btn:hover {
  background: rgba(0, 0, 0, 0.2);
}

.dark .sort-btn:hover {
  background: rgba(255, 255, 255, 0.2);
}

.sort-btn.active {
  background: rgba(0, 0, 0, 0.3);
}

.dark .sort-btn.active {
  background: rgba(255, 255, 255, 0.3);
}

/* Остальные стили */
.header-content {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.header-default {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.5rem;
}

.header-text {
  font-weight: 600;
}

.empty-cell {
  padding: 2.5rem 1rem;
  text-align: center;
  color: #6b7280;
  font-style: italic;
  background: #f8fafc;
}

.dark .empty-cell {
  color: #9ca3af;
  background: #374151;
}

.table-controls {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  margin-top: 1rem;
  padding-top: 1rem;
  border-top: 1px solid #e5e7eb;
}

.dark .table-controls {
  border-top-color: #374151;
}

.results-count {
  color: #6b7280;
  font-size: 0.875rem;
  font-weight: 500;
}

.dark .results-count {
  color: #9ca3af;
}
</style>
