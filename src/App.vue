<script setup>
import { ref, onMounted, computed } from 'vue'
import dayjs from 'dayjs'
import utc from 'dayjs/plugin/utc'
import timezone from 'dayjs/plugin/timezone'

import DateSelector from './components/DateSelector/DateSelector.vue'
import ZoneSelector from './components/ZoneSelector/ZoneSelector.vue'
import BookingGrid from './components/BookingGrid/BookingGrid.vue'


dayjs.extend(utc)
dayjs.extend(timezone)

const loading = ref(true)
const data = ref(null)

const selectedDay = ref('')
const selectedZones = ref([])

const fetchData = async () => {
  const res = await fetch('https://hh.frontend.ark.software/api/booking')
  data.value = await res.json()

  if (data.value?.available_days?.length) {
    selectedDay.value = data.value.available_days[0]
  }

  if (data.value?.tables?.length) {
    selectedZones.value = [...new Set(data.value.tables.map(t => t.zone))]
  }

  loading.value = false
}

onMounted(fetchData)

const tables = computed(() => {
  if (!data.value) return []
  return data.value.tables.filter(t => selectedZones.value.includes(t.zone))
})

const availableDays = computed(() => data.value?.available_days || [])
const zones = computed(() => [...new Set(data.value?.tables?.map(t => t.zone) || [])])

const openingTime = computed(() => data.value?.restaurant?.opening_time || '11:00')
const closingTime = computed(() => data.value?.restaurant?.closing_time || '23:00')
const tz = computed(() => data.value?.restaurant?.timezone || 'Asia/Vladivostok')

const timeSlots = computed(() => {
  if (!selectedDay.value) return []

  const start = dayjs.tz(`${selectedDay.value}T${openingTime.value}`, tz.value)
  const end = dayjs.tz(`${selectedDay.value}T${closingTime.value}`, tz.value)
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
  const start = dayjs.tz(`${selectedDay.value}T${openingTime.value}`, tz.value)
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

  <div v-else>
    <div class="filters">
      <DateSelector v-model="selectedDay" :days="availableDays" />
      <ZoneSelector v-model="selectedZones" :zones="zones" />
    </div>

    <BookingGrid
      :tables="tables"
      :timeSlots="timeSlots"
      :getEventStyle="getEventStyle"
    />
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

</style>