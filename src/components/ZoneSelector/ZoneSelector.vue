<script setup>
import './ZoneSelector.css'

defineProps({
  zones: { type: Array, required: true },
  modelValue: { type: Array, required: true }
})

const emit = defineEmits(['update:modelValue'])
</script>

<template>
  <div class="zone-selector">
    <label>Зоны:</label>
    <div v-for="zone in zones" :key="zone" class="zone-checkbox">
      <input
        type="checkbox"
        :value="zone"
        :checked="modelValue.includes(zone)"
        @change="e => {
          const val = e.target.value
          if (e.target.checked) {
            emit('update:modelValue', [...modelValue, val])
          } else {
            emit('update:modelValue', modelValue.filter(z => z !== val))
          }
        }"
      />
      {{ zone }}
    </div>
  </div>
</template>
