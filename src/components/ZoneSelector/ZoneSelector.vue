<script setup>
import './ZoneSelector.css'

const props = defineProps({
  zones: { type: Array, required: true },     
  modelValue: { type: Array, required: true }  
})

const emit = defineEmits(['update:modelValue'])

function toggleZone(zone) {
  const selected = new Set(props.modelValue)
  if (selected.has(zone)) selected.delete(zone)
  else selected.add(zone)
  emit('update:modelValue', [...selected])
}
</script>

<template>
  <div class="zone-selector">
    <label>Зоны:</label>
    <div class="zone-buttons">
      <button
        v-for="zone in props.zones"
        :key="zone"
        type="button"
        :class="['zone-button', { active: props.modelValue.includes(zone) }]"
        @click="toggleZone(zone)"
      >
        {{ zone }}
      </button>
    </div>
  </div>
</template>
