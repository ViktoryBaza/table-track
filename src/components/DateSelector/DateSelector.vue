<script setup>
import dayjs from 'dayjs'
import 'dayjs/locale/ru'
import './DateSelector.css'

dayjs.locale('ru')

defineProps({
  days: { type: Array, required: true }, 
  modelValue: { type: String, required: true }
})

const emit = defineEmits(['update:modelValue'])

function selectDay(day) {
  emit('update:modelValue', day)
}

function formatDay(day) {
  return dayjs(day).format('D MMMM')  
}

function formatSubLabel(day) {
  const d = dayjs(day).startOf('day')
  const today = dayjs().startOf('day')

  if (d.isSame(today, 'day')) return 'Сегодня'
  if (d.isSame(today.add(1, 'day'), 'day')) return 'Завтра'

  return d.format('dddd')
}
</script>

<template>
  <div class="date-selector">
    <label>Дата:</label>
    <div class="date-buttons">
      <button
        v-for="day in days"
        :key="day"
        :class="['date-button', { active: modelValue === day }]"
        @click="selectDay(day)"
      >
        <div class="day-title">{{ formatDay(day) }}</div>
        <div class="day-sub">{{ formatSubLabel(day) }}</div>
      </button>
    </div>
  </div>
</template>
