<script>
import './BookingGrid.css'
import dayjs from 'dayjs'
import utc from 'dayjs/plugin/utc'
import timezone from 'dayjs/plugin/timezone'

dayjs.extend(utc)
dayjs.extend(timezone)
export default {
  props: {
    tables: { type: Array, required: true },
    timeSlots: { type: Array, required: true },
    getEventStyle: { type: Function, required: true }
  
  },
  methods: {
    formatTime(dateStr) {
      if (!dateStr) return ''
      return dayjs(dateStr).tz(this.$root.tz || 'Asia/Vladivostok').format('HH:mm')
    },
    formatEventLabel(event) {
      if (event.status === 'Banquet') {
        return `Банкет — ${this.formatTime(event.start_time)}–${this.formatTime(event.end_time)}`
      }
      if (event.status === 'Живая очередь') {
        return `№${event.id} ${event.name_for_reservation}, ${event.num_people} чел., Живая очередь, ${event.phone_number}, ${this.formatTime(event.seating_time)}`
      }
      if (event.start_time && !event.name_for_reservation) {
        return `Заказ — ${event.status}, ${this.formatTime(event.start_time)}–${this.formatTime(event.end_time)}`
      }
      if (event.name_for_reservation) {
        return `№${event.id} ${event.name_for_reservation}, ${event.num_people} чел., ${event.status}, ${event.phone_number}, ${this.formatTime(event.seating_time)}`
      }
      return ''
    }
  }
}
</script>

<template>
  <div class="booking-wrapper">
    <div class="header-row">
      <div class="corner-cell"></div>
      <div v-for="table in tables" :key="table.id" class="table-header">
        {{ table.number }} <small>({{ table.zone }})</small>
      </div>
    </div>

    <div class="body">
      <div class="time-column">
        <div v-for="(time, i) in timeSlots" :key="'time-'+i" class="time-cell">
          {{ time }}
        </div>
      </div>

      <div class="tables-columns">
        <div v-for="(table, colIndex) in tables" :key="'col-'+table.id" class="table-column">
          <div
            v-for="order in [...table.orders, ...table.reservations]"
            :key="order.id"
            class="event"
            :style="getEventStyle(order, colIndex)"
          >
            {{ formatEventLabel(order) }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
