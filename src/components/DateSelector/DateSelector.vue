<script setup>
import './DateSelector.css'

defineProps({
  days: { type: Array, required: true },
  modelValue: { type: String, required: true }
})

const emit = defineEmits(['update:modelValue'])
</script>

<template>
  <div class="date-selector">
    <label>Дата:</label>
    <select :value="modelValue" @input="emit('update:modelValue', $event.target.value)">
      <option v-for="day in days" :key="day" :value="day">
        {{ day }}
      </option>
    </select>
  </div>
</template>
