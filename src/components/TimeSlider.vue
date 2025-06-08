<template>
  <div class="flex justify-center">

  <div class="bg-white rounded-lg px-5 w-3/5">
    <div class="flex items-center gap-4 w-full">
      <button
        @click="togglePlayback"
        class="play-button w-10 h-10 rounded-full bg-gray-300 flex items-center justify-center text-xl hover:bg-gray-300 focus:outline-none"
        aria-label="Play/Pause"
      >
        <span aria-hidden="true">
          <svg v-if="isPlaying" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-pause-icon lucide-pause "><rect x="14" y="4" width="4" height="16" rx="1"/><rect x="6" y="4" width="4" height="16" rx="1"/></svg>
          <svg v-else xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-play-icon lucide-play"><polygon points="6 3 20 12 6 21 6 3"/></svg>
        </span>
      </button>

      <vue-slider
        v-model="internalYear"
        :min="minYear"
        :max="maxYear"
        :marks="marks"
        :piecewise="true"
        :piecewise-label="true"
        :tooltip="'always'"
        :dot-size="16"
        :height="8"
        :process-style="{ background: 'linear-gradient(to right, #aaa, #444)' }"
        :rail-style="{ background: '#e5e7eb' }"
        :dot-style="{ border: '2px solid #6b7280', background: '#fff', boxShadow: '0 2px 6px rgba(0,0,0,0.15)' }"
        :mark-style="markStyle"
        :tooltip-style="tooltipStyle"
        class="slider flex-1"
        :edge-dot="true"
        @change="emitYear"
      />
    </div>
  </div>
  </div>

</template>


<script setup>
import { ref, watch, computed, onUnmounted } from 'vue'
import VueSlider from 'vue-slider-component'
import 'vue-slider-component/theme/default.css'

const props = defineProps({
  modelValue: { type: Number, required: true },
  minYear: { type: Number, default: 1900 },
  maxYear: { type: Number, default: 2023 }
})

const emit = defineEmits(['update:modelValue'])

const internalYear = ref(props.modelValue)
watch(() => props.modelValue, (v) => internalYear.value = v)
watch(internalYear, (v) => emit('update:modelValue', v))

const marks = computed(() => {
  const m = {}
  const start = Math.ceil(props.minYear / 10) * 10
  const end = Math.floor(props.maxYear / 10) * 10
  for (let y = start; y <= end; y += 10) m[y] = ''
  return m
})

const markStyle = {
  color: '#6b7280',
  fontSize: '12px',
  transform: 'translateX(-50%)',
  marginTop: '8px',
  whiteSpace: 'nowrap',
};

const tooltipStyle = {
  backgroundColor: '#374151',
  color: '#fff',
  borderRadius: '4px',
  padding: '4px 8px',
  fontSize: '12px',
};

let intervalId = null
const isPlaying = ref(false)

function togglePlayback() {
  if (isPlaying.value) stopPlayback()
  else startPlayback()
}

function startPlayback() {
  isPlaying.value = true
  intervalId = setInterval(() => {
    if (internalYear.value < props.maxYear) internalYear.value++
    else stopPlayback()
  }, 200)
}

function stopPlayback() {
  clearInterval(intervalId)
  isPlaying.value = false
}

onUnmounted(stopPlayback)

function emitYear() {}
</script>
