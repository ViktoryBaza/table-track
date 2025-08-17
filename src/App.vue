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

const sortedTables = computed(() => {
  if (!data.value) return []
  return data.value.tables
    .filter(t => selectedZones.value.includes(t.zone))
    .slice()  
    .sort((a, b) => {
      if (a.zone === b.zone) {
        return +a.number - +b.number   
      }
      return a.zone.localeCompare(b.zone)  
    })
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

const EVENT_WIDTH = 80
const SLOT_MIN = 30
const SLOT_H = 40
const OVERLAP_GAP = 4

const getEventStyle = (event, colIndex, table, hoveredId) => {
  const start = dayjs(event.start_time || event.seating_time)
  const end = dayjs(event.end_time || (event.start_time || event.seating_time))
  const startMin = getMinutesFromOpen(start)
  const endMin = getMinutesFromOpen(end)

  const top = (startMin / SLOT_MIN) * SLOT_H
  const height = Math.max(18, ((endMin - startMin) / SLOT_MIN) * SLOT_H)

  const events = [...(table.orders || []), ...(table.reservations || [])]
    .filter(e => e && (e.start_time || e.seating_time))
    .sort((a, b) =>
      dayjs(a.start_time || a.seating_time).diff(dayjs(b.start_time || b.seating_time))
    )

  const groups = []
  let curGroup = []
  events.forEach((e, i) => {
    const eStart = dayjs(e.start_time || e.seating_time)
    if (i === 0) {
      curGroup = [e]
      groups.push(curGroup)
      return
    }
    const prevStart = dayjs(events[i - 1].start_time || events[i - 1].seating_time)
    if (Math.abs(eStart.diff(prevStart, 'minute')) <= 30) {
      curGroup.push(e)
    } else {
      curGroup = [e]
      groups.push(curGroup)
    }
  })
  let inThirtyMinGroup = false
  let currentGroup = []
  let indexInGroup = 0
  groups.forEach(g => {
    const idx = g.findIndex(e => e.id === event.id)
    if (idx !== -1) {
      inThirtyMinGroup = g.length > 1
      currentGroup = g
      indexInGroup = idx
    }
  })

  let left = 0
  let width = EVENT_WIDTH

  if (inThirtyMinGroup) {
    const groupSize = currentGroup.length
    width = groupSize > 1 ? EVENT_WIDTH / groupSize : EVENT_WIDTH
    left = indexInGroup * width
  } else {
    const nonThirtyMinEvents = events.filter(e => {
      return !groups.some(g => g.includes(e) && g.length > 1)
    })

    left = 0
    for (let i = 0; i < nonThirtyMinEvents.length; i++) {
      const e = nonThirtyMinEvents[i]
      if (e.id === event.id) break

      const eStart = dayjs(e.start_time || e.seating_time)
      const eEnd = dayjs(e.end_time || (e.start_time || e.seating_time))
      if (start.isBefore(eEnd) && end.isAfter(eStart)) {
        left += OVERLAP_GAP
      }
    }

    width = Math.max(0, EVENT_WIDTH - left)
  }

  const isHovered = hoveredId === event.id

  let baseColor = '#7FD7CC'
  let borderColor = '#4AC99B'

  if (event.status === 'Banquet') {
    baseColor = '#B348F7'
    borderColor = '#7B439E'
  } else if (event.status === 'Живая очередь') {
    baseColor = '#0097FD'
    borderColor = '#007AFF'
  } else if (
    ['Новая', 'Заявка', 'Открыт', 'Закрыт', 'Отменен'].includes(event.status)
  ) {
    baseColor = '#FF7043'
    borderColor = '#FF7043'
  } else if (['New', 'Bill', 'Closed'].includes(event.status)) {
    baseColor = '#7FD7CC'
    borderColor = '#4AC99B'
  }

  return {
    position: 'absolute',
    top: `${top}px`,
    height: `${height}px`,
    left: isHovered ? '0px' : `${left}px`,
    width: isHovered ? `${EVENT_WIDTH}px` : `${width}px`,
    background: `${baseColor}26`,
    borderLeft: `4px solid ${borderColor}`,
    borderRadius: '6px',
    padding: '4px',
    boxSizing: 'border-box',
    color: 'white',
    fontSize: '12px',
    wordBreak: inThirtyMinGroup && !isHovered ? 'normal' : 'break-word', 
    overflow: 'hidden',
    backdropFilter: 'blur(4px)',
    transition: 'all 0.2s ease',
    zIndex: isHovered ? 1000 : 1
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
    :tables="sortedTables"
    :timeSlots="timeSlots"
    :getEventStyle="getEventStyle"
    />

  </div>
</template>


<style scoped>
.app-wrapper {
  background: #1e1e1e;
  color: white;
}
.filters {
  display: flex;
  gap: 20px;
  margin-bottom: 10px;
}
.loading {
  padding: 20px;
  color: white;
}
</style>