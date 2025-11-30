<template>
  <div class="country-badges">
    <span v-for="country in countriesArray" :key="country" class="country-badge">
      {{ country }}
    </span>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  value: {
    type: [Array, String],
    required: true,
  },
})

const countriesArray = computed(() => {
  if (Array.isArray(props.value)) {
    return props.value
  }

  try {
    return JSON.parse(props.value)
  } catch {
    return [props.value]
  }
})
</script>

<style scoped>
.country-badges {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}

.country-badge {
  display: inline-block;
  padding: 4px 10px;
  background: #dbeafe;
  color: #1e40af;
  font-size: 12px;
  font-weight: 500;
  border-radius: 9999px;
  border: 1px solid #93c5fd;
  white-space: nowrap;
  line-height: 1;
}

.dark .country-badge {
  background: #1e3a8a;
  color: #dbeafe;
  border-color: #3b82f6;
}
</style>
