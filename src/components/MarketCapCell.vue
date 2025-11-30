<template>
  <span class="font-mono text-sm">
    {{ formattedValue }}
  </span>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  value: {
    type: [Number, String],
    required: true,
  },
})

const formattedValue = computed(() => {
  const num = Number(props.value)
  if (isNaN(num)) return props.value

  if (num >= 1000000000) {
    return (num / 1000000000).toFixed(1) + 'B'
  } else if (num >= 1000000) {
    return (num / 1000000).toFixed(1) + 'M'
  } else if (num >= 1000) {
    return (num / 1000).toFixed(1) + 'K'
  } else {
    return num.toString()
  }
})
</script>
