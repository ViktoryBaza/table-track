<script>
import './BookingGrid.css'
import dayjs from 'dayjs'
import utc from 'dayjs/plugin/utc'
import timezone from 'dayjs/plugin/timezone'
import phoneSvg from '/src/assets/icons/phone.svg';


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
    return dayjs(dateStr)
      .tz(this.$root.tz || 'Asia/Vladivostok')
      .format('HH:mm')
  },

  translateStatus(status, type = 'order') {
    if (type === 'order') {
      const statusMap = {
        'New': 'Открытый',
        'Bill': 'Пречек',
        'Closed': 'Закрытый',
        'Banquet': 'Банкет'
      }
      return statusMap[status] || status
    } else if (type === 'reservation') {
      const statusMap = {
        'Новая': 'Ожидает подтверждения',
        'Заявка': 'Ожидаем',
        'Открыт': 'В зале',
        'Закрыт': 'Закрыт',
        'Живая очередь': 'Живая очередь'
      }
      return statusMap[status] || status
    }
  },

  formatEventLabel(event) {
  const type = event.name_for_reservation ? 'reservation' : 'order';
  const status = this.translateStatus(event.status, type);

  const rawPhone = event.phone_number || '';
  const phoneNumber = rawPhone.startsWith('+') ? rawPhone.slice(1) : rawPhone;
  const phoneIcon = `<img src="${phoneSvg}" class="phone-icon" alt="телефон" />`;

  let statusClass = '';
  if (status === 'Ожидает подтверждения') statusClass = 'status-await-confirm';
  else if (status === 'Ожидаем') statusClass = 'status-await';
  else if (status === 'В зале') statusClass = 'status-in-hall';
  else if (status === 'Отменен') statusClass = 'status-cancel';
  else if (status === 'Пречек') statusClass = 'status-precheck';
  else if (status === 'Закрытый' || status === 'Новый') statusClass = 'status-closed-new';
  else if (status === 'Живая очередь') statusClass = 'status-queue';

  if (event.name_for_reservation) {
  return `<span class="order-id">№${event.id}</span> <br>
          ${event.name_for_reservation}; ${event.num_people}чел<br>
          <span class="${statusClass}">${status}</span><br>
          <span class="phone-wrapper">${phoneIcon} ${phoneNumber}</span>
          ${this.formatTime(event.seating_time)}${event.end_time ? '–' + this.formatTime(event.end_time) : ''}`;
}

if (event.start_time && !event.name_for_reservation) {
  if (status === 'Банкет') {
    return `<span class="${statusClass}">${status}</span><br>
            ${this.formatTime(event.start_time)}–${this.formatTime(event.end_time)}`;
  }

  return `<span>Заказ</span> 
          <span class="${statusClass}"><br>${status}</span><br>
          ${this.formatTime(event.start_time)}–${this.formatTime(event.end_time)}`;
}



  return '';
}
}
}
</script>

<template>
  <div class="booking-wrapper">
      <div class="header-row">
      <div class="corner-cell"></div>
      <div 
        v-for="table in tables" 
        :key="table.id" 
        class="table-header"
      >
      <div class="table-group">
        <div class="table-number">
        #<span class="table-number-digit">{{ table.number }}</span>
      </div>

        <div class="table-capacity">{{ table.capacity }} чел</div>
      </div>
        <div class="table-zone">{{ table.zone }}</div>
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
        v-for="order in [...(table.orders || []), ...(table.reservations || [])]"
        :key="order.id"
        class="event"
        :style="getEventStyle(order,colIndex, table)"
        v-html="formatEventLabel(order)"
      >
      </div>
      </div>
    </div>
    </div>
  </div>
</template>
