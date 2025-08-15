<script setup>
import { ref, onMounted, computed } from 'vue'
import dayjs from 'dayjs'
import utc from 'dayjs/plugin/utc'
import timezone from 'dayjs/plugin/timezone'

dayjs.extend(utc)
dayjs.extend(timezone)

const loading = ref(true)
const data = ref(null)

const fetchData = async () => {
  const res = await fetch('https://hh.frontend.ark.software/api/booking')
  data.value = await res.json()
  loading.value = false
}

onMounted(fetchData)

const tables = computed(() => data.value?.tables || [])
const openingTime = computed(() => data.value?.restaurant?.opening_time || '11:00')
const closingTime = computed(() => data.value?.restaurant?.closing_time || '23:00')
const tz = computed(() => data.value?.restaurant?.timezone || 'Asia/Vladivostok')
const currentDay = computed(() => data.value?.current_day || '2025-04-04')

const timeSlots = computed(() => {
  if (!currentDay.value) return []

  const start = dayjs.tz(`${currentDay.value}T${openingTime.value}`, tz.value)
  const end = dayjs.tz(`${currentDay.value}T${closingTime.value}`, tz.value)
  const slots = []
  let t = start
  while (t.isBefore(end)) {
    slots.push(t.format('HH:mm'))
    t = t.add(30, 'minute')
  }
  return slots
})

const getMinutesFromOpen = (timeStr) => {
  if (!timeStr) return 0
  const start = dayjs.tz(`${currentDay.value}T${openingTime.value}`, tz.value)
  const time = dayjs.tz(timeStr).tz(tz.value)
  return time.diff(start, 'minute')
}

const getEventStyle = (event, colIndex) => {
  const startMin = getMinutesFromOpen(event.start_time || event.seating_time)
  const endMin = getMinutesFromOpen(event.end_time)
  const top = (startMin / 30) * 40
  const height = ((endMin - startMin) / 30) * 40
  return {
    top: top + 'px',
    height: height + 'px',
    left: (colIndex * 120) + 'px',
    background: event.status === 'Banquet' ? 'purple' : 'teal'
  }
}
</script>

<template>
  <div v-if="loading" class="loading">Загрузка...</div>

  <div v-else class="booking-grid">
    <div class="time-header"></div>
    <div v-for="table in tables" :key="table.id" class="table-header">
      {{ table.number }} <small>({{ table.zone }})</small>
    </div>

    <div v-for="(time, i) in timeSlots" :key="'time-'+i" class="time-cell">
      {{ time }}
    </div>

    <div
      v-for="(table, colIndex) in tables"
      :key="'col-'+table.id"
      class="table-column"
    >
      <div
        v-for="order in [...table.orders, ...table.reservations]"
        :key="order.id"
        class="event"
        :style="getEventStyle(order, colIndex)"
      >
        {{ order.name_for_reservation || order.status }}
      </div>
    </div>
  </div>
</template>

<style scoped>
.booking-grid {
  display: grid;
  grid-template-columns: 100px repeat(auto-fit, 120px);
  position: relative;
}
.table-header {
  background: #ddd;
  text-align: center;
  font-weight: bold;
}
.time-cell {
  height: 40px;
  border-bottom: 1px solid #ccc;
  font-size: 12px;
}
.table-column {
  position: relative;
}
.event {
  position: absolute;
  width: 100px;
  color: white;
  font-size: 12px;
  padding: 2px;
  border-radius: 4px;
}
.loading {
  padding: 20px;
}
</style>
