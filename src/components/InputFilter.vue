<template>
  <div class="filter-container">
    <span class="filter-label">{{ placeholder }}</span>
    <input v-model="inputValue" @input="onInput" :placeholder="placeholder" class="filter-input" />
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  placeholder: {
    type: String,
    default: 'Search...',
  },
  column: {
    type: String,
    required: true,
  },
})

const emit = defineEmits(['filter'])

const inputValue = ref('')

const onInput = () => {
  emit('filter', { key: props.column, value: inputValue.value })
}

const reset = () => {
  inputValue.value = ''
  emit('filter', { key: props.column, value: '' })
}

defineExpose({
  reset,
})
</script>

<style scoped>
.filter-container {
  display: flex;
  flex-direction: column;
}

.filter-label {
  font-size: 14px;
  font-weight: 500;
  margin-bottom: 4px;
}
</style>
