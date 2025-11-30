<template>
  <span class="font-mono">
    {{ formattedPhone }}
  </span>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  value: {
    type: String,
    required: true,
  },
})

const formattedPhone = computed(() => {
  let phone = props.value

  // Удаляем все нецифровые символы, кроме +
  phone = phone.replace(/[^\d+]/g, '')

  // Заменяем начало 8 на +7
  if (phone.startsWith('8')) {
    phone = '+7' + phone.slice(1)
  }

  // Если нет кода страны, добавляем +7
  if (!phone.startsWith('+')) {
    phone = '+7' + phone
  }

  return phone
})
</script>
