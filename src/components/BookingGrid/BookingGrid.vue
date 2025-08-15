<script setup>
import './BookingGrid.css'

defineProps({
  tables: { type: Array, required: true },
  timeSlots: { type: Array, required: true },
  getEventStyle: { type: Function, required: true }
})
</script>

<template>
  <div class="booking-grid">
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
