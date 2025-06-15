<template>
  <div
    role="tablist"
    @keydown="onKeydown"
    class="flex gap-4 justify-center"
  >
    <button
      v-for="(option, index) in options"
      :key="option.value"
      role="tab"
      :id="`tab-${option.value}`"
      :tabindex="option.value === modelValue ? 0 : -1"
      :aria-selected="option.value === modelValue"
      @click="updateSelection(option.value)"
      :class="[
        'px-5 py-2 border-2 rounded-md text-sm cursor-pointer transition-all duration-300',
        option.value === modelValue
          ? 'border-blue-600 bg-blue-100 font-semibold text-blue-800'
          : 'border-gray-300 bg-white text-gray-900 hover:bg-gray-100 focus:outline-none focus:ring-2 focus:ring-blue-400'
      ]"
    >
      {{ option.label }}
    </button>
  </div>
</template>
  
<script setup>
import { defineProps, defineEmits } from 'vue'
  
const props = defineProps({
  options: {
    type: Array,
    required: true,
  },
  modelValue: {
    type: String,
    required: true
  }
})
  
const emit = defineEmits(['update:modelValue'])
  
// Update selection and focus active tab
function updateSelection(value) {
  emit('update:modelValue', value)
  const btn = document.getElementById(`tab-${value}`)
  btn && btn.focus()
}
  
// Keyboard navigation for tabs
function onKeydown(event) {
  const { key } = event
  const idx = props.options.findIndex(opt => opt.value === props.modelValue)
  let newIdx = idx
  
  if (key === 'ArrowRight') {
    newIdx = (idx + 1) % props.options.length
  } else if (key === 'ArrowLeft') {
    newIdx = (idx - 1 + props.options.length) % props.options.length
  } else {
    return
  }
  
  event.preventDefault()
  const newValue = props.options[newIdx].value
  updateSelection(newValue)
}
</script>